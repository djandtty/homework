1. Включить отображение меню Grub.
```
nano /etc/default/grub
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=10
update-grub
```
После перезагрузки будет отображаться окно загрузчика  

2. Попасть в систему без пароля несколькими способами.  
В загрузчике выбираю Ubuntu - нажимаю E  
Ищу строку внизу linux...  
Изменяю ro на rw и добавляю запись init=/bin/bash  
Жму CTRL + X - идет загрузка и попадаю в систему под root  

3. Установить систему с LVM, после чего переименовать VG.  
```
root@ubuntu1:~# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg   1   1   0 wz--n- <23.00g 11.50g

root@ubuntu1:~# lvs
  LV        VG          Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-test -wi-ao---- <11.50g

root@ubuntu1:~# vgrename ubuntu-vg ubuntu-test
  Volume group "ubuntu-vg" successfully renamed to "ubuntu-test"

root@ubuntu1:~# nano /boot/grub/grub.cfg
```
Тут меняю записи ubuntu--vg на ubuntu--test и ubuntu--lv на ubuntu--testt
```
linux   /vmlinuz-6.8.0-55-generic root=/dev/mapper/ubuntu--test-ubuntu--testt ro recovery nomodeset dis_ucode_ldr

root@ubuntu1:~# vgs
  VG          #PV #LV #SN Attr   VSize   VFree
  ubuntu-test   1   1   0 wz--n- <23.00g 11.50g

root@ubuntu1:~# lvs
  LV           VG          Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-testt ubuntu-test -wi-ao---- <11.50g
```
