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



