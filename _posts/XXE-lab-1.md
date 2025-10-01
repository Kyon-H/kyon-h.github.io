---
layout: post
title: vulnhub靶场 XXE Lab 1
subtitle: vulnhub XXE Lab 1
date: 2023-07-03 14:49
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - Vulnhub
  - 靶场实战
published: true
---

## 1. 信息搜集

nmap 探测主机

![1751525486551-c4d82d51-abe7-4e07-851f-ec1979401898.png](https://img.ghostliner.top/uV0tzr.png)

扫描端口

![1751525607131-936e33a2-5587-4823-8839-9c3bd2051632.png](https://img.ghostliner.top/o6HFh8.png)

访问 80 端口

![1751525656689-57cfdb60-6a54-44ec-9bb8-6a388088f11f.png](https://img.ghostliner.top/72YuN7.png)

扫描目录

![1751525843692-3ebf4cd9-399b-4f6e-9d03-222bd9367d17.png](https://img.ghostliner.top/8mmoYY.png)

访问 robot.txt

![1751525872452-ddf8ce67-2e31-41b3-920b-3d6a8bc20a3b.png](https://img.ghostliner.top/yjb1e5.png)

访问 `http://192.168.163.131/xxe/admin.php`

![1751525936517-9598a38e-c985-4ec7-b4e1-20c6d88ff89b.png](https://img.ghostliner.top/FI8u5Y.png)

## 2. 漏洞探测

bp 抓包发现支持 `application/xml`

![1751525997201-16bdc328-a2da-4e1b-9365-523208c11121.png](https://img.ghostliner.top/69DMjG.png)

构造 xml 数据

![1751527865717-d37aea37-e1dc-4bb0-89cc-2b915683e8f5.png](https://img.ghostliner.top/P0H4C0.png)

无效，继续扫描 xxe 目录

![1751527872253-34646fef-b5ac-4948-a796-787eed650f09.png](https://img.ghostliner.top/yKbYgv.png)

访问 `http://192.168.163.131/xxe/`

![1751527904102-08bc314c-4729-4927-bc45-39ff5060cf1b.png](https://img.ghostliner.top/r9Yu1k.png)

bp 抓包构造 xml

![1751528068835-99ecadc3-c162-4650-8492-1ff7b2042de7.png](https://img.ghostliner.top/zEB5zr.png)

![1751528057485-e3d2814b-44a6-4e04-85e9-cd071121b711.png](https://img.ghostliner.top/4kazHV.png)

成功访问

### 2.1. 漏洞利用

![1751532283597-abb0753a-a862-4283-ac71-99f7a355d08d.png](https://img.ghostliner.top/nVMzDW.png)

解码获得

![1751532403522-3754f0a7-6c81-4f07-a85f-01f11873cca2.png](https://img.ghostliner.top/r22q8J.png)

获取 `admin.php`，获得用户名、密码

![1751532661233-7a22fde5-e5d7-46bd-ac88-19e9c3599510.png](https://img.ghostliner.top/TgoPwd.png)

MD5 解码

![1751532719700-189b5aae-d8e1-4334-b1af-4d82c2dc6547.png](https://img.ghostliner.top/9cNTAK.png)

登录成功获取到

![1751532809260-a1189edf-cf42-4b90-bd57-db6c6602510e.png](https://img.ghostliner.top/ewSgub.png)

`the flag in (JQZFMMCZPE4HKWTNPBUFU6JVO5QUQQJ5)`

解码

![1751533155205-64f492f3-8c9e-4239-a89a-4b7b24cb0201.png](https://img.ghostliner.top/vVZI0p.png)

![1751533184816-8590a6d5-add9-4c1e-85a1-ec71eccb8a5d.png](https://img.ghostliner.top/hjXFRS.png)

### 2.2. 获取 flag

![1751533237089-a0580453-d854-4f45-9a31-3b020765642b.png](https://img.ghostliner.top/T931RK.png)

![1751534416951-c12fc00f-7926-4517-9a5f-e31a0c792f10.png](https://img.ghostliner.top/YOMoa3.png)

放到 php 网站中

![1751534536537-bf317401-e97b-4a6f-a68d-111d833f7bd6.png](https://img.ghostliner.top/XisCYG.png)

flag：**SAFCSP{xxe_is_so_easy}**
