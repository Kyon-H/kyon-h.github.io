---
layout: post
title: ppt大纲
subtitle: ppt大纲
date: 2025-07-19 16:33
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
published: false
---
# 个人服务器搭建

## 背景

## 选择

云服务器、NAS、企业级服务器、个人组装

### 区别

### 个人组装优势

## 如何搭建

### 网络配置

### 硬件配置

主机本身：cpu、内存

周边：远程开关机、ups

### 系统安装与分区

ubuntu系统推荐，其他系统种类

磁盘格式化类型

分区大小
### 软件配置

#### 基础软件配置

##### ssh

pam.d配置登录通知

##### vpn

**tailscale**
原理: 
功能：subnets exitnodes magicdns
##### 服务器面板

1panel 与宝塔相比的优点
##### WAF

雷池waf

### 域名与证书配置

#### 购买域名

cloudflare托管

#### ddns

ddns-go, lucky, 自写脚本

#### acme.sh安装证书

优点：多域名申请；集申请、安装、通知与一体；配置方便