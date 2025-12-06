---
title: '项目编译下载'
sidebar_position: 2
---

# 项目编译下载

## 概述

在前面已经完成了 TuyaOpen 源码拉取以及必要工具下载，接下来，体验一下项目编译下载。可编译项目可在 `apps`、`examples`中进行选择。

这里以`sample_project`为例。首先，进入项目目录。

```
cd examples/get-started/sample_project
```

## 配置项目

使用命令 `tos.py config choice`对项目进行配置。

该命令会提供已经验证过的配置选项，您可以根据自己的硬件设备进行选择。

```
❯ tos.py config choice
[INFO]: Running tos.py ...
[INFO]: Fullclean success.
--------------------
1. LN882H.config
2. EWT103-W15.config
3. Ubuntu.config
4. ESP32-C3.config
5. ESP32-S3.config
6. ESP32.config
7. T3.config
8. T5AI.config
9. T2.config
10. BK7231X.config
--------------------
Input "q" to exit.
Choice config file:
```

我们的板子也是使用T5模组，所以这里可以选择`T5AI.config`。

我们的例程都会有生成一个config配置文件方便大家直接通过命令 `tos.py config choice`直接选择即可，而不用重新配置，节省配置时间。

当然，我们也可以通过 `tos.py config menu`命令去选择一条条配置，也是一样的效果。后续，我们也在TuyaOpen中创建我们板子对象，下一个章节介绍。

## 编译工程

编译项目，使用命令`tos.py build`，首次编译会比较慢。

```
❯ tos.py build
...
[NOTE]:
====================[ BUILD SUCCESS ]===================
 Target    : sample_project_QIO_1.0.0.bin
 Output    : xxx\TuyaOpen\examples\get-started\sample_project\dist\sample_project_1.0.0
...
========================================================
```

## Note

执行完命令`tos.py build`，在工程文件夹下新生成一个dist文件夹，里面就会编译生成的固件。

```
sample_project_QIO_1.0.0.bin        //bootload + 用户区固件
sample_project_UA_1.0.0.bin         //用户区固件
sample_project_UG_1.0.0.bin         //升级固件
```

sample_project_QIO_1.0.0.bin 烧录进板子就能运行起来了。

## 项目烧录下载

将板子的下载口与电脑相连，并下载了对应的串口驱动，使用命令`tos.py flash`烧录固件。

```
❯ tos.py flash
...
[INFO]: Flash write success.
```

## 查看日志

将板子的日志串口与电脑相连，并下载了对应的串口驱动，使用命令`tos.py monitor`查看日志。若有多个串口就需要选择日志口。

如需查看完整日志，可在命令后，手动复位设备。

```
❯ tos.py monitor
...
ap0:W(216):[01-01 00:00:00 ty N][sample_project.c:38] Application information:
ap0:W(216):[01-01 00:00:00 ty N][sample_project.c:39] Project name:        sample_project
ap0:W(217):[01-01 00:00:00 ty N][sample_project.c:40] App version:         1.0.0
ap0:W(217):[01-01 00:00:00 ty N][sample_project.c:41] Compile time:        Dec  2 2025
ap0:W(217):[01-01 00:00:00 ty N][sample_project.c:42] TuyaOpen version:
ap0:W(218):[01-01 00:00:00 ty N][sample_project.c:43] TuyaOpen commit-id:  95aa4496d6c9917f094dcd26f16f8853f7a77eab
ap0:W(218):[01-01 00:00:00 ty N][sample_project.c:44] Platform chip:       T5AI
ap0:W(219):[01-01 00:00:00 ty N][sample_project.c:45] Platform board:      TUYA_T5AI_CORE
ap0:W(219):[01-01 00:00:00 ty N][sample_project.c:46] Platform commit-id:  9552e2cfcd9c97bcf43a2ac9728f9f268f832235
ap0:W(220):[01-01 00:00:00 ty D][sample_project.c:48] hello world

ap0:W(220):[01-01 00:00:00 ty D][sample_project.c:54] cnt is 1
ap0:W(220):[01-01 00:00:00 ty D][sample_project.c:62] cnt is 10
```

如需退出日志查看，按键`ctrl + C`并回车。

```
^C[INFO]: Press "Entry" ...
[INFO]: Monitor exit.
```

这时候会发现工程下面会有一个monitor.log文件记录了日志内容。

## 清除编译缓存

清理编译缓存，使用命令`tos.py clean`或`tos.py clean -f`（深度清理）。

```
❯ tos.py clean -f
[INFO]: Running tos.py ...
[INFO]: Fullclean success.
```

清除缓存之后，就需要重新编译才能下载程序了。
