---
layout: post
title: Minecraft 服务器安装
subtitle: Minecraft 服务器安装
date: 2024-11-16 00:11
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - Linux
  - MC
  - Minecraft
  - fabric
published: true
---
## 1.安装fabric服务端

<mark>依赖项：Java、curl</mark> 

[在不使用GUI的情况下安装 Fabric Server](https://fabricmc.net/wiki/zh_cn:player:tutorials:install_server) 

**创建服务器目录**

```shell
mkdir fabric
cd fabric
```

**下载 Facric Installer**

[下载页面 https://fabricmc.net/use/](https://fabricmc.net/use/) 

```shell
# 下载
curl -o installer.jar https://maven.fabricmc.net/net/fabricmc/fabric-installer/1.0.1/fabric-installer-1.0.1.jar
# 运行 1.20.6改为想要下的版本
java -jar installer.jar server -mcversion 1.20.6 -downloadMinecraft
# 删除 Fabric Installer
rm installer.jar
# 重命名 Jar 文件
mv server.jar vanilla.jar
mv fabric-server-launch.jar server.jar
echo "serverJar=vanilla.jar" > fabric-server-launcher.properties
 
# 启动 Minecarft 服务器
java -jar -Xms1G -Xmx2G server.jar nogui
```

初次启动失败后按提示修改`eula.txt`文件，再次启动即可
```
eula=true
```

其他安装方法：[Installing a Fabric Server without a GUI](https://fabricmc.net/wiki/player:tutorials:install_server) 

## 2.安装screen

[Linux终端命令神器--Screen命令详解。助力Linux使用和管理-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1844735) 

```shell
apt install screen
```

命令
```shell
# 创建名为mc的终端
# -R 若重名则直接进入之前创建的screen
# -S 可能会创建同名的screen
screen -R mc
# 退出，后台运行
Ctrl+a
d
# 查看screen终端
# pid.name 创建时间 Detached/Attached
screen -ls
# 返回screen终端
screen -r [pid/name]
```

**创建启动脚本**

创建`screen.sh`文件

```shell
#!/bin/bash
cd fabric
screen -R mc java -jar -Xms1G -Xmx2G server.jar nogui
```

## 3.修改配置

[Minecraft 服务器server.properties属性文件介绍 (最详细 最全 汉化) - 哔哩哔哩](https://www.bilibili.com/opus/422753987430124575) 

[服务端配置文件格式 - 中文 Minecraft Wiki](https://zh.minecraft.wiki/w/%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F?variant=zh) 

服务器配置文件：`server.properties`
```
# 关闭正版验证(可选)
online-mode=false
# 开启白名单(建议)
white-list=true
```

## 4.导入存档

Windows下`...\.minecraft\saves\`文件夹下的存档文件夹打包传到服务器，解压到MC服务器文件夹，修改`server.properties`文件
```
level-name=存档文件夹名
```

## 5.MCSManager

使用`MCSManager`代替`screen`，方便启动游戏、管理文件、监视运行情况等

**自动安装**
```shell
sudo su -c "wget -qO- https://script.mcsmanager.com/setup_cn.sh | bash"
```

**启动**
```shell
# 先启动面板守护进程。
# 这是用于进程控制，终端管理的服务进程。
systemctl start mcsm-daemon.service
# 再启动面板 Web 服务。
# 这是用来实现支持网页访问和用户管理的服务。
systemctl start mcsm-web.service

# 重启面板命令
systemctl restart mcsm-daemon.service
systemctl restart mcsm-web.service

# 停止面板命令
systemctl stop mcsm-web.service
systemctl stop mcsm-daemon.service
```

前端面板端口：23333；
后端服务端口：24444

**设置Nginx反代**

后端反代端口12444，location中添加WebSocket设置，前端正常设置反代
```
location / {
	......
	# 支持反代 WebSocket 
	proxy_set_header Upgrade $http_upgrade; 
	proxy_set_header Connection "upgrade";
	......
}
```

**添加远程节点**

节点IP/域名、端口号填写与后端反代配置相同，如果填入的是局域网IP，会出现远程无法连接情况。
```
#节点格式
mc.v6.army
wss://mc.v6.army
IP地址
wss://IP地址
```

秘钥在服务端初次运行时产生，保存于`/opt/mcsmanager/daemon/data/Config/global.json`

**创建应用实例**

实例类型：Minecraft Java服务器
节点选择：选择远程服务器节点
部署方式：服务器现有目录

之后填写启动命令`java -jar -Xms1G -Xmx3G server.jar nogui`和`server.jar`所在目录

## 6.模组

[模组检索 - MC百科](https://www.mcmod.cn/modlist.html)可以下载和查看mod是否需要安装到服务端

服务器端mod文件放到mc目录下的mods文件夹
```shell
$ ls mods/

AdvancedBackups-fabric-1.20-3.6.4.jar          FallingTree-1.20.1-4.3.2.jar
appleskin-fabric-mc1.20.1-2.5.1.jar            jade-1.20-Fabric-11.12.0.jar
create-fabric-0.5.1-f-build.1335+mc1.20.1.jar  jei-1.20.1-fabric-15.20.0.105.jar
fabric-api-0.92.2+1.20.1.jar                   journeymap-1.20.1-5.10.3-fabric.jar
```

## 7.光影

[Iris Shaders - MC百科](https://www.mcmod.cn/class/3697.html)

[Iris & Oculus Flywheel Compat - MC百科](https://www.mcmod.cn/class/7283.html)

fabric版安装`iris`到`mods`文件夹，若安装了机械动力模组还需安装`iris-flywheel-compat`优化飞轮

```
iris-1.7.5+mc1.20.1.jar
iris-flywheel-compat-fabric1.20.1+1.1.4.jar
```

光影包压缩文件直接放到`shaderpacks`文件夹
