1. Запустить nginx на нестандартном порту 3-мя разными способами  
Способ 1:  
Находим в /var/log/audit.log информацию о блокировании порта:  
```
[root@selinux ~]# cat /var/log/audit/audit.log | grep 4881
type=AVC msg=audit(1748605526.330:718): avc:  denied  { name_bind } for  pid=6411 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
```
Отправляем эту строчку утилите audit2why:  
```
[root@selinux ~]# grep 1748605526.330:718 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1748605526.330:718): avc:  denied  { name_bind } for  pid=6411 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0

        Was caused by:
        The boolean nis_enabled was set incorrectly.
        Description:
        Allow nis to enabled

        Allow access by executing:
        # setsebool -P nis_enabled 1
```
Утилита audit2why показывает почему трафик блокируется. Исходя из вывода утилиты, мы видим, что нам нужно поменять параметр nis_enabled. Включим параметр nis_enabled и перезапустим nginx:  
```
[root@selinux ~]# setsebool -P nis_enabled 1
[root@selinux ~]# systemctl restart nginx
```
Видим что nginx работает.  
```
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Fri 2025-05-30 11:50:02 UTC; 7s ago
    Process: 8856 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 8857 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 8858 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 8859 (nginx)
```
Отключить параметр:  
```
[root@selinux ~]# getsebool -a | grep nis_enabled
nis_enabled --> on
[root@selinux ~]# setsebool -P nis_enabled off
```
Способ 2: Добавление нестандартного порта в имеющийся тип
Не можем запустить nginx  
```
[root@selinux ~]# systemctl restart nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
```
Ищем имеющийся тип для http траффика:  
```
[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
```
Добавим порт в тип http_port_t:  
```
[root@selinux ~]# semanage port -a -t http_port_t -p tcp 4881
Port tcp/4881 already defined, modifying instead
[root@selinux ~]# semanage port -l | grep  http_port_t
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
```
Проверяем работу nginx:  
```
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Fri 2025-05-30 11:52:53 UTC; 48s ago
```
Способ 3 Формирование и установка модуля SELinux:  
Воспользуемся утилитой audit2allow для того, чтобы на основе логов SELinux сделать модуль, разрешающий работу nginx на нестандартном порту:
```
grep nginx /var/log/audit/audit.log | audit2allow -M nginx
******************** IMPORTANT ***********************
To make this policy package active, execute:

semodule -i nginx.pp
```
Audit2allow сформировал модуль, и сообщил нам команду, с помощью которой можно применить данный модуль:  
`[root@selinux ~]# semodule -i nginx.pp`  
Проверяем работу nginx:  
```
[root@selinux ~]# systemctl start nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Fri 2025-05-30 11:55:57 UTC; 3s ago
```
2. Обеспечить работоспособность приложения при включенном selinux
Не получается внести изменения:  
```
nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab   
> update add www.ddns.lab. 60 A 192.168.50.15
> send
update failed: SERVFAIL
> quit
```
Смотрим логи SELinux на клиенте - их нет:  
`cat /var/log/audit/audit.log | audit2why`  
Смотрим логи SELinux на сервере:  
```
cat /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1715608488.544:2949): avc:  denied  { create } for  pid=6970 comm="isc-worker0000" name="named.ddns.lab.view1.jnl" scontext=system_u:system_r:named_t:s0 tcontext=system_u:object_r:etc_t:s0 tclass=file permissive=0

	Was caused by:
		Missing type enforcement (TE) allow rule.

		You can use audit2allow to generate a loadable module to allow this access.
```
Ошибка в контексте безопасности. Вместо типа named_t используется тип etc_t  
Смотрим какой контекст у каталога с зонами по умолчанию:  
```
semanage fcontext -l | grep named
... 
/var/named(/.*)?                                   all files          system_u:object_r:__named_zone_t__:s0 
...
```
Меняем контекс каталога /etc/named на named_zone_t:  
```
chcon -R -t named_zone_t /etc/named
ls -laZ /etc/named
drw-rwx---. root named system_u:object_r:named_zone_t:s0 .
drwxr-xr-x. root root  system_u:object_r:etc_t:s0       ..
drw-rwx---. root named unconfined_u:object_r:named_zone_t:s0 dynamic
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.50.168.192.rev
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.dns.lab
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.dns.lab.view1
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.newdns.lab
```
Тперь изменения вносятся успешно  


