1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default)  
Буду использовать файл лога `/var/log/auth.log`  
И ключевое слово `failure`  
Создаю конфиг для обозначения этих переменных:  
```
sudo nano /etc/default/logcheck.conf
WORD="failure"
FILE="/var/log/auth.log"
```
Создаю скрипт для поиска и записи в лог при нахождении:  
```
sudo nano /usr/local/bin/logcheck.sh

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


