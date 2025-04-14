1. ps ax  
Результат ДЗ - рабочий скрипт который можно запустить  
В выводе вижу:
```
ps ax
    PID TTY      STAT   TIME COMMAND
      1 ?        Ss     0:02 /sbin/init
```
PID - это числовые имена каталогов в /proc  
TTY - это из файла /proc/PID/stat - 16е поле. Если значение TTY равно 0, или если поле отсутствует или невозможно разрешить, это означает, что процесс не связан с каким-либо терминалом, и в выводе ps будет отображаться ?.  
STAT - это из файла /proc/PID/status - строка state  
TIME - это из файла /proc/1955/stat - utime (14-е поле): время, затраченное процессом в режиме пользователя, в тиках. stime (15-е поле): время, затраченное процессом в режиме ядра, в тиках. Формула = TIME = (utime + stime)/100.  
COMMAND - это из файла /proc/1955/cmdline  

Скрипт ps_ax.sh:  
```
#!/bin/bash

printf "PID\tTTY\tSTAT\tTIME\tCOMMAND\n"

# PID
PIDs=$(ls /proc/ | egrep [0-9] | sort -n)

for PID in ${PIDs[@]}; do


# TTY
  if [ -e "/proc/$PID/stat" ]; then
    STAT=($(sed 's/\([^)]+\)//' "/proc/$PID/stat"))
    TTY=${STAT[6]}
  fi

  if [ "$TTY" == 0 ]; then
    TTY="?"
  fi

# STAT
  if [ -e "/proc/$PID/status" ]; then
    STATE=$(cat /proc/$PID/status | grep "State" | awk '{print $2}')
  fi

# TIME
  if [ -e "/proc/$PID/stat" ]; then
    TICK_COUNT=$(getconf CLK_TCK)
    utime="${STAT[13]} / $TICK_COUNT"
    stime="${STAT[14]} / $TICK_COUNT"
    TEMP_TIME=$((utime + stime))

    minutes=$((TEMP_TIME / 60))
    seconds=$((TEMP_TIME % 60))

    TIME=$(printf "%02d:%02d" "$minutes" "$seconds")
  fi

# COMMAND
  if [ -e "/proc/$PID/cmdline" ]; then
    COMMAND=$(cat /proc/$PID/cmdline | tr -d '\0')
  fi

  if [ -z "$COMMAND" ]; then
    COMMAND=$(head -1 /proc/$PID/status | awk '{print "["$2"]"}')
  fi

# print
printf "%d\t%s\t%s\t%s\t%s\n" "$PID" "$TTY" "$STATE" "$TIME" "$COMMAND"

done
```

