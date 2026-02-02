---
title: 'LED实验'
sidebar_position: 1
---

# LED实验

## 前言

本章实验将介绍如何使用TuyaOpen让T5控制板载的LED闪烁。

## GPIO模块介绍

### 概述

GPIO（General Purpose I/O Ports）是通用输入/输出端口，通俗地说，就是一些引脚，可以通过它们输出高低电平或者通过它们读入引脚的状态是高电平或是低电平。

**注意：模组引脚都有复用功能，可查看“platform\T5AI\t5_os\projects\tuya_app\ap\config\bk7258_ap\usr_gpio_cfg.h”文件。当某个外设使用时，可能这个引脚没有用到，但是属于该外设功能相关的引脚，那这个引脚也是不能作为普通IO使用。比如我们现在的LED引脚是P18，当使用LCD外设时，它就作为GPIO_DEV_LCD_VSYNC使用，所以这时候你把它作为一个普通的IO使用，是无效的**

## API 描述

**1，tkl_gpio_init函数**

GPIO初始化函数。

```C
OPERATE_RET tkl_gpio_init(TUYA_GPIO_NUM_E pin_id, const TUYA_GPIO_BASE_CFG_T *cfg);
```

**1.1 参数描述**

|  形参  |   描述   |
| :----: | :------: |
| pin_id | 管脚编号 |
|  cfg   | GPIO配置 |

`pin_id`：GPIO 引脚编号，此编号区别于芯片原始引脚号，是由涂鸦根据芯片 PA、PB ... PN 上的引脚个数顺序往下编号的：

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

`cfg`：GPIO 基础配置。

```C
/* TuyaOpen/tools/porting/adapter/utilities/include/tuya_cloud_types.h */

/**
 * @brief gpio config
 */
typedef struct {
    TUYA_GPIO_MODE_E mode;      /* 模式配置，请看TUYA_GPIO_MODE_E枚举 */
    TUYA_GPIO_DRCT_E direct;    /* 方向配置，请看TUYA_GPIO_DRCT_E枚举 */
    TUYA_GPIO_LEVEL_E level;    /* 电平配置：请看TUYA_GPIO_LEVEL_E枚举 */
} TUYA_GPIO_BASE_CFG_T;

/**
 * @brief gpio level
 */
typedef enum {
    TUYA_GPIO_LEVEL_LOW = 0,    /* 输出低电平 */
    TUYA_GPIO_LEVEL_HIGH,       /* 输出高电平 */
    TUYA_GPIO_LEVEL_NONE,       /* 高阻态 */
} TUYA_GPIO_LEVEL_E;

/**
 * @brief gpio direction
 */
typedef enum {
    TUYA_GPIO_INPUT = 0,        /* 输入模式 */
    TUYA_GPIO_OUTPUT,           /* 输出模式 */
} TUYA_GPIO_DRCT_E;

/**
 * @brief gpio mode
 */
typedef enum {
    TUYA_GPIO_PULLUP = 0,       /* 输入模式: 上拉配置 */
    TUYA_GPIO_PULLDOWN,         /* 输入模式：下拉配置 */
    TUYA_GPIO_HIGH_IMPEDANCE,   /* 输入模式：高阻态 */
    TUYA_GPIO_FLOATING,         /* 输入模式：浮空输入 */
    TUYA_GPIO_PUSH_PULL,        /* 输出模式：推挽输出 */
    TUYA_GPIO_OPENDRAIN,        /* 输出模式：开漏输出 */
    TUYA_GPIO_OPENDRAIN_PULLUP, /* 输出模式：开漏且上拉输出 */
} TUYA_GPIO_MODE_E;

```

**1.2 返回值**

OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**2，tkl_gpio_deinit函数**

GPIO 恢复初始状态。

```C
OPERATE_RET tkl_gpio_deinit(TUYA_GPIO_NUM_E pin_id);
```

**2.1 参数描述**

`pin_id`：GPIO 引脚编号

**2.2 返回值**

OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**3，tkl_gpio_write函数**

GPIO 输出电平。

```C
OPERATE_RET tkl_gpio_write(TUYA_GPIO_NUM_E pin_id, TUYA_GPIO_LEVEL_E level);
```

**3.1 参数描述**

`pin_id`：GPIO 引脚编号

`level`： GPIO 输出电平

**3.2 返回值：**

OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**本实验涉及了上述GPIO函数，其他相关函数将在后续章节中结合具体应用进行详细介绍**。

## 硬件设计

### 例程功能

