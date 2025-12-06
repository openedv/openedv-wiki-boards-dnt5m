---
title: 'TIMER实验'
sidebar_position: 1
---

# TIMER实验

## 前言

本章实验将介绍如何使用TuyaOpen让T5启动定时器。

## TIMER模块介绍

### 概述

timer是微处理器中用来进行时间计量的片上外设，根据实际配置要求，通常有不同的定时精度，如16位，32位等，实际使用时通常需要配置的参数有，定时周期，计数方式，中断服务程序等。

## API 描述

**1，tkl_timer_init函数**

​	初始化定时器。

```C
OPERATE_RET tkl_timer_init(TUYA_TIMER_NUM_E timer_id, TUYA_TIMER_BASE_CFG_T *cfg);
```

​	**1.1 参数描述**

|   形参   |    描述    |
| :------: | :--------: |
| timer_id | 定时器编号 |
|   cfg    | TIMER配置  |

​	`timer_id`：定时器编号：

```C
/* TuyaOpen/tools/porting/adapter/utilities/include/tuya_cloud_types.h */
/**
 * @brief timer num
 *
 */
typedef enum {
    TUYA_TIMER_NUM_0,   /* TIMER 0 */
    TUYA_TIMER_NUM_1,   /* TIMER 1 */
    TUYA_TIMER_NUM_2,   /* TIMER 2 */
    TUYA_TIMER_NUM_3,   /* TIMER 3 */
    TUYA_TIMER_NUM_4,   /* TIMER 4 */
    TUYA_TIMER_NUM_5,   /* TIMER 5 */
    TUYA_TIMER_NUM_MAX,
} TUYA_TIMER_NUM_E;
```

​	`cfg`：TIMER配置，包含定时方式，回调函数，回调函数参数。

```C
/* TuyaOpen/tools/porting/adapter/utilities/include/tuya_cloud_types.h */
typedef struct {
    TUYA_TIMER_MODE_E mode;     /* 模式 */
    TUYA_TIMER_ISR_CB cb;       /* 定时器中断服务回调 */
    void *args;                 /* 定时器中断服务传参 */
} TUYA_TIMER_BASE_CFG_T;

typedef enum { 
    TUYA_TIMER_MODE_ONCE = 0,   /* 单次 */
    TUYA_TIMER_MODE_PERIOD      /* 周期 */
} TUYA_TIMER_MODE_E;

typedef void (*TUYA_TIMER_ISR_CB)(void *args);  /* 定时器中断服务函数 */
```

​	**1.2 返回值**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**2，tkl_timer_start函数**

​	开启定时器。

```C
OPERATE_RET tkl_timer_start(TUYA_TIMER_NUM_E timer_id, UINT_T us);
```

​	**2.1 参数描述**

​	`timer_id`：GPIO 引脚编号

​	`us`：定时周期（us）

​	**2.2 返回值**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**3，tkl_timer_stop函数**

​	停止定时器。

```C
OPERATE_RET tkl_timer_stop(TUYA_TIMER_NUM_E timer_id);
```

​	**3.1 参数描述**

​	`timer_id`：GPIO 引脚编号

​	**3.2 返回值：**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**4，tkl_timer_deinit函数**

​	卸载定时器。

```C
OPERATE_RET tkl_timer_deinit(TUYA_TIMER_NUM_E timer_id);
```

​	**4.1 参数描述**

​	`timer_id`：GPIO 引脚编号

​	**4.2 返回值：**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**本实验涉及了上述TIMER函数，其他相关函数将在后续章节中结合具体应用进行详细介绍**。

## 硬件设计

### 例程功能

1，启动定时器，在1s周期内翻转LED状态。

### 硬件资源

1，LED

​	LED - P18

### 原理图

正点原子T5 AI开发板上LED的连接原理图，如下图所示。

![](.\img\02.png)

## 程序设计

### 1，TIMER驱动代码

这里我们只讲解核心代码，详细的源码请大家参考光盘资料本实验对应的源码。TIMER驱动源码包括两个文件：timer.c和timer.h

timer.h文件是对TIMER引脚做了相关定义以及函数声明。

```C
/* Define the timer ID
   This should match the actual timer ID used in the hardware configuration. */
#define TIMER_ID    TUYA_TIMER_NUM_3

/* Function Declaration */
void g_timer_init(uint32_t us);  /* Function to initialize the timer */
```

timer.c文件是对定时器3进行初始化，并配置定时中断服务函数。

```C
/**
 * @brief       Timer callback function
 *
 * @param[in]   arg:parameters
 * @return      none
 */
static void __timer_callback(void *args)
{
    args = args;  /* Avoid unused parameter warning */
    /* TAL_PR_ , PR_ , these two types of prints have locks inside, do not use them in interrupts */
    tkl_log_output("\r\n------------ TIMER Callbcak ------------\r\n");
    led_toggle();  /* Toggle the LED state */
}

/**
 * @brief       Initialize the timer
 *
 * @param[in]   us: The value to set the timer
 * @return      none
 */
void g_timer_init(uint32_t us)
{
    OPERATE_RET rt = OPRT_OK;
    /* Convert microseconds to timer ticks */
    TUYA_TIMER_BASE_CFG_T sg_timer_cfg = {.mode = TUYA_TIMER_MODE_PERIOD, .args = NULL, .cb = __timer_callback};
    /* Calculate the timer value based on the input microseconds */
    TUYA_CALL_ERR_LOG(tkl_timer_init(TIMER_ID, &sg_timer_cfg));
    /* start timer */
    TUYA_CALL_ERR_LOG(tkl_timer_start(TIMER_ID, us));
}
```

上述源码中，调用tkl_timer_init函数配置周期模式以及设置定时中断服务函数，然后调用tkl_timer_start函数开启定时器中断。

### 2，CMakeLists.txt文件

CMakeLists.txt文件配置内容如下。

```
# Add Driver
set(src_dirs
    LED		# 添加驱动组件
    TIMER
)

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

### 3，main.c驱动代码

在main.c里面编写如下代码。

```C
#include "tal_api.h"
#include "tkl_output.h"
#include "tal_cli.h"
#include "led.h"
#include "timer.h"

/**
 * @brief       user_main
 *
 * @param[in]   none
 * @return      none
 */
void user_main(void)
{
    tal_log_init(TAL_LOG_LEVEL_DEBUG, 1024, (TAL_LOG_OUTPUT_CB)tkl_log_output); /* Initialize log output */
    led_init();             /* Initialize LED */
    g_timer_init(1000000);  /* Initialize timer with 1 second interval */

    while (1) {
        PR_NOTICE("\r\n---------------------------\r\n");
        tal_system_sleep(10000);     /* Sleep for 1000 ms */
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

从user_main函数可以看到，首先调用led_init、g_timer_init函数初始化LED和定时器，其中定时器定时事件为1000000us（1s）。

## 运行验证

程序下载完成后，可看到LED以每次1000ms闪烁。
