---
layout: post
title: kali 安装教程
subtitle: kali 安装教程
date: 2024-11-06 21:31
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - Kali
  - Linux
  - 操作系统
---
# Windows10安装kali

可参考：[安装 WSL | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/install)

## Windows10安装WSL

### 前言

适用于 Linux 的 Windows 子系统 (WSL) 可让开发人员直接在 Windows 上按原样运行 GNU/Linux 环境（包括大多数命令行工具、实用工具和应用程序），且不会产生传统虚拟机或双启动设置开销。
### 系统需求

* 必须运行 Windows 10 版本 2004 及更高版本（内部版本 19041 及更高版本）或 Windows 11。

（若要检查 Windows 版本及内部版本号，选择 Windows 徽标键 + R，然后键入“winver”，选择“确定” 。 ）
### 安装WSL

以管理员权限打开PowerShell，输入 wsl --install 命令，然后重启计算机。

```powershell
wsl --install
```

查看可用的发行版列表

```shell
wsl --list --online
```
### 问题

1. 安装WSL时或查看分发版时报错：**错误: 0x80072ee7**，尝试挂节点。

## WSL安装kali

**安装kali**

```powershell
wsl --install -d kali-linux
```

==默认安装在C盘，之后会移动到其他盘==

**创建用户**

初次启动Kali会要求创建用户，按照习惯创建用户：kali，输入两次密码。

更改root密码：

```shell
sudo su #切换root用户
passwd root #更改root用户密码
```

**移动到d盘**

参考：[Windows10子系统WSL修改默认安装目录到其他盘_zhang-ge的博客-CSDN博客](https://blog.csdn.net/weixin_40837318/article/details/108233688)

查看WSL分发版本

```powershell
wsl -l --all -v
```

<img src="https://kyonk.v6.army:1443/uw8PFJ.png" alt="image-20221214170315792.png" title="image-20221214170315792.png" />

如果VERSION显示为1，可用以下命令将版本升级到2：

```powershell
wsl --set-default-version 2
#或者
wsl --set-version kali-linux 2
```

导出kali为tar文件

```powershell
wsl --export kali-linux d:\kali-linux.tar
```

注销当前分发版

```powershell
wsl --unregister kali-linux
```

重新导入并安装kali

```powershell
wsl --import kali-linux d:\kali-linux d:\kali-linux.tar --version 2
```

==之后可删除kali-linux.tar文件==

默认进入kali时为root用户，可在PowerShell用以下命令设置进入kali时的用户

```powershell
# kali config --default-user <username>
kali config --default-user kali
```
### 开启kali

利用PowerShell运行Kali，命令：`kali`
### Kali安装软件

参考：[Minimum Install Setup Information | Kali Linux Documentation](https://www.kali.org/docs/troubleshooting/common-minimum-setup/) 
#### 前言

最小安装是指没有安装kali-linux-headless或至少安装了kali-tools中的一个。这可能有很多原因，但是如果这些原因发生变化，有人想在安装后从他们的系统中获得更多的效用，他们就需要知道如何获得某些信息。

* [Metapackages](https://www.kali.org/docs/general-use/metapackages/) ：元包有助于一次快速轻松地安装许多工具。这可以很容易地从最小的安装过渡到功能齐全的桌面环境。

* [Kali Network Repositories](https://www.kali.org/docs/general-use/kali-linux-sources-list-repositories/) 和 [Kali Branches](https://www.kali.org/docs/general-use/kali-branches/) ：如果用户想要一个更静态的安装，了解Kali网络存储库和Kali分支是很有用的。

#### 利用Metapackages安装

要安装一个元包，我们首先需要更新并安装所需的包。在kali中输入命令：

```shell
sudo apt update
sudo apt install -y kali-linux-default
```

==安装完成需要几分钟，以及选择设置，选择默认设置即可==

**configuring**

<img src="https://kyonk.v6.army:1443/36COke.png" alt="71c950ec-49e3-4876-856f.png" title="71c950ec-49e3-4876-856f.png" />

选择NO

#### 安装GUI

参考：[Kali Linux gets a GUI desktop in Windows Subsystem for Linux (bleepingcomputer.com)](https://www.bleepingcomputer.com/news/security/kali-linux-gets-a-gui-desktop-in-windows-subsystem-for-linux/)

安装kali-win-kex

```shell
sudo apt install -y kali-win-kex
```

<img src="https://kyonk.v6.army:1443/GGsTru.png" alt="image-20221214182736518.png" title="image-20221214182736518.png" />

==选择gdm3==

安装完成后输入`kex`，之后要求设置密码（密码长度至少为6）。设置完成即开启gui。

kex命令：

```shell
kex <mode> <command> <parameters>
<mode>:
--esm	# 使用Windows本机RDP在专用窗口中启动KeX桌面 即远程桌面连接
--sl	# 将KeX整合到Windows桌面中 
--win	# 在专用窗口中启动KeX桌面
<command>
--status
--start
--stop
--passwd	#设置kex server密码
```

**ESM方式启动**

远程桌面连接方式

**SL方式启动**

``` shell
kex --sl --start
kex --sl
```

==报错：电源管理插件出错(plugin "power manger plugin" unexpectedly...)，点击Remove== 

**WIN方式启动**

此方式为默认方式，即输入`kex` 和 `kex --win` 一样，之后输入密码

<img src="https://kyonk.v6.army:1443/NCzlIC.png" alt="image-20221214185520103.png" title="image-20221214185520103.png" />

==可通过命令：key --win --passwd 修改密码==

默认为全屏显示，按下`F8` 可取消全屏

<img src="https://kyonk.v6.army:1443/6KnJrM.png" alt="image-20221214190323736.png" title="image-20221214190323736.png" />

注意：在GUI界面点击叉号关闭界面，并未完全关闭服务，可使用命令查询和关闭服务

```shell
kex --[win|sl|esm] --[status|stop]
```
### 关闭kali-linux

```shell
# 关闭单个发行版
wsl -t kali-linux
# 全部关闭
wsl --shutdown
```

启动

```shell
# 点击应用或使用命令
wsl --distribution kali-linux
```
## 问题及解决方法

1. 关闭kali后，再次开启时报错，显示文件被占用

   未搞清原因，可以重启解决。

   开启WSL后可以使用VirtualBox和VMware，应该不是虚拟机之间冲突。

2. kex --sl 开启GUI报错

   <img src="https://kyonk.v6.army:1443/YmIf7C.png" alt="image-20221214224239951.png" title="image-20221214224239951.png" />

   删除下列文件

	```shell
	rm /tmp/.X11-unix
	```

   再次运行时还会报错，但能正常出现界面

   参考：[ubuntu - Xvfb -screen --> Cannot establish any listening sockets - Make sure an X server isn't already running - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/232749/xvfb-screen-cannot-establish-any-listening-sockets-make-sure-an-x-server)

3. `kex --sl`无法开启界面

	````shell
	#重新安装kex
	sudo apt-get purge kali-win-kex
	sudo apt-get update
	sudo apt-get install kali-win-kex
	kex --sl --wtstart -s
	````