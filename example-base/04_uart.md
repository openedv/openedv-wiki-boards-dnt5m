---
title: 'UART实验'
sidebar_position: 1
---

# UART实验

## 前言

本章实验将介绍如何使用TuyaOpen让T5实现UART通信，从而实现LED控制。

## UART模块介绍

### 概述

UART(Universal Asynchronous Receiver/Transmitter) 是一种通用串行数据总线，用于异步通信。该总线双向通信，可以实现全双工传输和接收。

## API 描述

**1，tkl_uart_init函数**

​	UART初始化函数。

```C
OPERATE_RET tkl_uart_init(TUYA_UART_NUM_E port_id, TAL_UART_CFG_T *cfg);
```

​	**1.1 参数描述**

|  形参   |    描述    |
| :-----: | :--------: |
| port_id | 串口端口号 |
|   cfg   |  串口配置  |

​	`port_id`：端口号：

```C
/* TuyaOpen/tools/porting/adapter/utilities/include/tuya_cloud_types.h */
typedef enum {
    TUYA_UART_NUM_0,            // UART 0
    TUYA_UART_NUM_1,            // UART 1
    TUYA_UART_NUM_2,            // UART 2
    TUYA_UART_NUM_3,            // UART 3
    TUYA_UART_NUM_4,            // UART 4
    TUYA_UART_NUM_5,            // UART 5
    TUYA_UART_NUM_MAX,
} TUYA_UART_NUM_E;
```

​	`cfg`：串口基础配置，包含波特率，奇偶校验位，停止位，流控制。

```C
/* TuyaOpen/src/tal_driver/include/tal_uart.h */
typedef struct {
    uint32_t rx_buffer_size;
#ifdef CONFIG_TX_ASYNC
    uint32_t tx_buffer_size;
#endif
    uint8_t open_mode;
    TUYA_UART_BASE_CFG_T base_cfg;
} TAL_UART_CFG_T;

/* TuyaOpen/platform/T5AI/tuyaos/tuyaos_adapter/include/utilities/include/tuya_cloud_types.hh */
/**
 * @brief uart config
 *
 */
typedef struct {
    UINT_T                      baudrate;   /* 波特率 */
    TUYA_UART_PARITY_TYPE_E     parity;     /* 奇偶校验位 */
    TUYA_UART_DATA_LEN_E        databits;   /* 数据位 */
    TUYA_UART_STOP_LEN_E        stopbits;   /* 停止位 */
    TUYA_UART_FLOWCTRL_TYPE_E   flowctrl;   /* 流控制 */
} TUYA_UART_BASE_CFG_T;

/**
 * @brief uart parity
 *
 */
typedef enum {
    TUYA_UART_PARITY_TYPE_NONE    = 0,      /* 无奇偶校验位 */
    TUYA_UART_PARITY_TYPE_ODD     = 1,      /* 奇校验位 */
    TUYA_UART_PARITY_TYPE_EVEN    = 2,      /* 偶校验位 */
} TUYA_UART_PARITY_TYPE_E;

/**
 * @brief uart databits
 *
 */
typedef enum {
    TUYA_UART_DATA_LEN_5BIT      = 0x05,    /* 5位数据位 */
    TUYA_UART_DATA_LEN_6BIT      = 0x06,    /* 6位数据位 */
    TUYA_UART_DATA_LEN_7BIT      = 0x07,    /* 7位数据位 */
    TUYA_UART_DATA_LEN_8BIT      = 0x08,    /* 8位数据位 */
} TUYA_UART_DATA_LEN_E;

/**
 * @brief uart stop bits
 *
 */
typedef enum {
    TUYA_UART_STOP_LEN_1BIT      = 0x01,    /* 1位停止位 */
    TUYA_UART_STOP_LEN_1_5BIT1   = 0x02,    /* 1.5位停止位 */
    TUYA_UART_STOP_LEN_2BIT      = 0x03,    /* 2位停止位 */
} TUYA_UART_STOP_LEN_E;

typedef enum {
    TUYA_UART_FLOWCTRL_NONE = 0,            /* 无流控制方案 */
    TUYA_UART_FLOWCTRL_RTSCTS,              /* 请求/清除发送 */
    TUYA_UART_FLOWCTRL_XONXOFF,             /* 暂停传输/回复传输 */
    TUYA_UART_FLOWCTRL_DTRDSR,              /* 数据终端准备好/数据准备好 */
} TUYA_UART_FLOWCTRL_TYPE_E;
```

​	**1.2 返回值**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

**2，tal_uart_read函数**

​	读取串口数据。

```C
int tal_uart_read(TUYA_UART_NUM_E port_id, uint8_t *data, uint32_t len);
```

​	**2.1 参数描述**

​	`port_id`：串口端口号

​	`data`：读取数据缓存

