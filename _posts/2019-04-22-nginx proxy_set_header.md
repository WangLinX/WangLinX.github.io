---
layout:     post
title:      nginx proxy_set_header
subtitle:   nginx proxy_set_header
date:       2019-04-22
author:     WLX
header-img:  
catalog: 	 true
tags:
    - nginx
---

proxy_set_header 

- proxy_set_header Host $host:$server_port;


设置Host请求头， 如果不设置这个， Host请求头会变成upstream的名字

```editorconfig
upstream cluster {
  zone cluster 254k;
  server 10.0.3.17:9084 weight=1;
  keepalive 100;
}

server {
        listen   80  backlog=65535;
        server_name 10.0.3.16;

    keepalive_timeout 100;
        keepalive_requests 10240;

        location / {
        #proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout 60;
        proxy_read_timeout 60;

```
proxy_set_header Host $host:$server_port;注释掉后进行抓包

```editorconfig
^@^C^QÉ<85>#|<86>^Vl^Gn<95>*É<80>^X^@:^ZÓ^@^@^A^A^H
ØòvB:HG^QGET /TrioRobot/index.php HTTP/1.1^M
X-Real-IP: 10.0.3.16^M
X-Forwarded-For: 10.0.3.16^M
Host: cluster^M
User-Agent: curl/7.29.0^M
Accept: */*^M
^M

```
可以看出Host变成了cluster，恰恰upstream的名字就是cluster。

