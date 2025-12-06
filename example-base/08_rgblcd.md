---
title: 'RGBLCD实验'
sidebar_position: 1
---

# RGBLCD实验

## 前言

本章实验将介绍如何使用TuyaOpen让T5驱动RGBLCD显示屏，实现屏幕刷新。

上层代码跟07_spilcd是一样的，只是底层的代码需要进行改动。

## 操作流程

使用display组件，需要打开显示驱动的使能宏，操作如下：

```
tos.py config menu
```

然后到configure device driver中使能display。

![](./img/enable_display.png)

还需要选择对应的屏幕类型，7寸RGBLCD。

![](./img/select_rgblcd.png)

记得最后要保存。

方便快捷直接使用 `tos.py config choice`命令选择生成好的配置文件。

## RGBLCD简介

这里，笔者使用正点原子的ATK-MD0700R-800480为例--参考文档**ATK-MD0700R模块用户手册_V1.1.pdf**。

### RGBLCD的信号线
 RGBLCD的信号线如下表：

| 裸屏信号线     | 信号说明                     |
| ------------- | ---------------------------  |
| LCD_CLK       | 像素同步时钟信号线            |
| LCD_HSYNC     | 水平同步信号线                |
| LCD_VSYNC     | 垂直同步信号线                |
| LCD_DE        | 数据使能线                    |
| LCD_R[7:0]    | 红色数据线，一般为8位         |
| LCD_G[7:0]    | 绿色数据线，一般为8位         |
| LCD_B[7:0]    | 蓝色数据线，一般为8位         |

一般RGB屏都有上表信号线，有24根颜色数据线（RGB各8根，即RGB888格式），这样可以表示最多1600W色，DE、VS、HS和CLK，用于控制数据传输。

像素同步时钟信号线LCD_CLK：液晶屏与外部使用同步通讯方式，以CLK信号作为同步时钟，在同步时钟的驱动下，每个时钟传输一个像素点数据。

水平同步信号线LCD_HSYNC：有时也被称为行同步信号，用于表示液晶屏一行像素数据的传输结束，每传输完成液晶屏的一行像素数据时，LCD_HSYNC发生电平跳变，如分辨率为800x480的显示屏（800列，480行），传输一帧的图像LCD_HSYNC的电平会跳变480次。

垂直同步信号线LCD_VSYNC：有时也被称为场同步信号，用于表示液晶屏一帧像素数据的传输结束，每传输完成一帧像素时，LCD_VSYNC会发生电平跳变。其中“帧”是图像的单位，一幅图像称为一帧，在液晶屏中，一帧指一个完整屏液晶像素点。

数据使能信号线DE：用于表示数据的有效性，当DE信号线为高电平时，RGB信号线表示的数据有效。

RGB数据线：用来传输颜色数据。

### RGBLCD的驱动模式

RGB屏一般有2种驱动模式：DE模式和HV模式。DE模式使用DE信号来确定有效数据（DE为高/低时，数据有效），而HV模式，则需要行同步信号HSYNC和场同步信号VSYNC，来表示扫描的行和列。

DE模式和HV模式的行扫描时序图（以800*480的LCD面板为例），如下图所示：

![](.\img\rgblcd_dehv_mode.png)

从图中可以看出，DE和HV模式，时序基本一样，DE模式需要提供DE信号（DEN），而HV模式，则无需DE信号。

图中的HSD即HSYNC信号，用于行同步，注意：在DE模式下面，是可以不用HSYNC信号和VSYNC信号，即可以不接，液晶照样可以正常工作。在引脚不是太充足的情况下，可以选择使用DE模式。

图中的thpw为水平同步有效信号脉宽，用于表示一行数据的开始；thb为水平后廊，表示从水平有效信号开始，到有效数据输出之间的像素时钟个数；thfp为水平前廊，表示一行数据结束后，到下一个水平同步信号开始之前的像素时钟个数。

上图仅是一行数据的扫描，输出800个像素点数据，而液晶面板总共有480行，这就还需要一个垂直扫描时序图，如下图所示：

![](.\img\rgblcd_v_mode.png)

图中的VSD就是垂直同步信号LCD_VSYNC，HSD就是水平同步信号LCD_HSYNC，DE为数据使能信号。

如图可知，一个垂直扫描，刚好就是480个有效的DE脉冲信号，每一个DE时钟周期，扫描一行，总共扫描480行，完成一帧数据的显示。这就是800*480的LCD面板扫描时序，其他分辨率的LCD面板，时序类似。

