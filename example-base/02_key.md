---
title: 'KEY实验'
sidebar_position: 1
---

# KEY实验

## 前言

本章实验将介绍如何使用TuyaOpen的Button组件。

## BUTTON组件介绍

### 概述

BUTTON组件是TuyaOpen中用于处理用户输入的核心组件。它提供了统一的接口来管理不同类型的按键设备，支持多种按键事件检测和处理机制。通过这个驱动，应用可以轻松地实现按键输入检测、事件处理和状态管理，而无需关心底层硬件的具体实现细节。

## API 描述

**1，tdd_gpio_button_register函数**

​	IO注册button。

```C
OPERATE_RET tdd_gpio_button_register(char *name, BUTTON_GPIO_CFG_T *gpio_cfg);
```

​	**1.1 参数描述**

|   形参   |   描述   |
| :------: | :------: |
|   name   |   名称   |
| gpio_cfg | 按键配置 |

​	`name`：按键名称（自定义）

​	`gpio_cfg`：按键配置。

```C
/* TuyaOpen/src/peripherals/button/tdd_button/include/tdd_button_gpio.h */
typedef struct {
    TUYA_GPIO_NUM_E pin;        /* 管脚编号 */
    TUYA_GPIO_LEVEL_E level;    /* 默认电平 */
    TDD_GPIO_TYPE_U pin_type;   /* 触发类型 */
    TDL_BUTTON_MODE_E mode;     /* 触发模式 */
} BUTTON_GPIO_CFG_T;

typedef union {
    TUYA_GPIO_MODE_E gpio_pull; /* for BUTTON_TIMER_SCAN_MODE */
    TUYA_GPIO_IRQ_E irq_edge;   /* for BUTTON_IRQ_MODE */
} TDD_GPIO_TYPE_U;

/* TuyaOpen/src/peripherals/button/tdl_button/include/tdl_button_driver.h */
typedef enum {
    BUTTON_TIMER_SCAN_MODE = 0, /* 扫描触发 */
    BUTTON_IRQ_MODE,            /* 外部中断触发 */
} TDL_BUTTON_MODE_E;

/* TuyaOpen/tools/porting/adapter/utilities/include/tuya_cloud_types.h */
/**
 * @brief gpio interrupt mode
 */
typedef enum {
    TUYA_GPIO_IRQ_RISE = 0,     /* 上升沿 */
    TUYA_GPIO_IRQ_FALL,         /* 下降沿 */
    TUYA_GPIO_IRQ_RISE_FALL,    /* 边沿 */
    TUYA_GPIO_IRQ_LOW,          /* 低电平 */
    TUYA_GPIO_IRQ_HIGH,         /* 高电平 */
} TUYA_GPIO_IRQ_E;
```

​	**1.2 返回值**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**2，tdl_button_create函数**

​	创建button按键。

```C
OPERATE_RET tdl_button_create(char *name, TDL_BUTTON_CFG_T *button_cfg, TDL_BUTTON_HANDLE *handle);
```

​	**2.1 参数描述**

|    形参    |   描述   |
| :--------: | :------: |
|    name    |   名称   |
| button_cfg | 按键配置 |
|   handle   | 按键句柄 |

​	`name`：被搜索按键名称

​	`button_cfg`：按键配置。

```C
typedef struct {
    uint16_t long_start_valid_time;     /* 长按有效时间 */
    uint16_t long_keep_timer;           /* 长按保持时间 */
    uint16_t button_debounce_time;      /* 防抖动时间 */
    uint8_t button_repeat_valid_count;  /* 重复有效次数 */
    uint16_t button_repeat_valid_time;  /* 重复有效时间 */
} TDL_BUTTON_CFG_T;
```

​	`handle`：返回的按键句柄

​	**2.2 返回值**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**3，tdl_button_event_register函数**

​	注册按键事件。

```C
void tdl_button_event_register(TDL_BUTTON_HANDLE handle, TDL_BUTTON_TOUCH_EVENT_E event, TDL_BUTTON_EVENT_CB cb);
```

​	**3.1 参数描述**

​	`handle`：按键句柄

​	`event`： 按键事件类型

