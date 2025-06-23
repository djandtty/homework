Vagrantfile - приложен  

```
    1  sudo useradd otusadm && sudo useradd otus
    2  echo "otusadm:Otus2022!" | sudo chpasswd && echo "otus:Otus2022!" | sudo chpasswd
    3  sudo groupadd -f admin
    4  usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
    5  cat /etc/group | grep admin
    6  nano /usr/local/bin/login.sh
#!/bin/bash
#Первое условие: если день недели суббота или воскресенье
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
 #Второе условие: входит ли пользователь в группу admin
 if getent group admin | grep -qw "$PAM_USER"; then
        #Если пользователь входит в группу admin, то он может подключиться
        exit 0
      else
        #Иначе ошибка (не сможет подключиться)
        exit 1
    fi
  #Если день не выходной, то подключиться может любой пользователь
  else
    exit 0
fi
```
Добавим права на исполнение файла:  

    `7  chmod +x /usr/local/bin/login.sh`  
И дальше тестировал:  
```
    8  nano /etc/pam.d/sshd
#%PAM-1.0
auth       required     pam_exec.so debug /usr/local/bin/login.sh
auth       include      common-auth
account    include      common-account
password   include      common-password
session    include      common-session

    9  nano /usr/local/bin/login.sh
   10  nano /etc/pam.d/sshd
   11  tail -f /var/log/auth.log
   12  nano /usr/local/bin/login.sh
   13  nano /etc/pam.d/sshd
   14  cat /etc/os-release
   15  nano /etc/pam.d/sshd
   16  nano /usr/local/bin/login.sh
   17  tail -f /var/log/auth.log
   18  sudo systemctl restart sshd
   19  tail -f /var/log/auth.log
   20  nano /etc/pam.d/sshd
   21  sudo systemctl restart sshd
   22  nano /usr/local/bin/login.sh
   23  tail -f /var/log/auth.log
   24  nano /usr/local/bin/login.sh
   25  sudo systemctl restart sshd
   26  nano /usr/local/bin/login.sh
   27  sudo systemctl restart sshd
```
