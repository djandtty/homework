Выполнил в соответствии с описанием в методичке  
```
root@web:~# history
    1  date
    2  datetimectl
    3  timedatectl
    4  cat /etc/os-release
    5  timedatectl set-timezone Europe/Moscow
    6  timedatectl
    7  date
    8  apt update && apt install -y nginx
    9  systemctl status nginx
   10  ss -tln | grep 80
   11  nginx -v
   12  nano /etc/nginx/nginx.conf
   13  nginx -t
   14  systemctl restart nginx
   15  mv /var/www/html/index.nginx-debian.html /var/www/
```
```
root@log:~# history
    1  timedatectl set-timezone Europe/Moscow
    2  timedatectl
    3  date
    4  apt list rsyslog
    5  nano /etc/rsyslog.conf
    6  systemctl restart rsyslog
    7  ss -tuln
    8  cat /var/log/rsyslog/web/nginx_access.log
    9  cat /var/log/rsyslog/web/nginx_error.log
```
