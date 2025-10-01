---
layout: post
title: Docker安装与使用
subtitle: Docker安装与使用
date: 2024-08-25 11:42
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - Docker
number headings: first-level 2, max 4, 1.1., auto
published: false
---
## 1. 安装

### 1.1. Linux 安装 Docker

```shell
apt install docker.io docker-compose -y
```

### 1.2. 镜像加速

```json
{
	"registry-mirrors": [
		"https://docker.1panel.live",
		"https://docker.m.daocloud.io"
	]
}
```

## 2. 导入镜像

```shell
docker compose load -i xxx.tar
docker compose build
docker compose up -d
```