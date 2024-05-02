# sysd
Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig
```
Сконфигурируем Vagrantfile
Создадим synced_folder "shfile" для гостевой машины. Для того чтобы в дальнейшем разместить в ней скрипты и копировать их.
Создадим и Поместим необходимые скрипты для дальнейшего использования в папку /home/sysd/scripts
```
Создаём файл с конфигурацией для сервиса
```
watchlog
# Configuration file for my watchlog service
# Place it to /etc/sysconfig

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```
Затем создаем watchlog.log
и пишем туда строки на своё усмотрение,
плюс ключевое слово ‘ALERT’

```
Создадим скрипт:
watchlog.sh
#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi
```
```
Добавим права на запуск файла(указываем сразу в Vagrant):
chmod +x /opt/*.sh

Создадим юнит для сервиса watchlog.service: 
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
```
Создадим юнит для таймера watchlog.timer :
```
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.targettart	
```

Затем достаточно только стартануть timer:
Добавляем исполнение в Vagrantfile, разворачиваем машину, смотрим результат:

И убедиться в результате:
```
[root@sysd vagrant]# tail -f /var/log/messages
May  2 22:09:12 centos8 systemd[1]: Started My watchlog service.
May  2 22:09:12 centos8 systemd[1]: Stopping User runtime directory /run/user/0...
May  2 22:09:12 centos8 systemd[1]: run-user-0.mount: Succeeded.
May  2 22:09:12 centos8 systemd[1]: user-runtime-dir@0.service: Succeeded.
May  2 22:09:12 centos8 systemd[1]: Stopped User runtime directory /run/user/0.
May  2 22:09:12 centos8 systemd[1]: Removed slice User Slice of UID 0.
May  2 22:09:48 centos8 systemd[1]: Starting My watchlog service...
May  2 22:09:48 centos8 root[5978]: Thu May  2 22:09:48 UTC 2024: I found word, Master!
May  2 22:09:48 centos8 systemd[1]: watchlog.service: Succeeded.
```
```
Устанавливаем spawn-fcgi и необходимые для него пакеты (Также предустанавливаем в Vagrantfile):
yum install epel-release -y && yum install spawn-fcgi php php-cli
mod_fcgid httpd -y
```
```
раскоментировать 2 последние строки в файле vi /etc/sysconfig/spawn-fcgi

# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi"
```
```
Cам юнит файл будет примерно следующего вида:
[root@sysd vagrant]# vi /etc/systemd/system/spawn-fcgi.service

[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
```
Убеждаемся, что все успешно работает:
```
Предварительно сделано в Vagrantfile
[root@sysd vagrant]# systemctl start spawn-fcgi
[root@sysd vagrant]# systemctl status spawn-fcgi
```
```
[root@sysd vagrant]# systemctl status spawn-fcgi
● spawn-fcgi.service - LSB: Start and stop FastCGI processes
   Loaded: loaded (/etc/rc.d/init.d/spawn-fcgi; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2024-05-02 22:58:24 UTC; 11s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 6052 ExecStart=/etc/rc.d/init.d/spawn-fcgi start (code=exited, status=0/SUCCESS)
    Tasks: 33 (limit: 11402)
   Memory: 18.3M
   CGroup: /system.slice/spawn-fcgi.service
           ├─6066 /usr/bin/php-cgi
           ├─6067 /usr/bin/php-cgi
           ├─6068 /usr/bin/php-cgi
           ├─6069 /usr/bin/php-cgi
           ├─6070 /usr/bin/php-cgi
           ├─6071 /usr/bin/php-cgi
           ├─6072 /usr/bin/php-cgi
           ├─6073 /usr/bin/php-cgi
           ├─6074 /usr/bin/php-cgi
           ├─6075 /usr/bin/php-cgi
           ├─6076 /usr/bin/php-cgi
           ├─6077 /usr/bin/php-cgi
           ├─6078 /usr/bin/php-cgi
           ├─6079 /usr/bin/php-cgi
           ├─6080 /usr/bin/php-cgi
           ├─6081 /usr/bin/php-cgi
           ├─6082 /usr/bin/php-cgi
           ├─6083 /usr/bin/php-cgi
           ├─6084 /usr/bin/php-cgi
           ├─6085 /usr/bin/php-cgi
           ├─6086 /usr/bin/php-cgi
           ├─6087 /usr/bin/php-cgi
           ├─6088 /usr/bin/php-cgi
           ├─6089 /usr/bin/php-cgi
           ├─6090 /usr/bin/php-cgi
           ├─6091 /usr/bin/php-cgi
           ├─6092 /usr/bin/php-cgi
           ├─6093 /usr/bin/php-cgi
           ├─6094 /usr/bin/php-cgi
           ├─6095 /usr/bin/php-cgi
           ├─6096 /usr/bin/php-cgi
           ├─6097 /usr/bin/php-cgi
           └─6098 /usr/bin/php-cgi

May 02 22:58:24 sysd systemd[1]: Starting LSB: Start and stop FastCGI processes...
May 02 22:58:24 sysd spawn-fcgi[6052]: Starting spawn-fcgi: [  OK  ]
May 02 22:58:24 sysd systemd[1]: Started LSB: Start and stop FastCGI processes.

```
```
Для запуска нескольких экземпляров сервиса будем использовать шаблон в
конфигурации файла окружения (/usr/lib/systemd/system/httpd.service ):
Добавлены в Vagrantfile
httpd-@first.service
httpd@second.service
```
```
Проверяем статус:
[root@sysd vagrant]# systemctl status httpd@first.service
● httpd@first.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@first.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2024-05-02 22:54:33 UTC; 9min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 5407 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 214 (limit: 11402)
   Memory: 27.5M
   CGroup: /system.slice/system-httpd.slice/httpd@first.service
           ├─5407 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─5413 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─5414 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─5415 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─5416 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           └─5417 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND

May 02 22:54:33 sysd systemd[1]: Starting The Apache HTTP Server...
May 02 22:54:33 sysd httpd[5407]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
May 02 22:54:33 sysd systemd[1]: Started The Apache HTTP Server.
May 02 22:54:33 sysd httpd[5407]: Server configured, listening on: port 80
[root@sysd vagrant]# systemctl status httpd@second.service
● httpd@second.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@second.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2024-05-02 22:54:33 UTC; 9min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 5418 (httpd)
   Status: "Running, listening on: port 8080"
    Tasks: 214 (limit: 11402)
   Memory: 26.7M
   CGroup: /system.slice/system-httpd.slice/httpd@second.service
           ├─5418 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─5630 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─5631 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─5632 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─5634 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           └─5635 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND

May 02 22:54:33 sysd systemd[1]: Starting The Apache HTTP Server...
May 02 22:54:33 sysd httpd[5418]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
May 02 22:54:33 sysd systemd[1]: Started The Apache HTTP Server.
May 02 22:54:33 sysd httpd[5418]: Server configured, listening on: port 8080
```


















