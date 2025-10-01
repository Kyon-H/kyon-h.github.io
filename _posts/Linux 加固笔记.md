---
layout: post
title: Linux 加固
subtitle: Linux 加固
date: 2022-12-19 22:49
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 渗透测试
published: false
number headings: first-level 2, max 4, 1.1., auto
---
## 1. Linux 命令

### 1.1. 快捷键

|        |                |
| ------ | :------------: |
| ctrl+A |    光标移动到行首     |
| ctrl+E |    光标移动到行尾     |
| ctrl+K |  清除光标后至行尾的内容   |
| ctrl+U | 清除光标前至行首间的所有内容 |

<table>
	<tr><th align="center" colspan="2">通配符</th></tr>
	<tr><td>*</td><td>匹配任意字符</td></tr>
	<tr><td>?</td><td>匹配单个字符</td></tr>
	<tr><td>[]</td><td>匹配括号里面任意一个，比如[0-9][a-z] </td></tr>
	<tr><td>{}</td><td>匹配多个 ll {*.log,*.txt} </td></tr>
	<tr><td>^</td><td>取反 ll *[^.txt]</td></tr>
</table>

### 1.2. 命令

```shell
history #查看历史输入
cat ~/.bash_history # 历史命令文件
#查找文件和目录
find . -name "*.txt" -exec rm -rf {} \;
whereis #查找二进制程序、代码等相关文件路径
which #查找并显示给定命令的绝对路径
locate #updatedb程序每天会跑一次，建立文件索引
#查看文件内容
less
#enter 下一行
#space 向下滚动一屏
#b 往回翻一屏
#/ 向后查找内容
#? 向前查找内容
#n 下一个
#N 上一个
tail -f a.log # 实时查看文件尾部内容
wc * #统计文件行数-l，单词数，字母数
ll | wc -l #统计文件个数
chown -R redis:redis /usr/local/soft/redis #改变文件或目录的属主和属组
```

```shell
#编辑vi/vim
vi
###########################################
-命令模式
dd删除一行
Ctrl+F 前进一屏（Forward）
Ctrl+B 后退一屏（Backspace）
Shift+G 跳到文档结尾
gg 跳到文档开头
/ 向后查找内容
? 向前查找内容
n下一个
N上一个
###########################################
-底行模式
:n 回到第n行
:set nu 显示行号
```

## 2. Linux 应急响应

### 2.1. Linux 攻击方式

#### 2.1.1. 暴力破解

ssh 22 端口 ；工具：hydra medusa python

#### 2.1.2. 漏洞利用

Linux 内核；
java python TP log4j
容器：tomcat jboss weblogic Spring
CMS：通达 OA 悟空 CMS dedeCMS wordpress

#### 2.1.3. 提权

sudo 提权
修改 /etc/sudoers.d 文件
`sudo awk 'begin{system("/bin/bash")}'`
应用 UDF IIS FTP
SUID 提权

#### 2.1.4. 权限维持

上传木马

隐藏用户
创建 uid，gid 为 0 的用户

端口复用 进程注入

定时任务 crontab

自启动
checkconfig

SSH Key 免密登录

#### 2.1.5. 痕迹清理

登录信息 history 日志
/var/log 目录
btmp 文件
`# lastb` 查看登陆失败信息
secure 文件

CPU`ps -aux --sort=-pcpu | head -10`

内存`free -m`

网络`netstat -antpl`

## 3. Linux 安全加固

### 3.1. 身份鉴别

#### 3.1.1. 删除多余用户

查看`/etc/passwd`文件

命令`userdel`

#### 3.1.2. 口令安全策略

**空口令检查** `awk -F: '($2 == ""){print $1}' /etc/shadow`

**口令有效期** `/etc/login.defs`

设置

```
PASS_MAX_DAYS
PASS_MIN_DAYS
PASS_MIN_LEN
```

**口令复杂度**

```
/etc/pam.d/system-auth
/etc/pam.d/password-auth
```

密码生成：`pwmake 100`

**登录失败策略** `/etc/pam.d/sshd`

```
deny #错误次数
unlock_time #解锁时间
```

### 3.2. 访问控制

#### 3.2.1. 禁止 root 用户远程登录

/etc/ssh/sshd_config

```shell
PermitRootLogin no
systemctl restart sshd
```

#### 3.2.2. 禁止 su 非法提权

只允许 root 和 wheel 组用户 su 到 root

`/etc/pam.d/su`

```shell
auth sufficient /lib/security/pam_rootok.so
auth required /lib/security/pam_wheel.so
group=wheel
```

### 3.3. 安全审计

#### 3.3.1. 审计服务

```shell
systemctl start auditd
systemctl start rsyslog
systemctl enable auditd
systemctl enable rsyslog
```

### 3.4. 资源控制

#### 3.4.1. IP 是否允许访问

黑名单 `/etc/hosts.deny`

白名单 `/etc/hosts.allow`

#### 3.4.2. 会话超时锁定

会话超时锁定 `/etc/profile`
