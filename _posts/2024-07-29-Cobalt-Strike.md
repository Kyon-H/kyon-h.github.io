---
layout: post
title: Cobalt Strike 使用方法
subtitle: Cobalt Strike 使用方法
date: 2024-07-29 11:05
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - CobaltStrike
published: true
---
## 1. Cobalt Strike 简介

**Cobalt Strike**:：C/S 架构的商业渗透软件，适合多人进行团队协作，可模拟 APT 做模拟对抗，进行内网渗透。

其集成了端口转发、服务扫描，自动化溢出，多模式端口监听，win exe 木马生成，win dll 木马生成，java 木马生成，office 宏病毒生成，木马捆绑；钓鱼攻击包括：站点克隆，目标信息获取，java 执行，浏览器自动攻击等等。

## 2. 配置与基础用法

- [ ] cs 配置与用法 🛫 2025-08-02 

```shell
apt install openjdk-21-jdk -y
chmod +x TeamServerImage
chmod +x teamserver
./teamserver 0.0.0.0 password
ufw allow from any to any port 50050
```

## 3. 插件配置

- [ ] cs 插件配置 📅 2025-08-02

## 4. 流量分析

### 4.1. HTTP

#### 4.1.1. 环境准备

- 下载：[Slzdude/cs-scripts](https://github.com/Slzdude/cs-scripts)
- 下载：[WBGlIl/CS_Decrypt](https://github.com/WBGlIl/CS_Decrypt)

```text
get_RSA.java 用来获取公私密钥
Beacon_metadata_RSA_Decrypt.py 解密元数据
CS_Task_AES_Decrypt.py 解密CS发送任务数据
Beacon_Task_return_AES_Decrypt.py 解密Beacon任务执行结果
```

**python 虚拟环境**

```shell
python3 -m venv venv
source venv/bin/activate
```

**cs-scripts 安装 module**

```shell
cd cs-scripts
pip install -r requirements.txt
```

**CS_Decrypt 安装 module**

```shell
pip3 install pycryptodome hexdump
```

修改 `Beacon_metadata_RSA_Decrypt.py` 脚本内容

```python
# 第5行改为
from Crypto.PublicKey import RSA  
from Crypto.Cipher import PKCS1_v1_5
# 第18、19行改为
private_key = RSA.import_key(PRIVATE_KEY.format(base64_key).encode())
cipher = PKCS1_v1_5.new(private_key)
ciphertext = cipher.decrypt(base64.b64decode(encode_data), 0)
```

![image.png](https://img.ghostliner.top/MNTJyW.png)

**抓包 HTTP 流量**

![image.png](https://img.ghostliner.top/BA3uKX.png)

#### 4.1.2. 解密 `.cobaltstrike.beacon_keys` 文件

关于 `.cobaltstrike.beacon_keys` 文件：
- 这是 Cobalt Strike 启动时自动生成的密钥文件。
- 里面存放了 Beacon 通信使用的 RSA 公私钥对，也就是 C2 加密的根密钥。
- 它是一个 Java 序列化对象，我们需要反序列化再提取出对应密钥。

复制 `.cobaltstrike.beacon_keys` 文件到 cs-scripts 目录下，进行解码

```shell
cp -f ../Server/.cobaltstrike.beacon_keys ./
python3 parse_beacon_keys.py
```

base64 格式的私钥

```
-----BEGIN PRIVATE KEY-----
MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBAMEfMwkoBmfb0DjzkqANhWLGDEg6
y63yaORX4pjzrnniyqh4pHkGNp8ktr79fQ4QkERB8HtMikxQZtnhi0+Synbv3dd9HUm0YExczY+Q
u0TBApA5jXHObJMailW4LhixwCgUX3pLaNu1he1OAyn1Z45xyaLZ2/8WCyst5jp2Uos9AgMBAAEC
gYAbJaQacuZnnhYok1C2r//ikR1z39P22T8WbiY7wvFxT8iWIxNXseBmwZXwxhhYrEpjVfOUmX9N
V/YFRbe8EVHlZKFNetw0omA9vHLwQMej9DQWrL86ShC6N0xYupf4LQjqelPhm/Gh5WrsxGFVctHQ
kYFW1a8tFA3ckcHHa3NezwJBANRKNLKBBev8dqRicPimCV7KbgxQNNiP9KARVpCblAt8oXj8qbH7
7RffAWBx6cEWzUp7YaNVue/hCHLr+tEgsz8CQQDo4qYiqp4bQhxPtQfmGgTaeNsc1Tmzdgtk4R5r
CXHZvTXnJBIPQVYuedRzyD+ogUo5d8nx34TY8Ub9qOvQvq6DAkASicReTiwRNoO5ySrqW713vJ+t
jZd/zdpj2/++MwfTlPeY1B+Rfllu+zdoj6oFBZO5zFpzY/oPu4v8VSUa/AsLAkEAr+4QvZ2Q5Vyz
EI/c0LqVMgoc1RJLjcQ+ZU4fcZLn/CqRHvVD41xjY6bLlVAQrxZE4VcaKuvFazIShCvpQX/bSwJB
AIrVvakfUhMbq0CNVt82Uwu8AX054P27bFgIvAU36dyCA+MF40hmeuN69CG53bQlh6p5gyIFvlVi
qU+lU9dPWsI=
-----END PRIVATE KEY-----
```

base64 格式公钥

```
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDBHzMJKAZn29A485KgDYVixgxIOsut8mjkV+KY
86554sqoeKR5BjafJLa+/X0OEJBEQfB7TIpMUGbZ4YtPksp2793XfR1JtGBMXM2PkLtEwQKQOY1x
zmyTGopVuC4YscAoFF96S2jbtYXtTgMp9WeOccmi2dv/FgsrLeY6dlKLPQIDAQAB
-----END PUBLIC KEY-----
```

#### 4.1.3. 秘钥解密心跳包 cookie

![image.png](https://img.ghostliner.top/BA3uKX.png)

查看 No.45 数据包

![image.png](https://img.ghostliner.top/xjPhx8.png)

得到 cookie 值为 base64 加密数据

```text
Hck8JAeWvJalkzPtPfu1M6uvCSjvi43l2bUbTgLK6/GAzoaEJ6UW60eurydGUQ42ypLxqRrOEiWYsXSHc4S5XErhbvaj6D8KlFzWMFRt5/tJCQjQmMv3pb3ziHSXp9ej65DFAjyn/Osh3smIVR+1VCAte7EogvDlgWspn+AYE70=
```

使用 cs_decrypt 解码，`Beacon_metadata_RSA_Decrypt.py` 脚本中填入 cookie 和私钥

![image.png](https://img.ghostliner.top/XNQNXs.png)

解码得到明文数据

![image.png](https://img.ghostliner.top/DOC2Xq.png)

```
AES key:3cb2e8e4293e9318e954ed214e862f7a
HMAC key:df63f5faf3e7d0bf3cc805793484ebef
```

#### 4.1.4. 解密下发任务

![image.png](https://img.ghostliner.top/BA3uKX.png)

查看 No.1192 数据包

![image.png](https://img.ghostliner.top/bcI5MB.png)

得到 Data 数据，将其转为 Base64

[在线Hex和Base64互转工具-hex转base64,base64转hex](https://config.net.cn/tools/HexToBase64.html)

```
943795324f2e39e3e48231421e398818f7cb6b44e712fecb41024c029c015d18700e60ed9ed2823be5584f96d51adbb85dd3ae200be7a10287fb5a7e96909e85
# HEX=>base64
lDeVMk8uOePkgjFCHjmIGPfLa0TnEv7LQQJMApwBXRhwDmDtntKCO+VYT5bVGtu4XdOuIAvnoQKH+1p+lpCehQ==
```

将 `AES key`、`HMAC key` 和 Base64 data 填入 `CS_Task_AES_Decrypt.py` 脚本

![image.png](https://img.ghostliner.top/AFVgE9.png)

执行得到任务数据 `whoami`

![image.png](https://img.ghostliner.top/2RvCI8.png)

#### 4.1.5. 解密 post 任务执行结果数据

![image.png](https://img.ghostliner.top/BA3uKX.png)

查看 No.1199 数据包

![image.png](https://img.ghostliner.top/8zID6A.png)

得到 Data 数据，将其转为 Base64

[在线Hex和Base64互转工具-hex转base64,base64转hex](https://config.net.cn/tools/HexToBase64.html)

```
000000302ada4276c3e168e76989708e415f8dd967b4b78d2459293ac19aaa0df1c4b30337c059bf1caf58daabe32f51f72f1bbf
# HEX=>base64
AAAAMCraQnbD4WjnaYlwjkFfjdlntLeNJFkpOsGaqg3xxLMDN8BZvxyvWNqr4y9R9y8bvw==
```

将 `AES key`、`HMAC key` 和 Base64 data 填入 `Beacon_Task_return_AES_Decrypt.py` 脚本

![image.png](https://img.ghostliner.top/5MVyd4.png)

解密得到任务执行结果 `ld-pc\ld`

![image.png](https://img.ghostliner.top/Z0qRAK.png)
