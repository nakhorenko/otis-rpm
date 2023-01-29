Ставим под рутом все необходимые пакеты 
```
[root@otusrpm vagrant]# yum install -y \
> redhat-lsb-core \
> wget \
> rpmdevtools \
> rpm-build \
> createrepo \
> yum-utils \
> gcc
```
Качаем пакет с nginx
```
[root@otusrpm vagrant]# wget https://nginx.org/packages/centos/8/SRPMS/nginx-1.20.2-1.el8.ngx.src.rpm
```
Далее качаем исходники пакета Openssl и распаковываем
```
[root@otusrpm vagrant]# wget https://github.com/openssl/openssl/archive/refs/heads/OpenSSL_1_1_1-stable.zip
[root@otusrpm vagrant]# ll
total 12692
-rw-r--r--. 1 root    root     1086865 ноя 16  2021 nginx-1.20.2-1.el8.ngx.src.rpm
-rw-r--r--. 1 root    root    11903781 янв 29 13:59 OpenSSL_1_1_1-stable.zip
drwxr-xr-x. 4 vagrant vagrant       34 янв 29 13:58 rpmbuild
```
```
[root@otusrpm vagrant]# unzip OpenSSL_1_1_1-stable.zip 
```
Обновляем/устанавливаем необходимые зависимости
```
[root@otusrpm vagrant]# yum-builddep rpmbuild/SPECS/nginx.spec
```
Создаем spec файл и меняем под себя
```
[root@otusrpm openssl-OpenSSL_1_1_1-stable]# touch nginx.spec
[root@otusrpm openssl-OpenSSL_1_1_1-stable]# nano nginx.spec
```
spec файл взят отсюда https://nginx.org/packages/centos/8/SRPMS/nginx-1.20.2-1.el8.ngx.src.rpm

И начинаем строить пакет nginx из исходников
```
[root@otusrpm vagrant]# rpmbuild -bb rpmbuild/SPECS/nginx.spec
Выполняется(%clean): /bin/sh -e /var/tmp/rpm-tmp.NnoTWB
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd nginx-1.20.2
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.20.2-1.el7.ngx.x86_64
+ exit 0
```
Дальше cd /root и 
```
[root@otusrpm ~]# ll rpmbuild/RPMS/x86_64/
total 2588
-rw-r--r--. 1 root root  808496 янв 29 14:08 nginx-1.20.2-1.el7.ngx.x86_64.rpm
-rw-r--r--. 1 root root 1836144 янв 29 14:08 nginx-debuginfo-1.20.2-1.el7.ngx.x86_64.rpm
```
Как видно, пакеты созданы

Сетапим nginx
```
[root@otusrpm x86_64]# yum localinstall -y /root/rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el7.ngx.x86_64.rpm
----------------------------------------------------------------------
  Verifying  : 1:nginx-1.20.2-1.el7.ngx.x86_64                                                                                                                                              1/1 

Installed:
  nginx.x86_64 1:1.20.2-1.el7.ngx 
```
Стартуем! (с)
```
[root@otusrpm x86_64]# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: http://nginx.org/en/docs/
[root@otusrpm x86_64]# systemctl start nginx
[root@otusrpm x86_64]# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Вс 2023-01-29 14:24:28 UTC; 1s ago
     Docs: http://nginx.org/en/docs/
  Process: 11463 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
 Main PID: 11464 (nginx)
   CGroup: /system.slice/nginx.service
           ├─11464 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
           └─11465 nginx: worker process

янв 29 14:24:28 otusrpm systemd[1]: Starting nginx - high performance web server...
янв 29 14:24:28 otusrpm systemd[1]: Can't open PID file /var/run/nginx.pid (yet?) after start: No such file or directory
янв 29 14:24:28 otusrpm systemd[1]: Started nginx - high performance web server.
```

По заданию, надо создать свой репозироий. Создаем в дефолтной директории nginx
```
[root@otusrpm x86_64]# mkdir /usr/share/nginx/html/repo
 ```
 Копируем туда собранный нами пакет и скачиваем перконовский rpm
 ```
  [root@otusrpm x86_64]# cp /root/rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el7.ngx.x86_64.rpm /usr/share/nginx/html/repo/
  [root@otusrpm x86_64]# wget https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm -O /usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
 ```
А дальше инициализируем свой репозиторий
```
[root@otusrpm repo]# createrepo /usr/share/nginx/html/repo/
Spawning worker 0 with 2 pkgs
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
```
В location / в файле /etc/nginx/conf.d/default.conf добавляем автоиндекс autoindex on; и делаем reload конфигурации nginx
```
[root@otusrpm repo]# nano /etc/nginx/conf.d/default.conf 
[root@otusrpm repo]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@otusrpm repo]# nginx -s reload
```
По ссылке можно увидеть 2 файла в нашем локальном репозитории
```
[root@otusrpm repo]# curl -L http://localhost/repo
<html>
<head><title>Index of /repo/</title></head>
<body>
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="repodata/">repodata/</a>                                          29-Jan-2023 14:32                   -
<a href="nginx-1.20.2-1.el7.ngx.x86_64.rpm">nginx-1.20.2-1.el7.ngx.x86_64.rpm</a>                  29-Jan-2023 14:26              808496
<a href="percona-orchestrator-3.2.6-2.el8.x86_64.rpm">percona-orchestrator-3.2.6-2.el8.x86_64.rpm</a>        16-Feb-2022 15:57             5222976
</pre><hr></body>
</html>
```
Осталось докинуть свой репозиторий в /etc/yum.repos.d/
```
[root@otusrpm repo]# cat >> /etc/yum.repos.d/otus.repo << EOF
> [otus]
> name=otus-linux
> baseurl=http://localhost/repo
> gpgcheck=0
> enabled=1
> EOF
```
```
[root@otusrpm repo]# cat /etc/yum.repos.d/otus.repo 
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
[root@otusrpm repo]# yum repolist enabled | grep otus
otus                                otus-linux                                 2
```