图中的tvpw为垂直同步有效信号脉宽，用于表示一帧数据的开始；tvb为垂直后廊，表示垂直同步信号以后的无效行数，tvfp为垂直前廊，表示一帧数据输出结束后，到下一个垂直同步信号开始之前的无效行数；这几个时间在配置RGBLCD设备时序时，需要进行设置。

### 正点原子7寸800480 RGBLCD屏幕信息

![](.\img\rgblcd_photo.png)

| 项目          | 说明                            |
| -------------| ---------------------------     |
| 通信接口      | LCD：并行24位RGB接口；触摸：IIC   |
| 颜色格式      | RGB888(兼容RGB565)               |
| 颜色深度      | 最大24位                          |
| LCD分辨率     | 800 * 480                         |
| 屏幕尺寸      | 7寸                               |
| 触摸屏类型    | 电容触摸                           |
| 触摸点数      | 最多5点同时触摸                    |
| 工作温度      | -20 ~ 70 ℃                       |
| 存储温度      | -30 ~ 80 ℃                       |
| 模块尺寸      | 92mm * 180mm                      |

#### 模块接口

![](.\img\rgblcd_interface.png)

图中J1就是对外接口，是一个 40PIN 的 FPC 座（ 0.5mm 间距），通过 FPC 线，可以连接到T5小系统板的RGBLCD接口上面，从而实现和T5的连接。该模块的接口十分完善，采用 RGB888 格式，并支持 DE&HV 模式，还支持触摸屏（电阻/电容）和背光控制。

## API 描述

API函数部分只有最底层的注册函数跟07_spilcd不一样，其余是一样的，所以这里只说明tdd_disp_rgb_device_register函数。
 
**1，tdd_disp_rgb_device_register函数**

​	通过RGB接口注册TFT显示设备到显示管理系统。

```C
OPERATE_RET tdd_disp_rgb_device_register(char *name, TDD_DISP_RGB_CFG_T *rgb);
```

​	**1.1 参数描述**

|  形参   |             描述             |
| :-----: | :--------------------------: |
|  name   |         屏幕设备名称         |
| rgb | 指向RGB接口的设备配置结构的指针 |

​	`name`：屏幕设备名称：

​	`rgb`：屏幕配置结构。

```C
/* TuyaOpen/src/peripherals/display/tdd_display/include/tdd_display_rgb.h */
typedef struct {
    TUYA_RGB_BASE_CFG_T         cfg;
    TUYA_DISPLAY_BL_CTRL_T      bl;
    TUYA_DISPLAY_IO_CTRL_T      power;
    TDD_DISPLAY_SEQ_INIT_CB     init_cb; 
    TUYA_DISPLAY_ROTATION_E     rotation;
    bool                        is_swap; 
}TDD_DISP_RGB_CFG_T;

typedef struct {
    UINT32_T clk;
    TUYA_RGB_DATA_CLK_EDGE_E out_data_clk_edge;  /** rgb data output in clk rising or falling */
    UINT16_T width;
    UINT16_T height;
	TUYA_DISPLAY_PIXEL_FMT_E pixel_fmt;
    UINT16_T hsync_back_porch;            /**< rang 0~0x3FF (0~1023), should refer rgb device spec*/
	UINT16_T hsync_front_porch;           /**< rang 0~0x3FF (0~1023), should refer rgb device spec*/
	UINT16_T vsync_back_porch;            /**< rang 0~0xFF (0~255), should refer rgb device spec*/
	UINT16_T vsync_front_porch;           /**< rang 0~0xFF (0~255), should refer rgb device spec*/
	UINT8_T hsync_pulse_width;            /**< rang 0~0x3F (0~7), should refer rgbdevice spec*/
	UINT8_T vsync_pulse_width;            /**< rang 0~0x3F (0~7), should refer rgb device spec*/
} TUYA_RGB_BASE_CFG_T;

typedef struct {
    TUYA_DISPLAY_BL_TYPE_E    type;
    union {
        TUYA_DISPLAY_IO_CTRL_T   gpio;
        TUYA_DISPLAY_PWM_CTRL_T  pwm;
    };
} TUYA_DISPLAY_BL_CTRL_T;

typedef struct {
    TUYA_GPIO_NUM_E   pin;
    TUYA_GPIO_LEVEL_E active_level;
} TUYA_DISPLAY_IO_CTRL_T;

typedef OPERATE_RET (*TDD_DISPLAY_SEQ_INIT_CB)(void);

typedef enum{
    TUYA_DISPLAY_ROTATION_0,
    TUYA_DISPLAY_ROTATION_90,
    TUYA_DISPLAY_ROTATION_180,
    TUYA_DISPLAY_ROTATION_270,
}TUYA_DISPLAY_ROTATION_E;
```

