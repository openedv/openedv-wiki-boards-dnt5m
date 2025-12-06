---
title: 'PWM实验'
sidebar_position: 1
---

# PWM实验

## 前言

本章实验将介绍如何使用TuyaOpen让T5控制板载的LED实现呼吸灯的效果。

## PWM模块介绍

### 概述

PWM（Pulse Width Modulation）,即脉冲宽度调制，其是利用微处理器的数字输出来对模拟电路进行控制的一种有效的技术。

## API 描述

**1，tkl_io_pinmux_config函数**

​	GPIO复用函数。

```C
OPERATE_RET tkl_io_pinmux_config(TUYA_PIN_NAME_E pin, TUYA_PIN_FUNC_E pin_func);
```

​	**1.1 参数描述**

|   形参   |   描述   |
| :------: | :------: |
|   pin    | 管脚编号 |
| pin_func | 复用配置 |

​	`pin`：GPIO 引脚编号，此编号区别于芯片原始引脚号，是由涂鸦根据芯片 PA、PB ... PN 上的引脚个数顺序往下编号的：

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

​	`pin_func`：复用配置。

```C
/* TuyaOpen/tools/porting/adapter/utilities/include/tuya_cloud_types.h */
/**
 * @brief tuya pinmux default func define
 */
#define  TUYA_IIC0_SCL       0x0
#define  TUYA_IIC0_SDA       0x1
#define  TUYA_IIC1_SCL       0x2
#define  TUYA_IIC1_SDA       0x3
#define  TUYA_IIC2_SCL       0x4
#define  TUYA_IIC2_SDA       0x5
#define  TUYA_IIC3_SCL       0x6
#define  TUYA_IIC3_SDA       0x7
#define  TUYA_IIC4_SCL       0x8
#define  TUYA_IIC4_SDA       0x9
#define  TUYA_IIC5_SCL       0xA
#define  TUYA_IIC5_SDA       0xB

#define  TUYA_UART0_TX       0x100
#define  TUYA_UART0_RX       0x101
#define  TUYA_UART0_RTS      0x102
#define  TUYA_UART0_CTS      0x103
#define  TUYA_UART1_TX       0x104
#define  TUYA_UART1_RX       0x105
#define  TUYA_UART1_RTS      0x106
#define  TUYA_UART1_CTS      0x107
#define  TUYA_UART2_TX       0x108
#define  TUYA_UART2_RX       0x109
#define  TUYA_UART2_RTS      0x10A
#define  TUYA_UART2_CTS      0x10B
#define  TUYA_UART3_TX       0x10C
#define  TUYA_UART3_RX       0x10D
#define  TUYA_UART3_RTS      0x10E
#define  TUYA_UART3_CTS      0x10F

#define  TUYA_SPI0_MISO      0x200
#define  TUYA_SPI0_MOSI      0x201
#define  TUYA_SPI0_CLK       0x202
#define  TUYA_SPI0_CS        0x203
#define  TUYA_SPI1_MISO      0x204
#define  TUYA_SPI1_MOSI      0x205
#define  TUYA_SPI1_CLK       0x206
#define  TUYA_SPI1_CS        0x207
#define  TUYA_SPI2_MISO      0x208
#define  TUYA_SPI2_MOSI      0x209
#define  TUYA_SPI2_CLK       0x20A
#define  TUYA_SPI2_CS        0x20B

#define  TUYA_PWM0           0x300
#define  TUYA_PWM1           0x301
#define  TUYA_PWM2           0x302
#define  TUYA_PWM3           0x303
#define  TUYA_PWM4           0x304
#define  TUYA_PWM5           0x305

#define  TUYA_ADC0           0x400
#define  TUYA_ADC1           0x401
#define  TUYA_ADC2           0x402
#define  TUYA_ADC3           0x403
#define  TUYA_ADC4           0x404
#define  TUYA_ADC5           0x405

#define  TUYA_DAC0           0x500
#define  TUYA_DAC1           0x501
#define  TUYA_DAC2           0x502
#define  TUYA_DAC3           0x503
#define  TUYA_DAC4           0x504
#define  TUYA_DAC5           0x505

#define  TUYA_I2S0_SCK       0x600
#define  TUYA_I2S0_WS        0x601
#define  TUYA_I2S0_SDO_0     0x602
#define  TUYA_I2S0_SDI_0     0x603
#define  TUYA_I2S1_SCK       0x604
#define  TUYA_I2S1_WS        0x605
#define  TUYA_I2S1_SDO_0     0x606
#define  TUYA_I2S1_SDI_0     0x607
```

