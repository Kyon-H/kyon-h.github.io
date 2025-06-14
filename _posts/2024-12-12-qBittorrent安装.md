---
layout: post
title: qBittorrent 安装
subtitle: qBittorrent 安装
date: 2024-12-12 18:54
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - qBittorrrent
  - PBH
  - Ubuntu
  - 软件安装
published: true
---
## 1. 下载安装

<mark>配置文件：~/.config/qBittorrent/</mark>

[qBittorrent Enhanced Edition : poplite (launchpad.net)](https://launchpad.net/~poplite/+archive/ubuntu/qbittorrent-enhanced)

[poplite/qBEE-Ubuntu-打包 (github.com)](https://github.com/poplite/qBEE-Ubuntu-Packaging)

```shell
sudo apt-get update && sudo apt-get install software-properties-common -y
# qBittorrent
sudo add-apt-repository ppa:poplite/qbittorrent-enhanced
sudo apt-get update
sudo apt-get install qbittorrent-enhanced qbittorrent-enhanced-nox
# 启动
sudo service qbittorrent-enhanced-nox start
```

设置启动用户

```shell
cd /lib/systemd/system/
vim qbittorrent-enhanced-nox.service
```

## 2. 配置

#### BitTorrent

trackers list: `https://trackerslist.com/all.txt`

#### Web UI

设置`0.0.0.0`访问

第三方 UI：
[VueTorrent/VueTorrent: The sleekest looking WEBUI for qBittorrent made with Vuejs! (github.com)](https://github.com/VueTorrent/VueTorrent)

下载 vuetorrent

```shell
#解压
unzip vuetorrent.zip
#移动vuetorrent文件夹到存放目录
mv vuetorrent /home/user/Templates
#设置文件夹权限
chmod 2775 public/
setfacl -m u:用户名:rwx public/
```

设置文件夹权限如下所示：

```shell
$ ls -lh
total 8.0K
drwxrwsr-x+ 3 larsluph users 4.0K Jan 20 16:00 public
-rw-rw-r--  1 larsluph users    5 Jan  9 13:45 version.txt
```

<mark>若设置错误导致页面无法显示：</mark>

```shell
#修改qBittorrent.conf文件
cd /home/user/.config/qBittorrent
vim qBittorrent.conf

#WebUI\AlternativeUIEnabled 改为false
WebUI\AlternativeUIEnabled=false
#WebUI\RootFolder 删去后面
WebUI\RootFolder=

#最后刷新或重启
```

qbittorrent 设置备用 webui 地址：`/home/user/Templates/vuetorrent/`

#### 高级

1.
<img src="https://img.ghostliner.top/GFdTet.png" alt="image-20240820230738951.png" title="image-20240820230738951.png" style="zoom:50%" />

2.
<img src="https://img.ghostliner.top/iNYa7t.png" alt="image-20240820230937953.png" title="image-20240820230937953.png" style="zoom:50%" />

3.
<img src="https://img.ghostliner.top/tvYzB5.png" alt="image-20240820231006390.png" title="image-20240820231006390.png" style="zoom:50%" />

4.
<img src="https://img.ghostliner.top/0gEBAj.png" alt="image-20240820231030176.png" title="image-20240820231030176.png" style="zoom:50%" />

5.
<img src="https://img.ghostliner.top/jgfnou.png" alt="image-20240820231044716.png" title="image-20240820231044716.png" style="zoom:50%" />

## 3. PeerBanHelper

防止 PCDN

[Linux 手动部署 · PBH-BTN/PeerBanHelper Wiki (github.com)](https://github.com/PBH-BTN/PeerBanHelper/wiki/Linux-手动部署)

```shell
# 使用 apt 工具安装 OpenJDK 21 或更高版本：
sudo apt-get update
sudo apt-get install openjdk-21-jdk-headless -y
# 验证安装
java -version
```

下载 jar 包，使用命令来启动 PBH：

```shell
java -jar -Xmx256M -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+ShrinkHeapInSteps -jar PeerBanHelper.jar
```

_通常情况下，PBH 会自动探测桌面环境，并在支持的情况下启用 GUI，但在部分设备上，GUI 可能会初始化失败。遇到此类情况，请手动禁用 GUI：_

```shell
java -jar -Xmx256M -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+ShrinkHeapInSteps -jar PeerBanHelper.jar nogui
```

PBH 默认端口：9898

设置开机自启

`cd /etc/systemd/system && vim peerbanhelper.service`

```
[Unit]
Description=Start PeerBanHelper jar file
After=multi-user.target

[Service]
ExecStart=/usr/bin/java -Xmx386M -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+ShrinkHeapInSteps -Dfile.encoding=UTF-8 -Dstdout.encoding=UTF-8 -Dstderr.encoding=UTF-8 -Dconsole.encoding=UTF-8 -jar PeerBanHelper.jar nogui
Type=simple
# 设置PeerBanHelper.jar存放和工作目录
WorkingDirectory=/home/user/bin/peerbanhelper

[Install]
WantedBy=multi-user.target
```

## 4.自动化

#### 4.1 自动下载

**RSS 订阅：**

```
# 音乐
http://www.kisssub.org/rss-3.xml
# 爱恋动漫
http://www.kisssub.org/rss-1.xml
# Nyaa Pantsu
https://ouo.si/feed?c=_&q=baha
```

**下载规则：**

| 名称   | FLAC                                                                          |
| ------ | ----------------------------------------------------------------------------- |
| 正则   | True                                                                          |
| 包含   | `.*2025.*TVアニメ.*flac`                                                      |
| 不包含 | `Hi-Res\|mkv\|mp4\|ost\|オリジナル\|サウンドトラック\|original.*sound.*track` |

| 名称   | MP3                                                                           |
| ------ | ----------------------------------------------------------------------------- |
| 正则   | True                                                                          |
| 包含   | `.*\[\d{6}\]TVアニメ.*(mp3\|320)`                                             |
| 不包含 | `Hi-Res\|mkv\|mp4\|ost\|オリジナル\|サウンドトラック\|original.*sound.*track` |

#### 4.2 邮件通知

| 选项   | 值           |
| ------ | ------------ |
| 发件人 | xxx@126.com  |
| 收件人 | zzz@qq.com   |
| 服务器 | smtp.126.com |
| 用户名 | xxx@126.com  |
| 密 码  | TOKEN        |

**获取 TOKEN**

[网易邮箱（126/163）：授权码获取攻略\_网易邮箱授权码-CSDN 博客](https://blog.csdn.net/kissradish/article/details/108447972)

#### 4.3 调用脚本

qBittorrent 设置种子下载完成时运行外部程序：`/path/hlink.py "%F" "%L"`

hlink.py:为文件建立硬链接，根据分类放入音乐库目录、番剧库目录、电影库目录......，主要函数如下：

##### 日志记录

```python
# logconfig.py
import os
import logging
PATH = os.path.dirname(os.path.abspath(__file__))
VIDEOS_LOG = os.path.join(PATH, 'log.txt')
def setup_logger(name: str, log_file=VIDEOS_LOG, level='INFO'):
    """
    @param name: Name of the logger
    @param log_file: Path to the log file
    @param level: Level of logging (DEBUG, INFO, WARNING, ERROR, CRITICAL)
    """
    # formatter
    console_format = logging.Formatter(
        '%(name)s - %(levelname)s - %(message)s')
    file_format = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s')
    # handler
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(console_format)
    file_handler = logging.FileHandler(log_file, encoding='utf-8')
    file_handler.setFormatter(file_format)
    # logger
    logger = logging.getLogger(name)
    if level == 'DEBUG':
        logger.setLevel(logging.DEBUG)
    elif level == 'INFO':
        logger.setLevel(logging.INFO)
    elif level == 'WARNING':
        logger.setLevel(logging.WARNING)
    elif level == 'ERROR':
        logger.setLevel(logging.ERROR)
    elif level == 'CRITICAL':
        logger.setLevel(logging.CRITICAL)
    logger.addHandler(console_handler)
    logger.addHandler(file_handler)
    return logger
```

##### 视频处理

```python
import re
# 正则表达式
# 匹配 [GM-Team][国漫][凡人修仙传 星海飞驰][Fan Ren Xiu Xian Zhuan][2023][37][AVC][GB][1080P]
REGEX_GMTEAM = re.compile(r'^(\[GM-Team\])\[.+?\]\[(.+?)\].*\[(\d\d)\](.*)')
# 匹配一般格式
# [Nekomoe kissaten][Monogatari Series - Off & Monster Season][06][1080p][JPSC]
# [Nekomoe kissaten] Monogatari Series - Off & Monster Season [06][1080p][JPSC]
# (\d\d.*?)匹配 01、01v2、01.5、01(OAD) 等情况
REGEX_COMMON = re.compile(r'^(\[.+?\]\s*)\[?(.+?)\]?\s*\[(\d\d.*?)\](.*)$')
```

```python
def file_size(filePath) -> int:
    # 返回文件大小(MB)
    return os.path.getsize(filePath) // (1024 * 1024)
   
def GMTeam(name) -> str:
    # [GM-Team字幕组],对目录或文件名进行处理
    rslt = REGEX_GMTEAM.match(name)
    if rslt:
	    name = f'{rslt.group(1)}{rslt.group(2)} - {rslt.group(3)} {rslt.group(4)}'
    return name

def common(name) -> str:
	# （常规字幕组）对目录或文件名进行处理
    # @param name: 单级目录名或单级文件名
    rslt = REGEX_COMMON.match(name)
    if rslt:
        name = f'{rslt.group(1)}{rslt.group(2)} - {rslt.group(3)} {rslt.group(4)}'
    return name
```

```python
import zhconv
def rename_path(filepath):
    path, filename = os.path.split(filepath)
    if str(filename).startswith('[GM-Team]'):
        filename = GMTeam(filename)
    else:
        filename = common(filename)
    filepath = os.path.join(path, filename)
    # 繁转简
    filepath = zhconv.convert(filepath, 'zh-cn')
    return filepath
```

```python
def link_all(src, dst):
    '''
    递归遍历源目录下的文件，硬链接或复制到目标目录
    @param src: 源目录
    @param dst: 目标目录
    '''
    for item in os.scandir(src):
        iname = item.name
        nsrc = os.path.join(src, iname)
        if item.is_dir():
            ndst = os.path.join(dst, common(iname))
            link_all(nsrc, ndst)
        elif item.is_file():
            ndst = os.path.join(dst, iname)
            link_file(nsrc, ndst)
    return
```

```python
import shutil
from send2trash import send2trash
def link_file(src, dst):
    '''
    针对单个文件进行硬链接
    @param src: 源文件路径
    @param dst: 目标文件路径
    '''
    dst = rename_path(dst)
    # 路径不存在时逐层创建目录
    if not os.path.exists(os.path.dirname(dst)):
        os.makedirs(os.path.dirname(dst))
    # 文件已存在
    if os.path.exists(dst):
        if file_size(src) != file_size(dst):
            send2trash(dst)
        else:
            return
    if file_size(src) > 4:
        # 视频文件硬链接
        logger.info('link file to: ' + dst)
        os.link(src, dst)
    else:
        # 复制
        logger.info('copy file to: ' + dst)
        shutil.copy2(src, dst)
    return
```

##### 音乐处理

对 rar 文件进行解压

```python
import rarfile
def unrar_file(src: str, dst: str):
    # 创建RARFile对象
    rar = rarfile.RarFile(src)
    # 解压RAR文件
    rar.extractall(dst)
    # 关闭RARFile对象
    rar.close()
    return
```

音乐类型直接复制到音乐库，调用音乐库格式化脚本

```python
def music_type(src: str):
    dst = src.replace(os.path.join(SRCPATH, 'Music'), MUSICPATH)
    if os.path.isdir(src):
        logger.info(f'copy dir to: {dst}')
        shutil.copytree(src, dst)
    elif src.endswith('.rar'):
        logger.info(f'unrar file to: {MUSICPATH}')
        unrar_file(src, MUSICPATH)
    else:
        logger.error(f'unknown file: {src}')
        return
    py_path = os.path.join(os.path.dirname(
        PATH), os.path.join('music', 'renameDir.py'))
    logger.info(f"call: {py_path}")
    os.system(f'python {py_path}')
    return
```

## 5. 手动更新

1. 下载最新版：[Releases · c0re100/qBittorrent-Enhanced-Edition](https://github.com/c0re100/qBittorrent-Enhanced-Edition/releases)，选择`qbittorrent-enhanced-nox_x86_64-linux-musl_static.zip`
2. 解压出 qbittorrent-nox 文件
3. 替换原 qbittorrent-nox 文件（可用 which 命令查找）
4. 重启 qbittorrent
