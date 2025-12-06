---
title: '新建Board'
sidebar_position: 3
---

# 创建新的板子支持包

## 概述

让开发者自主设计的新硬件开发板能让TuyaOpen支持，在命令行中可以支持选择配置。

## Board命名

### 必选字段

- 厂家名称：代表开发板的生产厂家。如涂鸦推出的开发板则会用 "TUYA" 开头
- 芯片名称：platform 名称，如 "T2"、"T5AI"

### 可选字段

- 体现开发板的主要特点或者功能：若 board 具有特定的功能模块或特点，如 "LCD"、"CAM"
- 系列名称和版本信息：一些厂商会有系列化的产品，可在命名中体现系列名称和版本号
- 其他：一些硬件标识等

### 命名规则

1. 字母必须大写
2. 字段和字段之间用下划线隔开
3. 字段结合顺序顺序，"厂家名称_核心板名称_可选字段"

示例： `ATK_T5AI_MINI_BOARD`

# 新建Board流程

## 概述

 针对开发者自己设计了一个新的硬件开发板，需要为这块新开发板做一些驱动初始化和适配工作。

在TuyaOpen目录下输入`tos.py new board`新建一款硬件开发板。

![new-project](.\img\new-board.png)

选择T5芯片型号，在终端输入`6`，然后输入Board名称。

![new-atk-t5ai-mini-board](.\img\new-atk-t5ai-mini-board.png)

我们可在`TuyaOpen\boards\T5AI`路径下找到创建的Board。

![atk-t5ai-mini-board-file](.\img\atk-t5ai-mini-board-file.png)

ATK_T5AI_MINI_BOARD相关配置，可参考TUYA_T5AI_BOARD文件夹，唯一不同的是LCD初始化操作和管脚配置，其他都是一样的初始化流程。

初学者可以不必关心里面的实现，让用起来。

## 配置查看
 
 使用`tos.py config menu`进入配置菜单，查看一下是否出现ATK_T5AI_MINI_BOARD。

![](.\img\atk_t5ai_mini_board_menu.png)

我们为板子提供了屏幕的选项以及摄像头的选项，方便直接调用到底层我们写好的代码，可以加快大家的开发速度。

还有一些外设器件的使能可以在configure device driver中找到。

![](.\img\configure_driver.png)

这里就是使能按键、LED、音频、屏幕等操作，后续涉及到的例程会提及到。

# 常见问题

## 如何删除创建的板子

直接删除 `boards/<platform>/<board_name>` 目录，并手动从`boards/<platform>/Kconfig`中移除相关配置

## 修改已创建的板子名称

需要手动修改：

1. 重命名板子目录
2. 更新平台 Kconfig 中的引用
3. 更新板子内部 Kconfig 的默认值
