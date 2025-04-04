1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default)  
Буду использовать файл лога `/var/log/auth.log`  
И ключевое слово `failure`  
Создаю конфиг для обозначения этих переменных:  
```
sudo vi /etc/default/logcheck.conf
WORD="failure"
FILE="/var/log/auth.log"
```
Создаю скрипт для поиска и записи в лог при нахождении:  
```
sudo vi /usr/local/bin/logcheck.sh

#!/bin/bash
source /etc/default/logcheck.conf
if grep -q "$WORD" "$FILE"; then
  logger "Keyword '$WORD' found in log $FILE."
fi
```
Задаю право на исполнение:  
`sudo chmod +x /usr/local/bin/logcheck.sh`  
Создаю сервис:  
```
sudo systemctl edit --force --full logcheck.service
Successfully installed edited file '/etc/systemd/system/logcheck.service'.

[Unit]
Description=Test Service
[Service]
Type=oneshot
EnvironmentFile=/etc/default/logcheck.conf
ExecStart=/usr/local/bin/logcheck.sh
```
Создаю таймер для этого сервиса:  
```
sudo systemctl edit --force --full logcheck.timer
Successfully installed edited file '/etc/systemd/system/logcheck.timer'.

[Unit]
Description=Test Timer to Test Service
[Timer]
OnBootSec=30s
OnUnitActiveSec=30s
Unit=logcheck.service
[Install]
WantedBy=timers.target
```
Проверяю скрипт:  
```
sudo /usr/local/bin/logcheck.sh
cat /var/log/syslog | grep failure
2025-04-02T07:26:06.086569+00:00 ubu2 root: Keyword 'failure' found in log /var/log/auth.log.
```
И проверяю статус служб:  
```
user@ubu2:~$ sudo systemctl status logcheck.service
○ logcheck.service - Test Service
     Loaded: loaded (/etc/systemd/system/logcheck.service; static)
     Active: inactive (dead)
user@ubu2:~$ sudo systemctl status logcheck.timer
○ logcheck.timer - Test Timer to Test Service
     Loaded: loaded (/etc/systemd/system/logcheck.timer; disabled; preset: enabled)
     Active: inactive (dead)
    Trigger: n/a
   Triggers: ● logcheck.service
```
Загружаю новые сервисы и запускаю таймер:  
```
sudo systemctl daemon-reload
sudo systemctl start logcheck.timer
```
Провеяю сработки:  
```
cat /var/log/syslog | grep failure
2025-04-02T07:26:06.086569+00:00 ubu2 root: Keyword 'failure' found in log /var/log/auth.log.
2025-04-02T07:28:41.024446+00:00 ubu2 root: Keyword 'failure' found in log /var/log/auth.log.
2025-04-02T07:29:15.623834+00:00 ubu2 root: Keyword 'failure' found in log /var/log/auth.log.
```
Проверяю статусы служб:  
```
sudo systemctl status logcheck.service
○ logcheck.service - Test Service
     Loaded: loaded (/etc/systemd/system/logcheck.service; static)
     Active: inactive (dead) since Wed 2025-04-02 07:29:49 UTC; 12s ago
TriggeredBy: ● logcheck.timer
    Process: 2421 ExecStart=/usr/local/bin/logcheck.sh (code=exited, status=0/SUCCESS)
   Main PID: 2421 (code=exited, status=0/SUCCESS)
        CPU: 12ms

Apr 02 07:29:49 ubu2 systemd[1]: Starting logcheck.service - Test Service...
Apr 02 07:29:49 ubu2 root[2424]: Keyword 'failure' found in log /var/log/auth.log.
Apr 02 07:29:49 ubu2 systemd[1]: logcheck.service: Deactivated successfully.
Apr 02 07:29:49 ubu2 systemd[1]: Finished logcheck.service - Test Service.
```
И таймера:  
```
sudo systemctl status logcheck.timer
● logcheck.timer - Test Timer to Test Service
     Loaded: loaded (/etc/systemd/system/logcheck.timer; disabled; preset: enabled)
     Active: active (waiting) since Wed 2025-04-02 07:28:41 UTC; 2s ago
    Trigger: Wed 2025-04-02 07:29:11 UTC; 27s left
   Triggers: ● logcheck.service

Apr 02 07:28:41 ubu2 systemd[1]: Started logcheck.timer - Test Timer to Test Service.
```


2. Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).


3. Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

Установил nginx:  
`sudo apt install nginx`
Редактирую unit:  
```sudo vi /etc/systemd/system/nginx@.service
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx-%I.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx-first.conf
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx-second.conf
И редактирую их
```
sudo vi /etc/nginx/nginx-first.conf

user www-data;
worker_processes auto;
pid /run/nginx-first.pid;
#pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;
events {
        worker_connections 768;
        # multi_accept on;
}
http {
        server {
                listen 9001;
        }
        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        include /etc/nginx/mime.types;
        default_type application/octet-stream;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;
        access_log /var/log/nginx/access.log;
        gzip on;
        include /etc/nginx/conf.d/*.conf;
#       include /etc/nginx/sites-enabled/*;
}

sudo vi /etc/nginx/nginx-second.conf

user www-data;
worker_processes auto;
pid /run/nginx-second.pid;
#pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;
events {
        worker_connections 768;
        # multi_accept on;
}
http {
        server {
                listen 9002;
        }
        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        include /etc/nginx/mime.types;
        default_type application/octet-stream;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;
        access_log /var/log/nginx/access.log;
        gzip on;
        include /etc/nginx/conf.d/*.conf;
#       include /etc/nginx/sites-enabled/*;
}
```
Делаю проверку и запускаю:  
```
nginx -t
sudo systemctl start nginx@first
sudo systemctl start nginx@second

Проверяю статус:
```
user@ubu2:/etc/nginx$ sudo systemctl status nginx@first.service
● nginx@first.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; preset: enabled)
     Active: active (running) since Fri 2025-04-04 15:12:55 UTC; 53min ago
       Docs: man:nginx(8)
    Process: 1794 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-first.conf -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 1798 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 1800 (nginx)
      Tasks: 2 (limit: 2272)
     Memory: 1.7M (peak: 1.9M)
        CPU: 18ms
     CGroup: /system.slice/system-nginx.slice/nginx@first.service
             ├─1800 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process on;"
             └─1801 "nginx: worker process"

Apr 04 15:12:55 ubu2 systemd[1]: Starting nginx@first.service - A high performance web server and a reverse proxy server...
Apr 04 15:12:55 ubu2 systemd[1]: Started nginx@first.service - A high performance web server and a reverse proxy server.
user@ubu2:/etc/nginx$ sudo systemctl status nginx@second.service
● nginx@second.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; preset: enabled)
     Active: active (running) since Fri 2025-04-04 15:21:55 UTC; 44min ago
       Docs: man:nginx(8)
    Process: 2005 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-second.conf -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 2007 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 2008 (nginx)
      Tasks: 2 (limit: 2272)
     Memory: 1.7M (peak: 1.9M)
        CPU: 12ms
     CGroup: /system.slice/system-nginx.slice/nginx@second.service
             ├─2008 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on;"
             └─2009 "nginx: worker process"

Apr 04 15:21:55 ubu2 systemd[1]: Starting nginx@second.service - A high performance web server and a reverse proxy server...
Apr 04 15:21:55 ubu2 systemd[1]: Started nginx@second.service - A high performance web server and a reverse proxy server.
```
Смотрю, какие порты слушаются:  
```
ss -tnulp | grep 900
tcp   LISTEN 0      511                0.0.0.0:9001       0.0.0.0:*
tcp   LISTEN 0      511                0.0.0.0:9002       0.0.0.0:*
```
Смотрю, список процессов:  
```
ps afx | grep nginx
   9027 pts/0    S+     0:00  |           \_ grep --color=auto nginx
   1800 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process on;
   1801 ?        S      0:00  \_ nginx: worker process
   2008 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on;
   2009 ?        S      0:00  \_ nginx: worker process
```
