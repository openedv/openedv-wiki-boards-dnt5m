---
title: 'CH343P 驱动安装'
sidebar_position: 6
---

# CH343P驱动安装

## 概述

[**CH343P**](https://www.wch.cn/products/CH343P.html) 是一个 USB 总线的转接芯片，实现 USB 转一个异步串口，可以供T5下载和调试使用。

CH343P 的驱动程序可在其官网的[**资料下载**](https://www.wch.cn/search?q=CH343P&t=downloads)页面进行下载，也可以通过[**这里**](https://www.wch.cn/downloads/CH343SER_EXE.html)下载最新的适用于 Windows 的一键式安装驱动程序。

## 安装

> 本章以 Windows 环境为例，介绍使用 CH343P 的一键式安装驱动程序在 Windows 下安装 CH343P 的驱动程序

双击打开下载好的 CH343P 一键式安装驱动程序。

![CH343P driver installer](./img/CH343P-driver-installer.png)

直接点击安装程序中的“安装”按钮，安装会自动安装 CH343P 的驱动程序，并在安装成功后，进行弹窗提示。

![CH343P driver install done](./img/CH343P-driver-install-done.png)

CH343P 驱动程序安装成功后，若正常连接了 CH343P 设备，则能够在 Windows 的“设备管理器”中看到对应的两个端口设备。

![check CH343P com ports](./img/check-CH343P-com-ports.png)

> 跳线帽DLT&RXD和DLR&TXD连接，则 CH343P 连接至 T5 Board 的下载口
>
> 跳线帽LGT&RXD和LGR&TXD连接，则 CH343P 连接至 T5 Board 的调试口

至此，CH343P 驱动程序安装完毕。