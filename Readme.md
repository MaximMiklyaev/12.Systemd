#Cкачиваем Vagrantfile, запускаем и подключаемся

          vagrant up
          vagrant ssh

#Установка всех пакетов и зависисмостей произойдет при поднятии Vagrantfile с помощью provision shell #Последующие действия будем выполнять из под root

          sudo -i

#Создаем файл конфигурации для сервиса

          touch /etc/sysconfig/watchlog

          nano /etc/sysconfig/watchlog

#Приводим в соответствие с помощью nano

          # Configuration file for my watchdog service
          # Place it to /etc/sysconfig
          # File and word in that file that we will be monit
          WORD="ALERT"
          LOG=/var/log/watchlog.log

#Создаем наш log файл с ключевым словом ALERT

          touch /var/log/watchlog.log

          nano /var/log/watchlog.log

#Создаем скрипт nano /opt/watchlog.sh

          nano /opt/watchlog.sh

#Вывод

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

#Делаем его исполняемым

          chmod +x /opt/watchlog.sh

#Создаем Unit файл сервиса

          nano /etc/systemd/system/watchlog.service

#Вывод

          [Unit]
          Description=My watchlog service
          [Service]
          Type=oneshot
          EnvironmentFile=/etc/sysconfig/watchlog
          ExecStart=/opt/watchlog.sh $WORD $LOG

#Создаем Unit файл таймера

          nano /etc/systemd/system/watchlog.timer

#Вывод

          [Unit]
          Description=Run watchlog script every 30 second
          [Timer]
          # Run every 30 second
          OnActiveSec=30sec
          Unit=watchlog.service
          [Install]
          WantedBy=multi-user.target

#Запускаем таймер

          systemctl enable watchlog.timer

          systemctl start watchlog.timer

#Проверяем работу

          tail -f /var/log/messages

#Вывод

          Feb 17 10:56:42 Systemd systemd[872]: Startup finished in 77ms.
          Feb 17 10:56:42 Systemd systemd[1]: Started User Manager for UID 1000.
          Feb 17 10:56:51 Systemd systemd[1]: Starting My watchlog service...
          Feb 17 10:56:51 Systemd root[914]: Wed Feb 17 10:56:51 UTC 2021: I found word, Master!
          Feb 17 10:56:51 Systemd systemd[1]: Started My watchlog service.
          Feb 17 10:59:12 Systemd systemd[872]: Starting Mark boot as successful...
          Feb 17 10:59:12 Systemd systemd[872]: Started Mark boot as successful.
          Feb 17 11:01:29 Systemd systemd[1]: Stopped Run watchlog script every 30 second.
          Feb 17 11:01:29 Systemd systemd[1]: Stopping Run watchlog script every 30 second.
          Feb 17 11:01:29 Systemd systemd[1]: Started Run watchlog script every 30 second.


#Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно так же называться 

#Устанавливаем

          yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y

#Правим файл /etc/sysconfig/spawn-fcgi

          nano /etc/sysconfig/spawn-fcgi

#Вывод

          # You must set some working options before the "spawn-fcgi" service will work.
          # If SOCKET points to a file, then this file is cleaned up by the init script.
          #
          # See spawn-fcgi(1) for all possible options.
          #
          # Example :
          SOCKET=/var/run/php-fcgi.sock
          OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"

#Создаем Unit файл сервиса

          nano /etc/systemd/system/spawn-fcgi.service

#Вывод

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

#Запускаем сервис и проверяем статус

          systemctl start spawn-fcgi
          systemctl status spawn-fcgi

#Вывод

● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-02-17 11:11:10 UTC; 8s ago
 Main PID: 1590 (php-cgi)
    Tasks: 33 (limit: 6113)
   Memory: 18.6M
   CGroup: /system.slice/spawn-fcgi.service
           ├─1590 /usr/bin/php-cgi
           ├─1592 /usr/bin/php-cgi
           ├─1593 /usr/bin/php-cgi
           ├─1594 /usr/bin/php-cgi
           ├─1595 /usr/bin/php-cgi
           ├─1596 /usr/bin/php-cgi
           ├─1597 /usr/bin/php-cgi
           ├─1598 /usr/bin/php-cgi
           ├─1599 /usr/bin/php-cgi
           ├─1600 /usr/bin/php-cgi
           ├─1601 /usr/bin/php-cgi
           ├─1602 /usr/bin/php-cgi
           ├─1603 /usr/bin/php-cgi
           ├─1604 /usr/bin/php-cgi
           ├─1605 /usr/bin/php-cgi
           ├─1606 /usr/bin/php-cgi
           ├─1607 /usr/bin/php-cgi
           ├─1608 /usr/bin/php-cgi
           ├─1609 /usr/bin/php-cgi
           ├─1610 /usr/bin/php-cgi
           ├─1611 /usr/bin/php-cgi
           ├─1612 /usr/bin/php-cgi
           ├─1613 /usr/bin/php-cgi
           ├─1614 /usr/bin/php-cgi
           ├─1615 /usr/bin/php-cgi
           ├─1616 /usr/bin/php-cgi
           ├─1617 /usr/bin/php-cgi
           ├─1618 /usr/bin/php-cgi
           ├─1619 /usr/bin/php-cgi
           ├─1620 /usr/bin/php-cgi
           ├─1621 /usr/bin/php-cgi
           ├─1622 /usr/bin/php-cgi
           └─1623 /usr/bin/php-cgi

Feb 17 11:11:10 Systemd systemd[1]: Started Spawn-fcgi startup service by Otus.

#Дополнить юнит-файл apache httpd возможностьб запустить несколько инстансов сервера с разными конфигами #Копируем Unit файл в Systemd

          cp /usr/lib/systemd/system/httpd.service /etc/systemd/system

