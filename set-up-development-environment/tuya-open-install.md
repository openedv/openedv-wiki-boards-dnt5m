---
title: 'TuyaOpen 环境搭建'
sidebar_position: 1
---

# TuyaOpen环境搭建

## 概述

TuyaOpen 是一个面向 AIoT 行业的开源、开放的开发框架，基于成熟的商业级 IoT 系统 TuyaOS 构建而成。它继承了跨平台、跨系统、组件化和安全合规等核心特性，已通过全球亿级设备和百万级用户的实践验证。

![tuyaopen](./img/tuyaopen-install.jpg)

## 安装

下载、安装以下工具，并添加到`环境变量`中，重启电脑，保证在终端中可以正确使用对应的命令：

- Python v3.14.2 或更高版本
- Git v2.0.0 或更高版本
- Make v3.0 或更高版本

**建议用与本章节一致的版本的工具**

[下载地址]：资料盘(A盘) -> 6，软件资料 -> TuyaOpen编译工具链软件.rar 

或者从网上下载最新版本进行安装

### Python v3.14.2安装

双击python-3.14.2-amd64.exe安装包。

![python install](.\img\python-install.png)

勾选"Add Python 3.14.2 to PATH"，然后点击Customize installation选项。

![python setup](.\img\python-setup.png)

点击“Next”下一步，进入配置安装路径。

![install location](.\img\customize-install-location.png)

最后点击“install”安装Python 3.14.2。

![python-end](.\img\python-end.png)

### Git v2.0.0安装

下载Git2.51.0安装，直接点击“Next”下一步就完成安装。

![git install](.\img\git-install.png)

### Make v3.0安装

双击make-3.81.exe安装包。

![make install](.\img\make-install.png)

**记得！！！安装完对应的软件之后，添加环境变量，添加环境变量，添加环境变量**

## 下载并激活 TuyaOpen

> [!CAUTION]
>
> 选择项目路径的时候，不使用中文，也不要包含空格等特殊字符，Windows环境不要选择C盘。

打开Git终端，输入`git clone https://gitee.com/tuya-open/TuyaOpen.git`克隆TuyaOpen源码库。

![git-clone-tuyaopen](.\img\git-clone-tuyaopen.png)

打开Windows PowerShell终端，然后进入TuyaOpen源码库，最后输入`.\export.ps1`命令或者`.\export.bat`执行安装操作。

![powershell-install-tuyaopen](.\img\powershell-install.png)

**记得！！！每次关闭Windows PowerShell终端，重新打开之后，还是得运行`.\export.ps1`命令执行环境准备。**

若提示无命令操作等信息，可先运行`Set-ExecutionPolicy RemoteSigned -Scope LocalMachine`开启**提升脚本运行权限**的命令，然后再一次运行`.\export.ps1`命令执行安装操作。

tuyaopen验证，使用命令 `tos.py version` 以及 `tos.py check`，会出现如下信息：

```
❯ tos.py version
[INFO]: Running tos.py ...
[INFO]: v1.3.0

❯ tos.py check
[INFO]: Running tos.py ...
[INFO]: [git] (2.43.0 >= 2.0.0) is ok.
[INFO]: [cmake] (4.0.2 >= 3.28.0) is ok.
[INFO]: [make] (4.3 >= 3.0.0) is ok.
[INFO]: [ninja] (1.11.1 >= 1.6.0) is ok.
[INFO]: Downloading submoudules ...
[INFO]: [do subprocess]: cd /home/huatuo/work/open/TuyaOpen && git submodule update --init
[INFO]: Download submoudules successfully.
```

若 check 命令失败：

```
# 工具校验不合格，请安装或升级对应工具

# submodules 下载失败，手动执行 git 命令
git submodule update --init
# 子模块更新完成后，再一次执行tos.py version和tos.py check命令
```

## 常见问题

> [!CAUTION]
>
> tos.py激活失败

如果激活失败，可能是因为没有安装 `python3-venv`，请安装后重新激活。

```
sudo apt-get install python3-venv
```

`tos.py`激活时会自动创建`./.venv`目录。如果激活失败，需要删除`./.venv`目录，并重新激活。

> [!IMPORTANT]
>
> 重新打开TuyaOpen

必须在终端输入`.\export.ps1`命令，进入tos模式，方能使用tos.py命令。
