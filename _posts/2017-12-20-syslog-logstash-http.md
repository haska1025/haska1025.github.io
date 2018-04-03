---
layout: post
title:  "logstash from syslog-ng to http"
date:   2017-12-20 16:47:53
categories: syslog-ng
---
### About

通过syslog-ng OSE采集服务log，推送给logstash, 然后再推送给http服务。

系统：ubuntu 14.04

syslog-ng : ubuntu 14.04发行版自带，3.5.3

logstash: 6.1.0

http服务：nginx + uwsgi + django

### syslog-ng安装部署

1. 用发型版自带即可，

   sudo apt-get install syslog-ng-core

2. 在/etc/syslog-ng/conf.d下面增加一个配置文件tang-iostat-logstash.conf，内容如下

```c
destination d_tang_iostat_logstash_tcp {
    network("192.168.32.212" port(5000));
};

log { source(tang_src);  filter{match("iostat" value("MESSAGE"));};  destination(d_tang_iostat_logstash_tcp); };
```

### logstash安装部署

1. jdk安装

logstash需要java 1.8 ,ubuntu 14.04自带openjdk是1.7，为此需要通过ppa来安装。

方法：

```c
sudo add-apt-repository ppa:webupd8team/java -y
sudo apt-get update
sudo apt-get install oracle-java8-installer
To automatically set up the Java 8 environment variables

sudo apt-get install oracle-java8-set-default
Check it

java -version
```

参考链接：https://askubuntu.com/questions/464755/how-to-install-openjdk-8-on-14-04-lts

2. logstash安装，参考官网document

https://www.elastic.co/guide/en/logstash/current/installing-logstash.html

3. logstash的启动停止

initctl start logstash

initctl stop logstash

参考:https://www.elastic.co/guide/en/logstash/current/running-logstash.html

### http服务demo

#### nginx

1. 安装

直接apt-get install 安装发行版自带即可。

参考：http://nginx.org/en/docs/beginners_guide.html

2. uwsgi配置

在 /etc/nginx/conf.d/ 增加文件django.conf, 内容如下：

```c
upstream django {
        server 127.0.0.1:9000;
}
```

修改/etc/nginx/sites-available/default,将请求转到django

```c
location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        #try_files $uri $uri/ =404;
        # Uncomment to enable naxsi on this location
        # include /etc/nginx/naxsi.rules
        include /etc/nginx/uwsgi_params;
        uwsgi_pass django; 
}
```

重新加载nginx配置文件, nginx -s reload

参考：https://www.nginx.com/resources/admin-guide/gateway-uwsgi-django/

#### uWSGI

##### install

参考官网document：http://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html

##### 在ubuntu服务化启动uWSGI服务

1. 给upstart 增加启动项
 
增加文件/etc/init/uwsgi.conf，内容如下：

```c
# Emperor uWSGI script

description "uWSGI Emperor"
start on runlevel [2345]
stop on runlevel [06]

respawn

exec uwsgi --master --die-on-term --emperor /etc/uwsgi/apps-enabled
```

2. 给django app增加配置文件

增加文件/etc/uwsgi/apps-enabled/upload_uwsgi.ini，内容如下：

```c
[uwsgi]
socket = 127.0.0.1:9000
# the base directory (full path)
chdir           = /var/www/nginx-transtest/mysite

# Django s wsgi file
module          = mysite.wsgi

# process-related settings
# master

# maximum number of worker processes
processes       = 4
threads = 2
```

3. 启动服务

initctl start uwsgi

参考：http://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html

#### django

django 是一个python web框架，具体参考官方文档：

https://docs.djangoproject.com/en/1.11/topics/install/

https://www.howtoforge.com/tutorial/how-to-install-django-on-ubuntu/



