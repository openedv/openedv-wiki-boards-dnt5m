---
title: '基础例程'
sidebar_position: 1
---

# 基础例程

## 前言

本章节主要介绍如何在TuyaOpen下，新建一个项目工程。  

## 创建`project`

### 概述

`tos.py new project` 命令用于在 TuyaOpen 开发环境中创建新项目，该命令会基于预定义的模板快速初始化一个新项目的基础结构。

## 操作原理

1，执行命令`tos.py new project`，系统会提示输入新项目的名称和选择平台

```
❯ tos.py new project
[INFO]: Running tos.py ...
[NOTE]: Input new project name.
input: 00_base
```

命令执行完成后，系统会在当前目录下创建一个名为 `00_base` 的文件夹，里面包含了新项目的基本结构。

2，目录结构

```
00_base
├── app_default.config  # 默认配置文件
├── CMakeLists.txt      # CMake 构建配置
└── src
    └── hello_world.c   # 示例代码文件
```

## 完善项目工程架构

为了方便管理自定义的BSP驱动和第三方组件，我们对基础工程进行工程完善。

1，管理组件驱动

在`00_base`文件夹下新建`components`文件夹，用来存储`BSP`文件夹和`Middlewares`文件夹。

```
00_base
├── components
    ├── BSP                 # 存储LED、KEY...等驱动
        ├── BSP_T           # BSP_T驱动
            ├── bsp.c
            └── bsp.h
        └── CMakeLists.txt  # 把驱动加载到构建系统
    └── Middlewares         # 存储第三方组件，如图像解码库....
        ├── Test            # 第三方组件
        └── CMakeLists.txt  # 把第三方组件加载到构建系统
├── app_default.config      # 默认配置文件
├── CMakeLists.txt          # CMake 构建配置
└── src
    └── hello_world.c       # 示例代码文件
```

`components/BSP/BSP_T/CMakeLists.txt`内容如下。

```
# Add Driver
set(src_dirs
    BSP_T       # 添加驱动，如LED、KEY等驱动
)

# 把BSP驱动加载到构建系统
foreach(dir ${src_dirs})
    set(SRC_DIR ${APP_PATH}/components/BSP/${dir})
    aux_source_directory(${SRC_DIR} DRIVE_SRC)
    target_sources(${EXAMPLE_LIB}
        PRIVATE
            ${DRIVE_SRC}
    )
    target_include_directories(${EXAMPLE_LIB}
        PRIVATE
            ${SRC_DIR}
    )
endforeach()
```

2，编译链接组件驱动

为了能引用`BSP`驱动和第三方组件，我们必须在`CMakeLists.txt`文件内容尾端添加以下代码段。

```
add_subdirectory(${APP_PATH}/components/BSP)            # 添加BSP子目录
add_subdirectory(${APP_PATH}/components/Middlewares)    # 添加Middlewares子目录
```

CMakeLists.txt的内容懂得如何修改即可,不需要深究其原理。

3，重命名

`src/hello_world.c`重命名为`main.c`。

```
00_base
├── components
    ├── BSP                 # 存储LED、KEY...等驱动
        ├── BSP_T           # BSP_T驱动
            ├── bsp.c
            └── bsp.h
        └── CMakeLists.txt  # 把驱动加载到构建系统
    └── Middlewares         # 存储第三方组件，如图像解码库....
        ├── Test            # 第三方组件
        └── CMakeLists.txt  # 把第三方组件加载到构建系统
├── app_default.config      # 默认配置文件
├── CMakeLists.txt          # CMake 构建配置
└── src
    └── main.c              # 示例代码文件
```

4，项目固化配置

输入`tos.py config choice`命令，选择`T5AI.config`。

```
[INFO]: Running tos.py ...
[NOTE]: Fullclean success.
--------------------
1. BK7231X.config
2. ESP32-C3.config
3. ESP32-S3.config
4. ESP32.config
5. EWT103-W15.config
6. LN882H.config
7. T2.config
8. T3.config
9. T5AI.config
10. Ubuntu.config
--------------------
Input "q" to exit.
Choice config file: 9
```

输入`tos.py config menu`命令，选择ATK_T5AI_MINI_BOARD板子配置参数。

![01.png](.\img\01.png)

英文输入法模式下，按下键盘的`s`，回车保存配置和`Esc`退出配置。

接着输入`tos.py config save`保存固化配置。