```C
typedef enum {
    TDL_BUTTON_PRESS_DOWN = 0,     /* 按下事件 */
    TDL_BUTTON_PRESS_UP,           /* 释放事件 */
    TDL_BUTTON_PRESS_SINGLE_CLICK, /* 单击事件 */
    TDL_BUTTON_PRESS_DOUBLE_CLICK, /* 双击事件 */
    TDL_BUTTON_PRESS_REPEAT,       /* 多次点击事件 */
    TDL_BUTTON_LONG_PRESS_START,   /* 长按开始事件 */
    TDL_BUTTON_LONG_PRESS_HOLD,    /* 长按保持事件 */
    TDL_BUTTON_RECOVER_PRESS_UP,   /* 上电后保持有效电平后恢复事件 */
    TDL_BUTTON_PRESS_MAX,          /* 无 */
    TDL_BUTTON_PRESS_NONE,         /* 无 */
} TDL_BUTTON_TOUCH_EVENT_E;        /* 按键事件 */
```

​	`cb`：事件回调函数

```C
/**
 * @brief       button event callback function
 * @param[in]   name button name
 * @param[in]   event button trigger event
 * @param[in]   argc repeat count/long press time
 * @return      none
 */
typedef void (*TDL_BUTTON_EVENT_CB)(char *name, TDL_BUTTON_TOUCH_EVENT_E event, void *argc);
```

​	**3.2 返回值：**

​	无。

**本实验涉及了上述BUTTON组件函数，其他相关函数将在后续章节中结合具体应用进行详细介绍**。

## 硬件设计

### 例程功能

1，按下KEY按键可对LED状态翻转。

### 硬件资源

1，LED

​	LED - P18

2，KEY

​	KEY - P12

### 原理图

正点原子T5 AI开发板上KEY的连接原理图，如下图所示。

![](.\img\03.png)

通过以上原理图可以看出，KEY对应的IO编号分别为P12。 

## 程序设计

### 1，KEY驱动代码

这里我们只讲解核心代码，详细的源码请大家参考光盘资料本实验对应的源码。KEY驱动源码包括两个文件：key.c和key.h

key.h文件是对key引脚做了相关定义以及函数声明。

```C
/* Key status enumeration based on button events */
typedef enum {
    KEY_STATUS_PRESS_DOWN = 0,     /* Key press down status */
    KEY_STATUS_PRESS_UP,           /* Key release status */
    KEY_STATUS_SINGLE_CLICK,       /* Single click status */
    KEY_STATUS_DOUBLE_CLICK,       /* Double click status */
    KEY_STATUS_REPEAT_CLICK,       /* Repeat click status */
    KEY_STATUS_LONG_PRESS_START,   /* Long press start status */
    KEY_STATUS_LONG_PRESS_HOLD,    /* Long press hold status */
    KEY_STATUS_RECOVER_PRESS_UP,   /* Recover press up status */
    KEY_STATUS_MAX                 /* Maximum status value */
} KEY_STATUS_E;

/* Define the GPIO pin for the key
   This should match the actual GPIO pin used for the key on the hardware. */
#define KEY_GPIO_PIN    TUYA_GPIO_NUM_12

/* Function Declaration */
void key_init(void);
KEY_STATUS_E get_key_status(void);
void reset_key_status(void);
```

key.c文件是对P12管脚进行初始化和按键回调函数。

