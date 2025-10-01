---
layout: post
title: grotesque 2
subtitle: grotesque 2
date: 2023-07-18 08:58
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
  - Vulnhub
published: true
number headings: first-level 3, max 4, 1.1., auto
---

### 1. 环境搭建

[Vuluhub 靶场 IP 问题 - Kyon-H Blog](https://blog.kyon.xin/2025/07/01/Vuluhub%E9%9D%B6%E5%9C%BAIP%E9%97%AE%E9%A2%98/)

### 2. 信息搜集

扫描 IP 发现主机 `192.168.163.141`

![image.png](https://img.ghostliner.top/52s9AI.png)

扫描端口

```shell
nmap -T4 -A -p- 192.168.163.141 # 无反应
# 简单扫描
nmap 192.168.163.141
```

![image.png](https://img.ghostliner.top/RYKakd.png)

发现有多个端口开放，其中有 80 22 端口

访问 80 只有一张图片，目录扫描未发现其他内容

发现 80 端口经常 close，目标机报错内存溢出，分配 8G 电脑会很卡

![image.png](https://img.ghostliner.top/BpH0yc.png)

尝试查看其他端口

```shell
nmap -T4 -A -p 30-80 192.168.163.141
```

![image.png](https://img.ghostliner.top/rfhKut.png)

### 3. 端口爆破

发现都是 http 端口，使用 bp 遍历，查看内容长度

![image.png](https://img.ghostliner.top/C4oYAj.png)

![image.png](https://img.ghostliner.top/4aOJKZ.png)

发现 258 端口，访问

![image.png](https://img.ghostliner.top/bjftUP.png)

获得 ssh 用户名，保存到`user.txt`文件

```
satan
raphael
angel
distress
greed
lust
```

尝试爆破 ssh 失败

发现 emoji 图片有一串数字
![image.png](https://img.ghostliner.top/2T2XEL.png)

010 打开，发现数字

![image.png](https://img.ghostliner.top/cVE15p.png)

```
b6e705ea1249e2bb7b0fd7dac9fcd1b3
```

MD5 解密获得 `solomon1`

![image.png](https://img.ghostliner.top/Lgv5kI.png)

### 4. SSH 爆破

```shell
hydra -L user.txt -p solomon1 ssh://192.168.163.141
```

![image.png](https://img.ghostliner.top/jroLmj.png)

得到用户名：`angel`，密码：`solomon1`

ssh 登录成功，用户家目录下发现第一个 flag

![image.png](https://img.ghostliner.top/paFVmJ.png)

查看当前目录文件

![image.png](https://img.ghostliner.top/okxkpq.png)

查看 quiet 目录文件，发现大量文件，内容为：quiet

![image.png](https://img.ghostliner.top/Uudel6.png)

![image.png](https://img.ghostliner.top/UXTl60.png)

按文件大小排序，大小都一样

![image.png](https://img.ghostliner.top/tf17QX.png)

### 5. 提权

尝试 sudo 和 find 提权，未发现特别内容

```shell
find / -user root -perm -4000 -type f 2>/dev/null
```

![image.png](https://img.ghostliner.top/KBfxht.png)

出现内存不足问题，重启虚拟机

![image.png](https://img.ghostliner.top/rBzC4N.png)

查看根目录，发现 `rootcreds.txt` 文件，查看内容，发现 root 密码：`sweetchild`

![image.png](https://img.ghostliner.top/mDrLyD.png)

root 登录成功，获取 flag 文件

![image.png](https://img.ghostliner.top/1Ga6sr.png)
