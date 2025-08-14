Устанавливаем Borg Backup на сервер и клиент:  
```
apt update  
apt install borgbackup
```
Подготовим и примонтируем дополнительный диск для хранения бэкапов:  
```
root@backupServer:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0   87M  1 loop /snap/lxd/28373
loop1    7:1    0 38.8M  1 loop /snap/snapd/21759
loop2    7:2    0 63.9M  1 loop /snap/core20/2318
sda      8:0    0   40G  0 disk 
└─sda1   8:1    0   40G  0 part /
sdb      8:16   0   10M  0 disk 
sdc      8:32   0    2G  0 disk 
root@backupServer:~# mkfs.ext4 /dev/sdc
root@backupServer:~# echo "`blkid | grep sdc | awk '{print $2}'` /var/backup ext4 defaults 0 0" >> /etc/fstab
root@backupServer:~# mount -a
root@backupServer:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           204M  888K  145M   1% /run
/dev/sda1        40G  1.7G   30G   5% /
tmpfs           718M     0  718M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
vagrant         91G   8G  82G  10% /vagrant
tmpfs           144M  4.0K  144M   1% /run/user/1000
/dev/sdc        1.9G   24K  1.8G   1% /var/backup
```
На сервере backup создаем пользователя и каталог /var/backup  
```
root@backupServer:~# mkdir /var/backup
root@backupServer:~# useradd -m -d /var/backup borg
root@backupServer:~# chown borg:borg /var/backup/
```
Для аутентификации удаленных клиентов - SSH-ключи. Cоздадим нужную структуру папок и файлов:
```
root@backupServer:~# su - borg
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ chmod 700 .ssh
$ chmod 600 .ssh/authorized_keys
```
Генерируем на клиенте пару ключей:  
```
root@client:~# ssh-keygen
root@client:~# cat .ssh/id_rsa.pub
```
На сервере добавим открытый ключ клиента в /var/backup/.ssh/authorized_keys  
Клиент может подключится к серверу по ssh:
`root@client:~# ssh borg@192.168.56.160`  
Дальше на клиенте:  
Настраиваем бэка. Инициализируем репозиторий borg на backup сервере с client сервера:  
`root@client:~# borg init --encryption=repokey borg@192.168.56.160:my_repo`  
Запускаем для проверки создания бэкапа:  
`root@client:~# borg create --stats --list borg@192.168.56.160:my_repo::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc`
Посмотреть информацию по бэкапам:  
`root@client:~# borg list borg@192.168.56.160:my_repo`
Посмотреть список файлов в бэкапе:  
`root@client:~# borg list borg@192.168.56.160:my_repo::etc-2025-08-08_10:34:41`  
Восстановить из резевной копии:  
`root@client:~# borg extract borg@192.168.56.160:my_repo::etc-2025-08-08_10:34:41 etc/hostname`  
Автоматизируем создание бэкапов с помощью systemd. Создаем сервис и таймер в каталоге /etc/systemd/system/  
```
root@client:~# vim /etc/systemd/system/borg-backup.service

[Unit]
Description=Borg Backup

[Service]
Type=oneshot

# Парольная фраза
Environment="BORG_PASSPHRASE=1111"
# Репозиторий
Environment=REPO=borg@192.168.56.160:my_repo
# Что бэкапим
Environment=BACKUP_TARGET=/etc

# Создание бэкапа
ExecStart=/bin/borg create \
    --stats                \
    ${REPO}::etc-{now:%%Y-%%m-%%d_%%H:%%M:%%S} ${BACKUP_TARGET}

# Проверка бэкапа
ExecStart=/bin/borg check ${REPO}

# Очистка старых бэкапов
ExecStart=/bin/borg prune \
    --keep-daily  90      \
    --keep-monthly 12     \
    --keep-yearly  1       \
    ${REPO}
root@client:~# vim /etc/systemd/system/borg-backup.timer

[Unit]
Description=Borg Backup

[Timer]
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
```
Включаем и запускаем службу таймера:  
```
root@client:~# systemctl enable borg-backup.timer
Created symlink /etc/systemd/system/timers.target.wants/borg-backup.timer → /etc/systemd/system/borg-backup.timer.
root@client:~# systemctl start borg-backup.timer
root@client:~# systemctl start borg-backup.service
```
Проверяем работу таймера:  
`root@client:~# systemctl list-timers --all`
Логи смотрим в systemd journal или в /var/log/syslog  
