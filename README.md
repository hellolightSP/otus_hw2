**Домашнее задание**

Работа с mdadm:

- добавить в Vagrantfile еще дисков;
- сломать/починить raid;
- собрать R0/R5/R10 на выбор;
- прописать собранный рейд в конф, чтобы рейд собирался при загрузке;
- создать GPT раздел и 5 партиций.
- 
**На проверку отправьте**
- измененный Vagrantfile,
- скрипт для создания рейда,
- конф для автосборки рейда при загрузке.

- Доп. задание*
Vagrantfile, который сразу собирает систему с подключенным рейдом и смонтированными разделами. После перезагрузки стенда разделы должны автоматически примонтироваться.


**Выполнение:**

- Добавил ещё один диск в Vagrantfile:
```
                :sata5 => {
                        :dfile => './sata5.vdi',
                        :size => 250, # Megabytes
                        :port => 5
                }
```
- Сделал задание со * , скрипт для создания рейда + конф для автосборки рейда при загрузке внёс в Vagrantfile



```
            yum install -y mdadm smartmontools hdparm gdisk
            mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
            mdadm --create --verbose --force /dev/md0 -l 5 -n 5 /dev/sd{b,c,d,e,f}
            cat /proc/mdstat
            mkdir /etc/mdadm/
            echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
            mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
            parted -s /dev/md0 mklabel gpt
            parted /dev/md0 mkpart primary ext4 0% 20%
            parted /dev/md0 mkpart primary ext4 20% 40%
            parted /dev/md0 mkpart primary ext4 40% 60%
            parted /dev/md0 mkpart primary ext4 60% 80%
            parted /dev/md0 mkpart primary ext4 80% 100%
            for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
            mkdir -p /mnt/part{1..5}
            for i in $(seq 1 5); do mount /dev/md0p$i /mnt/part$i; done
            echo "#raid devices" >> /etc/fstab
            for i in $(seq 1 5); do echo `sudo blkid /dev/md0p$i | awk '{print $2}'` /mnt/part$i ext4 defaults 0 0 >> /etc/fstab; done
```
