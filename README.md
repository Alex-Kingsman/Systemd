Systemd


создаём файл с конфигурацией
nano /etc/default/watchlog

cat /etc/default/watchlog
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log



Cоздаем /var/log/watchlog.log и пишем туда строки на своё усмотрение,
плюс ключевое слово ‘ALERT’

nano /var/log/watchlog.log

cat /var/log/watchlog.log
asd
asd
asd
asd
asd
as
dasd
ALERT



Cоздадим скрипт:
 cat /opt/watchlog.sh
#!/bin/bash

WORD=$1
LOG=$2
DATE=$(date)

if [ -z "$WORD" ] || [ -z "$LOG" ]; then
  echo "Usage: $0 <word> <logfile>"
  exit 1
fi

if grep -q "$WORD" "$LOG"
then
  logger "$DATE: I found word, Master!"
else
  exit 0
fi




Добавим права на запуск файла:

chmod +x /opt/watchlog.sh



Создадим юнит для сервиса:

cat  /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG


Создадим юнит для таймера:
cat /etc/systemd/system/watchlog.timer
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target


Запускаем timer:

systemctl start watchlog.timer

systemctl status watchlog.timer
● watchlog.timer - Run watchlog script every 30 second
     Loaded: loaded (/etc/systemd/system/watchlog.timer; enabled; preset: enabled)
     Active: active (elapsed) since Tue 2026-05-05 09:41:15 UTC; 24min ago
    Trigger: n/a
   Triggers: ● watchlog.service

May 05 09:41:15 srv systemd[1]: Stopping watchlog.timer - Run watchlog script every 30 second...
May 05 09:41:15 srv systemd[1]: Started watchlog.timer - Run watchlog script every 30 second.




tail -n 1000 /var/log/syslog  | grep word
2026-05-05T09:20:06.574009+00:00 srv systemd[1]: Started systemd-ask-password-console.path - Dispatch Password Requests to Console Directory Watch.
2026-05-05T09:20:06.574014+00:00 srv systemd[1]: systemd-ask-password-plymouth.path - Forward Password Requests to Plymouth Directory Watch was skipped because of an unmet condition check (ConditionPathExists=/run/plymouth/pid).
2026-05-05T09:20:06.574801+00:00 srv kernel: systemd[1]: Started systemd-ask-password-wall.path - Forward Password Requests to Wall Directory Watch.
2026-05-05T09:20:06.574936+00:00 srv kernel: audit: type=1400 audit(1777972806.002:5): apparmor="STATUS" operation="profile_load" profile="unconfined" name="1password" pid=529 comm="apparmor_parser"
2026-05-05T10:19:15.687472+00:00 srv alex: Tue May  5 10:19:15 AM UTC 2026: I found word, Master!
2026-05-05T10:19:45.102175+00:00 srv alex: Tue May  5 10:19:45 AM UTC 2026: I found word, Master!
2026-05-05T10:19:50.043657+00:00 srv root: Tue May  5 10:19:50 AM UTC 2026: I found word, Master!
2026-05-05T10:19:53.494093+00:00 srv alex: Tue May  5 10:19:53 AM UTC 2026: I found word, Master!
2026-05-05T10:24:34.157227+00:00 srv alex: Tue May  5 10:24:34 AM UTC 2026: I found word, Master!



Устанавливаем spawn-fcgi и необходимые для него пакеты:

apt install spawn-fcgi php php-cgi php-cli \ apache2 libapache2-mod-fcgid -y


создать файл с настройками

cat /etc/spawn-fcgi/fcgi.conf
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"


