Домашнее задание
Работа с LVM

Что нужно сделать?  

Настроить LVM в Ubuntu 24.04 Server  
Создать Physical Volume, Volume Group и Logical Volume  
   11  lvmdiskscan
   12  **pvcreate /dev/sdb**
   13  lvmdiskscan
   14  **vgcreate vege001 /dev/sdb**
   15  **lvcreate -l+80%FREE -n elve001 vege001**
   16  lvmdiskscan
   17  lsblk
   18  vgdisplay vege001
   19  vgdisplay -v vege001 | grep 'PV Name'
   20  lvdisplay /dev/vege001/elve001
   21  vgs
   22  lvs
   23  lvcreate -L50M -n elve001-new vege001
   24  lvs
   25  **mkfs.ext4 /dev/vege001/elve001**
   26  **mkdir /data**
   27  df -h
   28  lsblk
Отформатировать и смонтировать файловую систему  
   29  **mount /dev/vege001/elve001 /data/**
   30  lsblk
   31  df -h
Расширить файловую систему за счёт нового диска  
Выполнить resize  
   36  **vgextend vege001 /dev/sdc**
   37  vgs
   38  vgdisplay -v vege001 | grep 'PV Name'
   39  dd if=/dev/zero of=/data/test.log bs=1M count=800 status=progress
   40  df -Th /data/
   41  **lvextend -l+80%FREE /dev/vege001/elve001**
   42  df -Th /data/
   43  lvs
   44  lvs /dev/vege001/elve001
   45  **resize2fs /dev/vege001/elve001**
   46  df -Th /data/
Проверить корректность работы  
   47  **umount /data/**
   48  df -Th
   49  **mount /dev/vege001/elve001 /data/**
   50  df -Th

Не совсем понял, как проверить работоспособность, если речь про проверку superblock - то mount/unmount