#Переименовываем

          mv /etc/systemd/system/httpd.service /etc/systemd/system/httpd@.service

#Редактируем и приводим к соответствующему виду

          nano /etc/systemd/system/httpd@.service

#Вывод

          [Unit]
          Description=The Apache HTTP Server
          After=network.target remote-fs.target nss-lookup.target
          Documentation=man:httpd(8)
          Documentation=man:apachectl(8)
          [Service]

          Type=notify
          EnvironmentFile=/etc/sysconfig/httpd-%I
          ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
          ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
          ExecStop=/bin/kill -WINCH ${MAINPID}
          KillSignal=SIGCONT
          PrivateTmp=true

          [Install]
          WantedBy=multi-user.target

#Переходим в каталог с конфиругационными файлами

          cd /etc/httpd/conf

#Путем копирования создаем два конфигурационных файла

          scp httpd.conf first.conf
          scp httpd.conf second.conf

#Файл first.conf оставляем без изменений, во втором правим порт,а так же указываем путь до pid файла процесса

          nano second.conf

#Вывод

          ServerRoot "/etc/httpd"
          PidFile /var/run/httpd-second.pid
          #
          # Listen: Allows you to bind Apache to specific IP addresses and/or
          # ports, instead of the default. See also the <VirtualHost>
          # directive.
          #
          # Change this to Listen on specific IP addresses as shown below to
          # prevent Apache from glomming onto all bound IP addresses.
          #
          #Listen 12.34.56.78:80
          Listen 8080

#Переходим в sysconfig

          cd /etc/sysconfig/

#В самом файле окружения (которых будет два) задается опция для запуска веб-сервера с необходимым конфигурационным файлом

#

          nano /etc/sysconfig/httpd-first

#добовляем

          OPTIONS=-f conf/first.conf

#
          nano /etc/sysconfig/httpd-second

#добовляем

          OPTIONS=-f conf/second.conf


#Запускаем сервис с первым конфигурационным файлом и проверяем статус

          systemctl restart httpd@first.service

          systemctl status httpd@first.service

#Вывод

           httpd@first.service - The Apache HTTP Server
             Loaded: loaded (/etc/systemd/system/httpd@.service; disabled; vendor preset: disabled)
             Active: active (running) since Wed 2021-02-17 11:39:04 UTC; 5min ago
               Docs: man:httpd(8)
                     man:apachectl(8)
           Main PID: 1778 (httpd)
             Status: "Running, listening on: port 80"
              Tasks: 214 (limit: 6113)
             Memory: 26.9M
             CGroup: /system.slice/system-httpd.slice/httpd@first.service
                     ├─1778 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
                     ├─1779 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
                     ├─1780 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
                     ├─1781 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
                     ├─1782 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
                     └─1783 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND

          Feb 17 11:39:04 Systemd systemd[1]: Starting The Apache HTTP Server...
          Feb 17 11:39:04 Systemd httpd[1778]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globa>
          Feb 17 11:39:04 Systemd systemd[1]: Started The Apache HTTP Server.
          Feb 17 11:39:05 Systemd httpd[1778]: Server configured, listening on: port 80
          lines 1-21/21 (END)

#Запускаем сервис со вторым конфигурационным файлом и проверяем статус

          systemctl restart httpd@second.service

          systemctl status httpd@second.service

#Вывод

          ● httpd@second.service - The Apache HTTP Server
             Loaded: loaded (/etc/systemd/system/httpd@.service; disabled; vendor preset: disabled)
             Active: active (running) since Wed 2021-02-17 11:46:02 UTC; 8s ago
               Docs: man:httpd(8)
                     man:apachectl(8)
           Main PID: 2001 (httpd)
             Status: "Started, listening on: port 8080"
              Tasks: 214 (limit: 6113)
             Memory: 26.4M
             CGroup: /system.slice/system-httpd.slice/httpd@second.service
                     ├─2001 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
                     ├─2002 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
                     ├─2003 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
                     ├─2004 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
                     ├─2005 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
                     └─2006 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND

          Feb 17 11:46:02 Systemd systemd[1]: Starting The Apache HTTP Server...
          Feb 17 11:46:02 Systemd httpd[2001]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globa>
          Feb 17 11:46:02 Systemd systemd[1]: Started The Apache HTTP Server.
          Feb 17 11:46:02 Systemd httpd[2001]: Server configured, listening on: port 8080
          lines 1-21/21 (END)

#Так же проверяем порты на которых работают сервисы

          ss -tnlp

#Вывод

          State                  Recv-Q                  Send-Q                                    Local Address:Port                                    Peer Address:Port                  
          LISTEN                 0                       128                                             0.0.0.0:22                                           0.0.0.0:*                      users:(("sshd",pid=650,fd=5))
          LISTEN                 0                       128                                             0.0.0.0:111                                          0.0.0.0:*                      users:(("rpcbind",pid=563,fd=4),("systemd",pid=1,fd=91))
          LISTEN                 0                       128                                                [::]:22                                              [::]:*                      users:(("sshd",pid=650,fd=7))
          LISTEN                 0                       128                                                [::]:111                                             [::]:*                      users:(("rpcbind",pid=563,fd=6),("systemd",pid=1,fd=93))
          LISTEN                 0                       128                                                   *:8080                                               *:*                      users:(("httpd",pid=2006,fd=4),("httpd",pid=2005,fd=4),("httpd",pid=2004,fd=4),("httpd",pid=2003,fd=4),("httpd",pid=2001,fd=4))
          LISTEN                 0                       128                                                   *:80                                                 *:*                      users:(("httpd",pid=1783,fd=4),("httpd",pid=1782,fd=4),("httpd",pid=1781,fd=4),("httpd",pid=1780,fd=4),("httpd",pid=1778,fd=4))
                                