sudo systemctl status spawn-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
     Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; preset: enabled)
     Active: active (running) since Tue 2026-05-05 12:10:53 UTC; 10s ago
   Main PID: 10968 (php-cgi)
      Tasks: 33 (limit: 10267)
     Memory: 14.3M (peak: 15.0M)
        CPU: 25ms
     CGroup: /system.slice/spawn-fcgi.service
             ├─10968 /usr/bin/php-cgi
             ├─10969 /usr/bin/php-cgi
             ├─10970 /usr/bin/php-cgi
             ├─10971 /usr/bin/php-cgi
             ├─10972 /usr/bin/php-cgi
             ├─10973 /usr/bin/php-cgi
             ├─10974 /usr/bin/php-cgi
             ├─10975 /usr/bin/php-cgi
             ├─10976 /usr/bin/php-cgi
             ├─10977 /usr/bin/php-cgi
             ├─10978 /usr/bin/php-cgi
             ├─10979 /usr/bin/php-cgi
             ├─10980 /usr/bin/php-cgi
             ├─10981 /usr/bin/php-cgi
             ├─10982 /usr/bin/php-cgi
             ├─10983 /usr/bin/php-cgi
             ├─10984 /usr/bin/php-cgi
             ├─10985 /usr/bin/php-cgi
             ├─10986 /usr/bin/php-cgi
             ├─10987 /usr/bin/php-cgi
             ├─10988 /usr/bin/php-cgi
             ├─10989 /usr/bin/php-cgi
             ├─10990 /usr/bin/php-cgi
             ├─10991 /usr/bin/php-cgi
             ├─10992 /usr/bin/php-cgi
             ├─10993 /usr/bin/php-cgi
             ├─10994 /usr/bin/php-cgi
             ├─10995 /usr/bin/php-cgi
             ├─10996 /usr/bin/php-cgi
             ├─10997 /usr/bin/php-cgi
             ├─10998 /usr/bin/php-cgi
             ├─10999 /usr/bin/php-cgi
             └─11000 /usr/bin/php-cgi

May 05 12:10:53 srv systemd[1]: Started spawn-fcgi.service - Spawn-fcgi startup service by Otus.



Установим Nginx
 apt install nginx


оздадим новый Unit
/etc/systemd/system/nginx@.service


необходимо создать два файла конфигурации 

etc/nginx/nginx-first.conf


user www-data;
worker_processes auto;
pid /run/nginx-first.pid;

events {
    worker_connections 768;
}

http {

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access-first.log;
    error_log /var/log/nginx/error-first.log;

    server {
        listen 9001;

        location / {
            return 200 "OK\n";
        }
    }

    #include /etc/nginx/conf.d/*.conf;
    #include /etc/nginx/sites-enabled/*;
}




/etc/nginx/nginx-second.conf

user www-data;
worker_processes auto;
pid /run/nginx-second.pid;

events {
    worker_connections 768;
}

http {

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access-second.log;
    error_log /var/log/nginx/error-second.log;

    server {
        listen 9002;

        location / {
            return 200 "OK\n";
        }
    }

    #include /etc/nginx/conf.d/*.conf;
    #include /etc/nginx/sites-enabled/*;
}



systemctl start nginx@first
systemctl start nginx@second





ps afx | grep nginx


sudo ps afx | grep nginx
  11748 pts/0    T      0:00              \_ sudo systemctl status nginx@first.service
  11749 pts/1    Ss     0:00              |   \_ sudo systemctl status nginx@first.service
  11750 pts/1    T+     0:00              |       \_ systemctl status nginx@first.service
  11852 pts/0    T      0:00              \_ systemctl start nginx@first
  11986 pts/0    T      0:00              \_ sudo systemctl status nginx@second
  11987 pts/2    Ss     0:00              |   \_ sudo systemctl status nginx@second
  11988 pts/2    T+     0:00              |       \_ systemctl status nginx@second
  11998 pts/0    T      0:00              \_ sudo systemctl status nginx@first.service
  11999 pts/3    Ss     0:00              |   \_ sudo systemctl status nginx@first.service
  12000 pts/3    T+     0:00              |       \_ systemctl status nginx@first.service
  12012 pts/0    T      0:00              \_ sudo systemctl status nginx@second.service
  12013 pts/4    Ss     0:00              |   \_ sudo systemctl status nginx@second.service
  12014 pts/4    T+     0:00              |       \_ systemctl status nginx@second.service
  12103 pts/0    T      0:00              \_ sudo systemctl status nginx@first.service
  12104 pts/5    Ss     0:00              |   \_ sudo systemctl status nginx@first.service
  12105 pts/5    T+     0:00              |       \_ systemctl status nginx@first.service
  12115 pts/0    T      0:00              \_ sudo systemctl status nginx@second.service
  12116 pts/6    Ss     0:00              |   \_ sudo systemctl status nginx@second.service
  12117 pts/6    T+     0:00              |       \_ systemctl status nginx@second.service
  12127 pts/0    S+     0:00              \_ grep --color=auto nginx
  11904 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process on;
  11905 ?        S      0:00  \_ nginx: worker process
  11906 ?        S      0:00  \_ nginx: worker process
  11907 ?        S      0:00  \_ nginx: worker process
  11908 ?        S      0:00  \_ nginx: worker process
  12039 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on;
  12040 ?        S      0:00  \_ nginx: worker process
  12041 ?        S      0:00  \_ nginx: worker process
  12042 ?        S      0:00  \_ nginx: worker process
  12043 ?        S      0:00  \_ nginx: worker process
