---
layout: post
title: Vuluhub靶场IP问题
subtitle: Vuluhub靶场解决IP找不到问题
date: 2025-07-0115:57
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - Vulnhub
published: true
---
VMware导入ovf文件时提示导入失败，点击重试

![image.png](https://img.ghostliner.top/1zhmKZ.png)

在以下开机界面按下`e`键

![image.png](https://img.ghostliner.top/AoazIQ.png)

进入到黑色界面中找到 `ro quiet`关键字，将其修改为 `rw signie init=/bin/bash`

![image.png](https://img.ghostliner.top/MgivAp.png)

修改结果如下

![image.png](https://img.ghostliner.top/lF87Tq.png)

修改完后按下 `Ctrl+X`键，进入到root交互界面

`ip a`查看网卡名为 `ens33`

![image.png](https://img.ghostliner.top/po772s.png)

修改网卡配置文件，将 `enp0s3`改为 `ens33`

```shell
vi /etc/network/interfaces
:%s/enp0s3/ens33/g    # 全局替换
:wq    # 保存退出
```

![image.png](https://img.ghostliner.top/9OBFki.png)

最后重启虚拟机即可获取到ip