```
[INFO]: Running tos.py ...
Input save config name: atk_t5_config          
[NOTE]: Success save: C:/TuyaOpen/examples/test/config/atk_t5_config.config
```

后续有了这个配置文件，就不需要`tos.py config menu`操作，直接通过`tos.py config choice`选择配置即可。

最后的基础工程结构目录如下。

```
00_base
├── components
    ├── BSP                 # 存储LED、KEY...等驱动
        ├── BSP_T           # BSP_T驱动
            ├── bsp.c
            └── bsp.h
        └── CMakeLists.txt  # 把驱动加载到构建系统
    └── Middlewares         # 存储第三方组件，如图像解码库....
        ├── Test            # 第三方组件
        └── CMakeLists.txt  # 把第三方组件加载到构建系统
├── config
    └── atk_config.config
├── app_default.config      # 默认配置文件
├── CMakeLists.txt          # CMake 构建配置
└── src
    └── main.c              # 示例代码文件
```

5，修改启动方式

为了让初学者循序渐进的学习，这里我们把main.c启动方式编写为**类似**裸机的方式启动（实际上tuya_app_main就是一个任务函数）。

```C
/**
 ****************************************************************************************************
 * @file        main.c
 * @author      ALIENTEK
 * @brief       BASE DEMO
 * @license     Copyright (C) 2020-2030, ALIENTEK
 ****************************************************************************************************
 * @attention
 *
 * platform     : ALIENTEK T5 MINI Board
 * website      : www.alientek.com
 * forum        : www.openedv.com/forum.php
 *
 * change logs  :
 * version      data         notes
 * V1.0         20250725     the first version
 *
 ****************************************************************************************************
 */

#include "tal_api.h"
#include "tkl_output.h"
#include "tal_cli.h"

/**
 * @brief       user_main
 *
 * @param[in]   none
 * @return      none
 */
void user_main(void)
{
    int cnt = 0;
    tal_log_init(TAL_LOG_LEVEL_DEBUG, 1024, (TAL_LOG_OUTPUT_CB)tkl_log_output); /* Initialize log output */

    while (1) {
        PR_DEBUG("cnt is %d", cnt++);
        tal_system_sleep(1000); /* Sleep for 1000 ms */
    }
}

/**
 * @brief       main
 *
 * @param       argc
 * @param       argv
 * @return      void
 */
#if OPERATING_SYSTEM == SYSTEM_LINUX
void main(int argc, char *argv[])
{
    user_main();

    while (1) {
        tal_system_sleep(500);
    }
}
#else

/**
 * @brief       The application entry point
 * @param[in]   none
 * @return      none
 */
void tuya_app_main(void)
{
    user_main();
}

#endif
```

6，编译程序

命令行中输入`tos.py build`，进行项目编译。

```
[NOTE]:
====================[ BUILD SUCCESS ]===================
 Target    : 00_base_QIO_1.0.0.bin
 Output    : L:\t5_ai_board\fork_tuyaOpen\TuyaOpen\base_routine\00_base\dist\00_base_1.0.0
 Platform  : T5AI
 Chip      : T5AI
 Board     : ATK_T5AI_MINI_BOARD
 Framework : base
========================================================
```

7，烧录程序

命令行中输入`tos.py flash`，进行程序烧录。

```
[INFO]: Reboot done
[INFO]: Flash write success.
```

## 硬件设计

### 例程功能

1，串口打印cnt累加数值。（注意：① 切换跳线帽使用板载的TypeC口连接串口助手 ② 使用USB转串口模块连接LGT和LGR再连接串口助手）

### 硬件资源

1，串口

​	TXD - P0

​	RXD - P1

### 原理图

无

## 运行验证

```
ap0:W(215):go to tuya
ap0:W(216):[01-01 00:00:00 ty D][main.c:37] cnt is 0
ap1:W(1216):[01-01 00:00:01 ty D][main.c:37] cnt is 1
ap1:W(2216):[01-01 00:00:02 ty D][main.c:37] cnt is 2
ap1:W(3216):[01-01 00:00:03 ty D][main.c:37] cnt is 3
ap1:W(4216):[01-01 00:00:04 ty D][main.c:37] cnt is 4
ap1:W(5216):[01-01 00:00:05 ty D][main.c:37] cnt is 5
ap1:W(6216):[01-01 00:00:06 ty D][main.c:37] cnt is 6
ap1:W(7216):[01-01 00:00:07 ty D][main.c:37] cnt is 7
```