​	**1.2 返回值**

​	OPRT_OK表示成功。关于其他错误，请参考`tuya_error_code.h`。

## 硬件设计

### 例程功能

1，一秒周期内切换颜色刷新屏幕。

### 硬件资源

1，RGBLCD

​	LCD_R3: P23

​	LCD_R4: P22

​	LCD_R5: P21

​	LCD_R6: P20

​	LCD_R7: P19

​	LCD_G2: P42

​	LCD_G3: P41

​	LCD_G4: P40

​	LCD_G5: P26

​	LCD_G6: P25

​	LCD_G7: P24

​	LCD_B3: P47

​	LCD_B4: P46

​	LCD_B5: P45

​	LCD_B6: P44

​	LCD_B7: P43

​	LCD_PCLK: P14

​	LCD_DE: P16

​	LCD_BL: P9

​	LCD_RST: P27

### 原理图

正点原子T5 AI开发板上RGBLCD的连接原理图，如下图所示。

![](.\img\rgblcd_sch.png)

接线方面需要注意，如下图所示。

![](.\img\rgblcd_t5.png)

RGBLCD屏幕的FPC排线金属接口侧需要朝T5小系统板的TypeC方向接入竖立的FPC座子，并需要按下锁住固定。

## 程序设计

### RGBLCD底层初始化函数

```c    
static TDD_DISP_RGB_CFG_T sg_disp_rgb = {
    .cfg =
        {
            .clk = 32000000,
            .out_data_clk_edge = TUYA_RGB_DATA_IN_FALLING_EDGE,
            .pixel_fmt = TUYA_PIXEL_FMT_RGB565,
            .hsync_back_porch = 46,
            .hsync_front_porch = 210,
            .vsync_back_porch = 23,
            .vsync_front_porch = 22,
            .hsync_pulse_width = 2,
            .vsync_pulse_width = 2,
        },
};

static TUYA_DISPLAY_IO_CTRL_T sg_lcd_rst;

static OPERATE_RET __atk_t5ai_disp_rgb_md0700r_init(void)
{
    OPERATE_RET rt = OPRT_OK;
    TUYA_GPIO_BASE_CFG_T gpio_cfg;

    gpio_cfg.mode = TUYA_GPIO_PUSH_PULL;
    gpio_cfg.direct = TUYA_GPIO_OUTPUT;
    gpio_cfg.level = PIN_TRIG_LV(sg_lcd_rst.active_level);
    tkl_gpio_init(sg_lcd_rst.pin, &gpio_cfg);

    tal_system_sleep(20);

    tkl_gpio_write(sg_lcd_rst.pin, sg_lcd_rst.active_level);
    tal_system_sleep(200);

    tkl_gpio_write(sg_lcd_rst.pin, PIN_TRIG_LV(sg_lcd_rst.active_level));
    tal_system_sleep(120);

    return rt;
}

OPERATE_RET atk_t5ai_disp_rgb_md0700r_register(char *name, ATK_T5AI_DISP_MD0700R_CFG_T *dev_cfg)
{
    if (NULL == name || NULL == dev_cfg) {
        return OPRT_INVALID_PARM;
    }

    sg_disp_rgb.init_cb = __atk_t5ai_disp_rgb_md0700r_init;

    sg_disp_rgb.cfg.width     = dev_cfg->width;
    sg_disp_rgb.cfg.height    = dev_cfg->height;
    sg_disp_rgb.cfg.pixel_fmt = TUYA_PIXEL_FMT_RGB565;
    sg_disp_rgb.rotation      = dev_cfg->rotation;
    sg_disp_rgb.is_swap       = false;

    memcpy(&sg_disp_rgb.power, &dev_cfg->power, sizeof(TUYA_DISPLAY_IO_CTRL_T));
    memcpy(&sg_disp_rgb.bl, &dev_cfg->bl, sizeof(TUYA_DISPLAY_BL_CTRL_T));
    memcpy(&sg_lcd_rst, &dev_cfg->rst, sizeof(TUYA_DISPLAY_IO_CTRL_T));

    return tdd_disp_rgb_device_register(name, &sg_disp_rgb);
}
```

**其余源码跟07_spilcd例程内容是一样的，这里就不再赘述了。**

## 运行验证

程序下载完成后，1秒周期内屏幕刷新不同颜色。