​	**1.2 返回值**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**2，tkl_pwm_init函数**

​	PWM初始化。

```C
OPERATE_RET tkl_pwm_init(TUYA_PWM_NUM_E ch_id, CONST TUYA_PWM_BASE_CFG_T *cfg);
```

​	**2.1 参数描述**

​	`ch_id`：通道号（请参考对应的管脚来配置PWM通道）

```C
/* TuyaOpen/tools/porting/adapter/utilities/include/tuya_cloud_types.h */
/**
 * @brief PWM flag
 *
 */
typedef enum {
    TUYA_PWM_NUM_0,     /* PWM 0 */
    TUYA_PWM_NUM_1,     /* PWM 1 */
    TUYA_PWM_NUM_2,     /* PWM 2 */
    TUYA_PWM_NUM_3,     /* PWM 3 */
    TUYA_PWM_NUM_4,     /* PWM 4 */
    TUYA_PWM_NUM_5,     /* PWM 5 */
    TUYA_PWM_NUM_MAX,
} TUYA_PWM_NUM_E;
```

​	`cfg`：PWM基础配置，包含输出极性，占空比，频率。

```C
/* TuyaOpen/tools/porting/adapter/utilities/include/tuya_cloud_types.h */
/**
 * @brief pwm config
 */
typedef struct {
    TUYA_PWM_POLARITY_E polarity;   /* 极性 */
    TUYA_PWM_COUNT_E count_mode;    /* PWM计数方式 */
    /* pulse duty cycle = duty / cycle; exp duty = 5000,cycle = 10000; pulse duty cycle = 50% */
    uint32_t duty;                  /* 占空比 */
    uint32_t cycle;                 /* 周期 */
    uint32_t frequency;             /* 频率 */
} TUYA_PWM_BASE_CFG_T;

/**
 * @brief pwm polarity
 */
typedef enum {
    TUYA_PWM_NEGATIVE = 0,          /* 阴极 */
    TUYA_PWM_POSITIVE,              /* 阳极 */
} TUYA_PWM_POLARITY_E;

/**
 * @brief pwm count mode
 */
typedef enum {
    TUYA_PWM_CNT_UP = 0,            /* 向上计数 */
    TUYA_PWM_CNT_UP_AND_DOWN,       /* 可在双工互补模式下使用 */
} TUYA_PWM_COUNT_E;
```

​	**2.2 返回值**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**3，tkl_pwm_start函数**

​	开启PWM。

```C
OPERATE_RET tkl_pwm_start(TUYA_PWM_NUM_E ch_id);
```

​	**3.1 参数描述**

​	`ch_id`：通道号（请参考对应的管脚来配置PWM通道）

​	**3.2 返回值：**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**4，tkl_pwm_stop函数**

​	停止PWM。

```C
OPERATE_RET tkl_pwm_stop(TUYA_PWM_NUM_E ch_id);
```

​	**4.1 参数描述**

​	`ch_id`：通道号（请参考对应的管脚来配置PWM通道）

​	**4.2 返回值：**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**5，tkl_pwm_duty_set函数**

​	设置PWM占空比。

```C
OPERATE_RET tkl_pwm_duty_set(TUYA_PWM_NUM_E ch_id, UINT32_T duty);
```

​	**5.1 参数描述**

​	`ch_id`：通道号（请参考对应的管脚来配置PWM通道）

​	`duty`：设置占空比

​	**5.2 返回值：**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**本实验涉及了上述PWM函数，其他相关函数将在后续章节中结合具体应用进行详细介绍**。

## 硬件设计

### 例程功能

1，利用PWM原理，实现呼吸灯实验。

### 硬件资源

1，LED

​	LED - P18

2，PWM

​	PWM - PWM0（P18）

### 原理图

