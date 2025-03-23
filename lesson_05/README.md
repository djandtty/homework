1. Определить алгоритм с наилучшим сжатием:
Лучше всего gzip  
Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
`zfs set --help`
В выводе:
`compression     YES      YES   on | off | lzjb | gzip | gzip-[1-9] | zle | lz4 | zstd | zstd-[1-19] | zstd-fast | zstd-fast-[1-10,20,30,40,50,60,70,80,90,100,500,1000]`  

создать 4 файловых системы на каждой применить свой алгоритм сжатия;  
```
zpool create zp1 mirror /dev/sdb /dev/sdc
zpool create zp2 mirror /dev/sdd /dev/sde
zpool create zp3 mirror /dev/sdf /dev/sdg
zpool create zp4 mirror /dev/sdh /dev/sdi
zfs set compression=lzjb zp1
zfs set compression=lz4 zp2
zfs set compression=gzip zp3
zfs set compression=zle zp4
zfs get all | grep compressi
zp1  compression           lzjb                   local
zp2  compression           lz4                    local
zp3  compression           gzip                   local
zp4  compression           zle                    local
```
для сжатия использовать либо текстовый файл, либо группу файлов.  
```
zfs list
NAME    USED  AVAIL  REFER  MOUNTPOINT
zp1  26.4M   806M  26.3M  /zp1
zp2  18.5M   813M  18.4M  /zp2
zp3  11.4M   821M  11.2M  /zp3
zp4   139M   693M   139M  /zp4
```
3. Определить настройки пула.  
С помощью команды zfs import собрать pool ZFS.  
Командами zfs определить настройки:  
    - размер хранилища;  
    - тип pool;  
    - значение recordsize;  
    - какое сжатие используется;  
    - какая контрольная сумма используется.
```
zpool import -d zpoolexport/ otus

zpool list otus
NAME   SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus   480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -

zpool status otus
  pool: otus
 state: ONLINE
.......
          mirror-0 

zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local

zfs get compression otus
NAME  PROPERTY     VALUE           SOURCE
otus  compression  zle             local

zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
```

4. Работа со снапшотами:  
скопировать файл из удаленной директории;  
восстановить файл локально. zfs receive;  
найти зашифрованное сообщение в файле secret_message.  
```
wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download

zfs receive otus/question < otus_task2.file

find /otus/question -name "secret_message"
/otus/question/task1/file_mess/secret_message

cat /otus/question/task1/file_mess/secret_message
https://otus.ru/lessons/linux-hl/
```
