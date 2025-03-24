Основная часть:  
запустить 2 виртуальных машины (сервер NFS и клиента);  
на сервере NFS должна быть подготовлена и экспортирована директория;   
в экспортированной директории должна быть поддиректория с именем upload с правами на запись в неё;   
экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины (systemd, autofs или fstab — любым способом);  
монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3.  
Для самостоятельной реализации:   
настроить аутентификацию через KERBEROS с использованием NFSv4.  
  
На сервере:  
```
apt install nfs-kernel-server
cat /etc/nfs.conf
ss -tnplu
mkdir -p /mnt/les06/upload
ls -alh /mnt/
ls -alh /mnt/les06/
chown -R nobody:nogroup /mnt/les06/
chmod 0777 /mnt/les06/upload/
ls -alh /mnt/les06/
ls -alh /mnt/
nano /etc/exports
          /mnt/les06 192.168.1.91/32(rw,sync,root_squash)
cat /etc/exports
exportfs -r
exportfs -s
cd /mnt/les06/upload/
ls
exportfs -s
showmount -a 192.168.1.92
```

На клиенте:  
```
apt install nfs-common
vi /etc/fstab
      192.168.1.92:/mnt/les06/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0
systemctl daemon-reload
systemctl restart remote-fs.target
mount | grep mnt
ls /mnt/
cd /mnt/upload/
showmount -a 192.168.1.92
```