正点原子T5 AI开发板上LED的连接原理图，如下图所示。

![](.\img\02.png)

## 程序设计

### 1，PWM驱动代码

这里我们只讲解核心代码，详细的源码请大家参考光盘资料本实验对应的源码。PWM驱动源码包括两个文件：pwm.c和pwm.h

pwm.h文件是对PWM引脚做了相关定义以及函数声明。

```C
#define PWM_DUTY           0              /* 0% duty */
#define PWM_FREQUENCY      1000           /* 1kHz frequency */
#define TASK_PWM_PRIORITY  THREAD_PRIO_2  /* Task priority for PWM thread */
#define TASK_PWM_SIZE      4096           /* Stack size for PWM task */
#define PWM_ID             TUYA_PWM_NUM_0 /* PWM ID for the first PWM channel */

/* Function Declaration */
void pwm_init(void);                /* Function to initialize the PWM */
void pwm_set_duty(uint32_t duty);   /* Function to set the PWM duty cycle */
```

pwm.c文件是对P18管脚进行PWM复用和初始化。

```C
/**
 * @brief       PWM initialization
 * @param[in]   none
 * @return      none
 */
void pwm_init(void)
{
    OPERATE_RET rt = OPRT_OK;
    tkl_io_pinmux_config(TUYA_GPIO_NUM_18, TUYA_PWM0);
    /*pwm init*/
    TUYA_PWM_BASE_CFG_T pwm_cfg = {
        .duty       = PWM_DUTY,             /* 1-10000 */
        .frequency  = PWM_FREQUENCY,        /* 1kHz */
        .polarity   = TUYA_PWM_NEGATIVE,    /* Polarity of the PWM signal */
    };
    TUYA_CALL_ERR_LOG(tkl_pwm_init(PWM_ID, &pwm_cfg));
    /*start PWM*/
    TUYA_CALL_ERR_LOG(tkl_pwm_start(PWM_ID));
}

/**
 * @brief       Set PWM duty cycle
 * @param[in]   duty: Duty cycle value (0-10000, where 5000 is 50%)
 * @return      none
 */
void pwm_set_duty(uint32_t duty)
{
    OPERATE_RET rt = OPRT_OK;
    TUYA_CALL_ERR_LOG(tkl_pwm_duty_set(PWM_ID, duty));  /* Set the duty cycle */
    TUYA_CALL_ERR_LOG(tkl_pwm_start(PWM_ID));           /* Restart PWM to apply the new duty cycle */
}
```

上述源码中，pwm_init函数用来配置P18为PWM复用管脚和初始化PWM，然后pwm_set_duty函数用来更新占用比。

### 2，CMakeLists.txt文件

CMakeLists.txt文件配置内容如下。

```
# Add Driver
set(src_dirs
    LED		# 添加驱动组件
    PWM
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
#include "pwm.h"

/**
 * @brief       user_main
 *
 * @param[in]   none
 * @return      none
 */
void user_main(void)
{
    uint8_t dir = 1;        /* Direction flag: 1 for increasing, 0 for decreasing */
    uint16_t ledpwmval = 0; /* Initial PWM value */
    tal_log_init(TAL_LOG_LEVEL_DEBUG, 1024, (TAL_LOG_OUTPUT_CB)tkl_log_output); /* Initialize log output */
    pwm_init();             /* Initialize PWM */

    while (1) {
        tal_system_sleep(100);          /* Sleep for 100 ms */
        if (dir == 1)
        {
            ledpwmval += 5 * 100;       /* Increase PWM value by 5 */
        }
        else
        {
            ledpwmval -= 5 * 100;       /* Decrease PWM value by 5 */
        }

        if (ledpwmval > 9500)     
        {
            dir = 0;                    /* Change direction to decreasing */
        }

        if (ledpwmval < 5)
        {
            dir = 1;                    /* Change direction to increasing */
        }

        pwm_set_duty(ledpwmval);        /* Set the PWM duty cycle */
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

从user_main函数可以看到，每100ms周期配置输出LED灯的PWM占空比，从而实现呼吸灯的效果。

## 运行验证

程序下载完成后，可看到LED灯从亮到灭，后从灭到亮的转变。
