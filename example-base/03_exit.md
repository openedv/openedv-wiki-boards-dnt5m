---
title: 'EXIT实验'
sidebar_position: 1
---

# EXIT实验

## 前言

本章实验将介绍如何使用TuyaOpen让T5实现外部中断。

## EXIT模块介绍

### 概述

外部中断属于硬件中断，由微控制器外部事件触发。微控制器的特定引脚被设计为对特定事件（如按钮按压、传感器信号变化等）作出响应，这些引脚通常称为“外部中断引脚”。一旦外部中断事件发生，当前程序执行将立即暂停，并跳转到相应的中断服务程序（ISR）进行处理。 处理完毕后，程序会恢复执行，从被中断的地方继续。

## API 描述

**1，tkl_gpio_irq_init函数**

​	GPIO中断初始化。

```C
OPERATE_RET tkl_gpio_irq_init(TUYA_GPIO_NUM_E pin_id, const TUYA_GPIO_IRQ_T *cfg);
```

​	**1.1 参数描述**

|  形参  |   描述   |
| :----: | :------: |
| pin_id |  管脚号  |
|  cfg   | GPIO中断配置 |

​	`pin_id`：GPIO 引脚编号，此编号区别于芯片原始引脚号，是由涂鸦根据芯片 PA、PB ... PN 上的引脚个数顺序往下编号的：

```C
/* TuyaOpen/tools/porting/adapter/utilities/include/tuya_cloud_types.h */
typedef enum {
    TUYA_GPIO_NUM_0,  /* GPIO 0 */
    TUYA_GPIO_NUM_1,  /* GPIO 1 */
    TUYA_GPIO_NUM_2,  /* GPIO 2 */
    ............................
    TUYA_GPIO_NUM_MAX,
} TUYA_GPIO_NUM_E;
```

​	`cfg`：GPIO中断配置。

```C
/* TuyaOpen/platform/T5AI/tuyaos/tuyaos_adapter/include/utilities/include/tuya_cloud_types.h */
/**
 * @brief gpio interrupt config
 */
typedef struct {
    TUYA_GPIO_IRQ_E      mode;	/* 中断模式 */
    TUYA_GPIO_IRQ_CB     cb;	/* 中断服务函数 */
    VOID_T              *arg;	/* 中断服务函数传参 */
} TUYA_GPIO_IRQ_T;

/**
 * @brief gpio interrupt mode
 */
typedef enum {
    TUYA_GPIO_IRQ_RISE  = 0,    /* 上升沿 */
    TUYA_GPIO_IRQ_FALL,         /* 下降沿 */
    TUYA_GPIO_IRQ_RISE_FALL,    /* 边沿 */
    TUYA_GPIO_IRQ_LOW,          /* 低电平 */
    TUYA_GPIO_IRQ_HIGH,         /* 高电平 */
} TUYA_GPIO_IRQ_E;

typedef VOID_T (*TUYA_GPIO_IRQ_CB)(VOID_T *args);
```

​	**1.2 返回值**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**2，tkl_gpio_irq_enable函数**

​	GPIO中断使能。

```C
OPERATE_RET tkl_gpio_irq_enable(TUYA_GPIO_NUM_E pin_id);
```

​	**2.1 参数描述**

​	`pin_id`：GPIO 引脚编号

​	**2.2 返回值**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**本实验涉及了上述GPIO中断函数，其他相关函数将在后续章节中结合具体应用进行详细介绍**。

## 硬件设计

### 例程功能

1，按下KEY按键切换LED状态。

### 硬件资源

1，LED

​	LED - P18

2，KEY

​	KEY - P12

### 原理图

正点原子T5 AI开发板上EXIT的连接原理图，如下图所示。

![](.\img\03.png)

## 程序设计

### 1，EXIT驱动代码

这里我们只讲解核心代码，详细的源码请大家参考光盘资料本实验对应的源码。EXIT驱动源码包括两个文件：exit.c和exit.h

exit.h文件是对EXIT引脚做了相关定义以及函数声明。

```C
/* Define the GPIO pin for the key
   This should match the actual GPIO pin used for the key on the hardware. */
#define EXIT_GPIO_PIN    TUYA_GPIO_NUM_12

/* Function Declaration */
void exit_init(void);    /* Initialize the exit key */
```

exit.c文件是对P12管脚进行初始化和中断配置。

```C
/**
 * @brief       interrupt callback function
 * @param[in]   args:parameters
 * @return      none
 */
static void __gpio_irq_callback(void *args)
{
    args = args;    /* Suppress unused parameter warning */
    tkl_log_output("\r\n------------ GPIO IRQ Callbcak ------------\r\n");
    led_toggle();   /* Toggle the LED state */
}

/**
 * @brief       EXIT initialization
 * @param[in]   none
 * @return      none
 */
void exit_init(void)
{
    OPERATE_RET rt = OPRT_OK;

    /* GPIO input init */
    TUYA_GPIO_BASE_CFG_T in_pin_cfg = {
        .mode = TUYA_GPIO_PULLUP,
        .direct = TUYA_GPIO_INPUT,
    };
    /* GPIO init */
    TUYA_CALL_ERR_LOG(tkl_gpio_init(EXIT_GPIO_PIN, &in_pin_cfg));
    TUYA_GPIO_IRQ_T irq_cfg = {
        .cb = __gpio_irq_callback,
        .arg = NULL,
        .mode = TUYA_GPIO_IRQ_RISE,
    };
    /* GPIO irq init */
    TUYA_CALL_ERR_LOG(tkl_gpio_irq_init(EXIT_GPIO_PIN, &irq_cfg));

    /* irq enable */
    TUYA_CALL_ERR_LOG(tkl_gpio_irq_enable(EXIT_GPIO_PIN));
}
```

上述源码可以看到，首先我们调用tkl_gpio_init函数配置P12为输入和上拉，然后调用tkl_gpio_irq_init函数配置该管脚的外部中断触发条件以及中断服务函数，最后调用tkl_gpio_irq_enable开启外部中断。__gpio_irq_callback中断服务函数中，led灯状态翻转。

### 2，CMakeLists.txt文件

CMakeLists.txt文件配置内容如下。

```
# Add Driver
set(src_dirs
    LED		# 添加驱动组件
    EXIT
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
#include "exit.h"
#include "test.h"

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
    led_init();     /* Initialize LED */
    exit_init();    /* Initialize exit key */

    while (1) {
        PR_DEBUG("cnt is %d", cnt++);
        tal_system_sleep(10000);
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

## 运行验证

程序下载完成后，按下KEY按键可切换LED灯状态。