​	`len`：读取数据大小

​	**2.2 返回值**

​	 \>=0：数据大小；< 0：读取错误。

**3，tal_uart_write函数**

​	向串口写数据。

```C
int tal_uart_write(TUYA_UART_NUM_E port_id, const uint8_t *data, uint32_t len);
```

​	**3.1 参数描述**

​	`port_id`：串口端口号

​	`data`：发送数据缓存

​	`len`：发送数据大小

​	**3.2 返回值**

​	 \>=0：数据大小；< 0：读取错误。

**本实验涉及了上述UART函数，其他相关函数将在后续章节中结合具体应用进行详细介绍**。

## 硬件设计

### 例程功能

1，若串口调试助手发送“LED_ON”字符串，会打开板载的LED灯，若发送“LED_OFF”，则会关闭LED灯。

### 硬件资源

1，LED

​	LED - P18

2，UART

​	TXD - P0

​	RXD - P1

### 原理图

正点原子T5 AI开发板上UART的连接原理图，如下图所示。

![](.\img\04.png)

从上图可以发现，T5AI开发板的下载口与日志打印口共用一个USB转串口芯片，需要我们在P2端口进行选择，为了方便下载与调试，用户可购买USB转串口模块，一路用于下载，另一个用于日志查看。

> [!IMPORTANT]
>
> 建议板载USB转串口用于下载，而日志打印口可使用外接USB转串口。



## 程序设计

### 1，UART驱动代码

这里我们只讲解核心代码，详细的源码请大家参考光盘资料本实验对应的源码。UART驱动源码包括两个文件：uart.c和uart.h

uart.h文件是对UART引脚做了相关定义以及函数声明。

```C
/* Define the UART port number
   This should match the actual UART port used for the hardware. */
#define UART_PORT_NUM      TUYA_UART_NUM_0

/* Define the UART RX buffer size
   This size should be sufficient for the expected data throughput. */
#define RX_BUF_SIZE        256

/* Function Declaration */
void uart_init(uint32_t baudrate);    /* Initialize the uart */
```

uart.c文件是对串口0初始化。

```C
/**
 * @brief       UART initialization
 * @param[in]   baudrate: Baud rate for UART communication
 * @return      none
 */
void uart_init(uint32_t baudrate)
{
    OPERATE_RET rt = OPRT_OK;

    TAL_UART_CFG_T cfg = {0};
    cfg.base_cfg.baudrate   = baudrate;                     /* Set the baud rate */
    cfg.base_cfg.databits   = TUYA_UART_DATA_LEN_8BIT;      /* Set data bits to 8 */
    cfg.base_cfg.stopbits   = TUYA_UART_STOP_LEN_1BIT;      /* Set stop bits to 1 */
    cfg.base_cfg.parity     = TUYA_UART_PARITY_TYPE_NONE;   /* No parity */
    cfg.rx_buffer_size      = RX_BUF_SIZE;                  /* Set RX buffer size */
    cfg.open_mode           = O_BLOCK;                      /* Set open mode to blocking */
    TUYA_CALL_ERR_LOG(tal_uart_init(UART_PORT_NUM, &cfg));  /* Initialize UART with the configuration */
}

```

上述源码可以看到，首先我们配置串口波特率、数据位、停止位、奇偶校验位...，然后调用tal_uart_init函数对串口0进行初始化。

### 2，CMakeLists.txt文件

CMakeLists.txt文件配置内容如下。

```
# Add Driver
set(src_dirs
    LED		# 添加驱动组件
    UART
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
#include "uart.h"

/**
 * @brief       user_main
 *
 * @param[in]   none
 * @return      none
 */
void user_main(void)
{
    uint16_t len = 0;
    char data[10] = {0};
    char *a = "LED_ON";
    char *b = "LED_OFF";
    tal_log_init(TAL_LOG_LEVEL_DEBUG, 1024, (TAL_LOG_OUTPUT_CB)tkl_log_output); /* Initialize log output */
    led_init();         /* Initialize LED */
    uart_init(115200);  /* Initialize UART with baud rate 115200 */

    while (1) {

        len = tal_uart_read(UART_PORT_NUM, (uint8_t *)data, 10);

        if (len > 0)
        {
            data[len] = '\0';

            if (strcmp(a, data) == 0)
            {
                led_set_state(0);
            }
            else if (strcmp(b, data) == 0)
            {
                led_set_state(1);
            }

            tal_uart_write(UART_PORT_NUM, (const uint8_t*)data, len);

            memset(data, 0, 10);
        }

        tal_system_sleep(10);
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

从上述源码可以看出，若接收到"LED_ON"字符串，会打开板载LED灯，若接收到”LED_OFF“，则会关闭LED灯。

## 运行验证

程序下载完成后，串口调试助手可根据发送的字符串来控制LED灯亮灭。
