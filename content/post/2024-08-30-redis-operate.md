---
title:       "redis实操"
date:        2024-08-30
author:     "LBT"
tags:        ["redis", "operate"]
categories:  ["Tech", "TODO"]
---

## 1、安装并运行redis
查看[官网](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-on-linux/)的安装方法，centos上安装只需通过yum和systemctl管理
```shell
sudo yum install redis
sudo systemctl enable redis
sudo systemctl start redis
```
查看使用redis，ping有回应pong即成功
```shell
redis-cli
# 127.0.0.1:6379> ping
# PONG
```
查看redis配置信息
```
> systemctl status redis
● redis.service - Redis persistent key-value database
   Loaded: loaded (/usr/lib/systemd/system/redis.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/redis.service.d
           └─limit.conf
   Active: active (running) since Thu 2024-08-29 14:53:33 CST; 1 day 2h ago
 Main PID: 3431 (redis-server)
    Tasks: 3
   Memory: 1.0M
   CGroup: /system.slice/redis.service
           └─3431 /usr/bin/redis-server 127.0.0.1:6379

Aug 29 14:53:33 iZwz9a0qyaivfofzli7x6rZ systemd[1]: Starting Redis persistent key-value database...
Aug 29 14:53:33 iZwz9a0qyaivfofzli7x6rZ systemd[1]: Started Redis persistent key-value database.
```
可以看出启动信息在文件/usr/lib/systemd/system/redis.service中，继续查看
```
> cat /usr/lib/systemd/system/redis.service 
[Unit]
Description=Redis persistent key-value database
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/bin/redis-server /etc/redis.conf --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
```
得知配置文件为/etc/redis.conf，继续查看，配置内容如下
```

```
## 2、go sdk连接redis
todo