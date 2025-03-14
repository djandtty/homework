Домашнее задание: работа с mdadm  
Задание  
• Добавить в виртуальную машину несколько дисков  
• Собрать RAID-0/1/5/10 на выбор  
• Сломать и починить RAID  
• Создать GPT таблицу, пять разделов и смонтировать их в системе.  

На проверку отправьте:  
скрипт для создания рейда,   
отчет по командам для починки RAID и созданию разделов.  

1) **Создание рейда:**  
Не понял, зачем тут скрипт - raid10 создается 1 командой  
Из расчета, что у меня:  
```
user@test:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   25G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0 11.5G  0 lvm  /
sdb                         8:16   0    1G  0 disk
sdc                         8:32   0    1G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sdf                         8:80   0    1G  0 disk
sr0                        11:0    1 1024M  0 rom
```
**Выполнение:**  
`sudo mdadm --create --verbose /dev/md0 -l 10 -n 4 /dev/sd{b,c,d,e}`
3) **отчет по командам для починки RAID и созданию разделов.**  
Выводим зафейленный диск из рейда  
`sudo mdadm /dev/md0 --remove /dev/sdb`
Добавляем новый диск  
`sudo mdadm /dev/md0 --add /dev/sdf`
**Создание разделов:**  
```
sudo parted /dev/md0 mkpart primary ext4 0% 25%
sudo parted /dev/md0 mkpart primary ext4 25% 50%
sudo parted /dev/md0 mkpart primary ext4 50% 75%
sudo parted /dev/md0 mkpart primary ext4 75% 100%
for i in $(seq 1 4); do sudo mount /dev/md0p$i /mnt/0$i; done
```
