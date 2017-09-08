---
layout: post
title:  "apache config"
date:   2017-06-15 16:47:53
categories: apache
---

### config proxy http to https

1. 在/etc/apache2/ports.conf 增加

Listen 8043

2.在/etc/apache2/ sites-available/scl.xxx.com
```c
<VirtualHost *:8043>
        ServerName 192.168.38.99 
        SSLProxyEngine On
        RequestHeader set Front-End-Https "On"
        CacheDisable *
        ProxyPass /api https://192.168.38.99/api
        ProxyPassReverse /api https://192.168.38.99/api
        RedirectMatch ^/$ 192.168.38.99/api
</VirtualHost>
```
http://www.giuseppeurso.eu/en/redirect-from-http-to-https-and-viceversa-with-apache-proxypass/