1，在1秒的周期内，LED的电平状态回发生翻转。

### 硬件资源

1，LED

LED - P18

### 原理图

本章实验内容，需要控制板载LED闪烁，正点原子T5 AI开发板上LED的连接原理图，如下图所示。

![](.\img\02.png)

通过以上原理图可以看出，LED对应的IO编号分别为P18。当P18输出低电平时LED亮起，当P18输出高电平时LED熄灭。 

## 程序设计

在01_led例程中，我们在`01_led\components\BSP`路径下新建了一个LED文件夹和CMakeLists.txt文件。其中，LED文件夹用于存放LED驱动，而CMakeLists.txt文件则用于将LED驱动添加至构建系统，以便项目工程能够使用LED驱动功能。

### 1，LED驱动代码

这里我们只讲解核心代码，详细的源码请大家参考光盘资料本实验对应的源码。LED驱动源码包括两个文件：led.c和led.h

led.h文件是对LED引脚做了相关定义以及函数声明。

```C
/* Define the GPIO pin for the LED
   This should match the actual GPIO pin used for the LED on the hardware. */
#define LED_GPIO_PIN    TUYA_GPIO_NUM_18

/* Function Declaration */
void led_init(void);                /* Function to initialize the LED */
void led_toggle(void);              /* Function to toggle the LED state */
void led_set_state(bool state);     /* Function to set the LED state */
```

led.c文件是对P18管脚进行初始化和电平配置。

```C
static bool led_state = true;

/**
 * @brief       LED initialization
 * @param[in]   none
 * @return      none
 */
void led_init(void)
{
    OPERATE_RET rt = OPRT_OK;

    TUYA_GPIO_BASE_CFG_T out_pin_cfg =  /* LED GPIO configuration */
    {
        .mode = TUYA_GPIO_PUSH_PULL,    /* Push-pull mode */
        .direct = TUYA_GPIO_OUTPUT,     /* Output direction */
        .level = TUYA_GPIO_LEVEL_NONE   /* Initial level, will be set later */
    };
 
    TUYA_CALL_ERR_LOG(tkl_gpio_init(LED_GPIO_PIN, &out_pin_cfg));  /* Initialize LED GPIO pin */
}

/**
 * @brief       Set the LED state
 * @param[in]   state: true for ON, false for OFF
 * @return      none
 */
void led_set_state(bool state)
{
    led_state = state;  /* Set the LED state */
    tkl_gpio_write(LED_GPIO_PIN, led_state ? TUYA_GPIO_LEVEL_HIGH : TUYA_GPIO_LEVEL_LOW);  /* Write the state to the GPIO pin */
}

/**
 * @brief       Toggle the LED state
 * @param[in]   none
 * @return      none
 */
void led_toggle(void)
{
    led_state = !led_state;
    tkl_gpio_write(LED_GPIO_PIN,led_state ? TUYA_GPIO_LEVEL_HIGH : TUYA_GPIO_LEVEL_LOW);
}
```

上述源码中，led_init函数用来配置P18管脚输出电平，然后led_set_state和led_toggle函数分别设置管脚电平和电平翻转功能。

### 2，CMakeLists.txt文件

本例程的功能实验主要依靠LED驱动。要在main函数中，成功调用LED文件中的函数，就得需要新建和配置BSP文件夹下得CMakeLists.txt文件，配置内容如下。

```
# Add Driver
set(src_dirs
    LED		# 添加驱动组件
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

通过添加驱动组件的方式，把驱动组件添加到构建系统当中。

> [!CAUTION]
>
> 在以后得章节中，我们不会再一次新建CMakeLists.txt文件，我们会在此文件上略作修改，就可以把其他驱动添加至构建系统当中。

### 3，main.c驱动代码

在main.c里面编写如下代码。

```C
#include "tal_api.h"
#include "tkl_output.h"
#include "tal_cli.h"
#include "led.h"
#include "test.h"

/**
 * @brief       user_main
 *
 * @param[in]   none
 * @return      none
 */
void user_main(void)
{
    tal_log_init(TAL_LOG_LEVEL_DEBUG, 1024, (TAL_LOG_OUTPUT_CB)tkl_log_output); /* Initialize log output */
    led_init(); /* Initialize LED */

    while (1) 
    {
        led_toggle();           /* Toggle LED state */
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

从user_main函数可以看到，首先调用led_init函数初始化LED，然后调用led_toggle函数对LED状态进行翻转。

## 运行验证

程序下载完成后，可看到LED以每次1000ms闪烁。