```C
#define APP_BUTTON_NAME "app_button"

KEY_STATUS_E key_status = KEY_STATUS_MAX;

/**
 * @brief       Button function callback
 * @param[in]   name: Button name
 * @param[in]   event: Button event type
 * @param[in]   argc: Additional arguments (not used here)
 * @return      none
 */
static void __button_function_cb(char *name, TDL_BUTTON_TOUCH_EVENT_E event, void *argc)
{
    argc = argc; /* Avoid unused parameter warning */
    
    switch (event)
    {
        case TDL_BUTTON_PRESS_DOWN:         /* Button press down event */
        {
            led_toggle();
            PR_NOTICE("%s: press down", name);
            key_status = KEY_STATUS_PRESS_DOWN;
        }
        break;
        case TDL_BUTTON_PRESS_UP:           /* Button release event */
        {
            PR_NOTICE("%s: press up", name);
            key_status = KEY_STATUS_PRESS_UP;
        } 
        break;
        case TDL_BUTTON_PRESS_SINGLE_CLICK: /* Single click event */
        {
            PR_NOTICE("%s: press single", name);
            key_status = KEY_STATUS_SINGLE_CLICK;
        } 
        break;
        case TDL_BUTTON_PRESS_DOUBLE_CLICK: /* Double click event */
        {
            PR_NOTICE("%s: press double", name);
            key_status = KEY_STATUS_DOUBLE_CLICK;
        } 
        break;
        case TDL_BUTTON_PRESS_REPEAT:       /* Multiple click event */
        {
            PR_NOTICE("%s: press repeat", name);
            key_status = KEY_STATUS_REPEAT_CLICK;
        } 
        break;
        case TDL_BUTTON_LONG_PRESS_START:   /* Long press start event */
        {
            PR_NOTICE("%s: long press start", name);
            key_status = KEY_STATUS_LONG_PRESS_START;
        }
        break;
        case TDL_BUTTON_LONG_PRESS_HOLD:    /* Long press hold event */
        {
            PR_NOTICE("%s: long press hold", name);
            key_status = KEY_STATUS_LONG_PRESS_HOLD;
        } 
        break;
        case TDL_BUTTON_RECOVER_PRESS_UP:   /* Recover press up event */
        {
            PR_NOTICE("%s: recover press up", name);
            key_status = KEY_STATUS_RECOVER_PRESS_UP;
        } 
        break;
        default:
            break;
    }
}

/**
 * @brief       Get the current key status
 * @param[in]   none
 * @return      KEY_STATUS_E: Current key status
 */

KEY_STATUS_E get_key_status(void)
{
    return key_status;
}

/**
 * @brief       Reset the key status to the default value
 * @param[in]   none
 * @return      none
 */
void reset_key_status(void)
{
    key_status = KEY_STATUS_MAX;
}

/**
 * @brief       KEY initialization
 * @param[in]   none
 * @return      none
 */
void key_init(void)
{
    OPERATE_RET rt = OPRT_OK;
    static TDL_BUTTON_HANDLE sg_button_hdl = NULL;
    /* Initialize the button hardware configuration */
    BUTTON_GPIO_CFG_T button_hw_cfg = {
            .pin = KEY_GPIO_PIN,                    /* GPIO pin for the key */
            .mode = BUTTON_TIMER_SCAN_MODE,         /* Using timer scan mode */
            .pin_type.gpio_pull = TUYA_GPIO_PULLUP, /* Pull-up resistor */
            .level = TUYA_GPIO_LEVEL_LOW,           /* Active low */
        };
    /* Register the button hardware configuration */
    TUYA_CALL_ERR_GOTO(tdd_gpio_button_register(APP_BUTTON_NAME, &button_hw_cfg), __EXIT);

    /* button create */
    TDL_BUTTON_CFG_T button_cfg = { .long_start_valid_time      = 3000, /* Long press start time */
                                    .long_keep_timer            = 1000, /* Long press keep time */
                                    .button_debounce_time       = 50,   /* Debounce time */
                                    .button_repeat_valid_count  = 2,    /* Repeat valid count */
                                    .button_repeat_valid_time   = 50};  /* Repeat valid time */
    /* Create the button instance */
    TUYA_CALL_ERR_GOTO(tdl_button_create(APP_BUTTON_NAME, &button_cfg, &sg_button_hdl), __EXIT);
    /* Register the button event callback */
    tdl_button_event_register(sg_button_hdl, TDL_BUTTON_PRESS_DOWN, __button_function_cb);
    /* Register the long press start event callback */
    tdl_button_event_register(sg_button_hdl, TDL_BUTTON_LONG_PRESS_HOLD, __button_function_cb);

__EXIT:
        return;
}
```

从上述key_init函数可以看出，首先我们调用tdd_gpio_button_register函数注册按键组件，然后调用tdl_button_create函数创建按键以及配置触发时间，最后调用tdl_button_event_register函数配置按键回调函数。

### 2，CMakeLists.txt文件

CMakeLists.txt文件的配置内容如下。

```
# Add Driver
set(src_dirs
    LED		# 添加驱动组件
    KEY
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
#include "key.h"
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
    led_init(); /* Initialize LED */
    key_init(); /* Initialize key */

    while (1) 
    {
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

从user_main函数可以看到，首先调用led_init、key_init函数初始化LED和KEY，当按下按键时，会切换LED状态。

## 运行验证

程序下载完成后，KEY按键控制LED灯的状态翻转。
