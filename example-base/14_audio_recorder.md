---
title: '录音机实验'
sidebar_position: 1
---

# 录音机实验

## 前言

本章实验将介绍如何使用TuyaOpen让T5录制声音，并播放录制声音。

# 音频驱动介绍

### 概述

[音频驱动](https://github.com/tuya/TuyaOpen/tree/master/src/peripherals/audio_codecs) 是 TuyaOpen 中用于处理音频输入和输出的核心组件。它提供了统一的接口来管理不同类型的音频设备，如麦克风和扬声器。通过这个驱动，应用可以轻松地进行音频采集、播放和配置，而无需关心底层硬件的具体实现细节。

> [!NOTE]
>
> 根据主控芯片的不同，音频连接框架也会有所不同。如 `T5AI` 自带 ADC 和 DAC 接口，不需要 CODEC 芯片也可以实现音频系统。而 `ESP32-S3` 不支持 DAC，需要外挂 CODEC 芯片完成音频系统的搭建。

TuyaOpen框架面向应用层提供统一的音频服务接口： `Tuya Driver Layer` 。

`tdl_audio_manage.c/h`:实现了音频驱动的管理核心。它维护一个链表，用于注册和管理不同类型的音频设备驱动。应用通过调用`tdl_audio_find`、`tdl_audio_open`等函数来使用音频功能，而无需关心底层的具体实现。

`tdl_audio_driver.h`:定义了所有音频设备驱动必须遵守的“标准化接口”（`TDD_AUDIO_INTFS_T`）,包括`open`、 `play`、`config`、`close`等函数指针。

音频接口的中间层是进行特定硬件平台的具体实现。

`tdd_audio.c/h`：针对不同平台的音频驱动实现。它负责承上启下，向上实现了`TDL`所定义的`TDD_AUDIO_INTFS_T`标准接口，向下调用`TKL`层或者原厂提供的硬件抽象接口来控制真实的硬件。`tdd_audio_register`函数会将这个驱动的实现（函数指针）注册到 `TDL`层。

## API 描述

**1，tdd_audio_register函数**

音频设备注册接口。

```C
OPERATE_RET tdd_audio_register(const char *name, TDD_AUDIO_T5AI_T cfg);
```

**1.1 参数描述**

`name`：音频设备名字

`cfg`：音频设备配置参数

```C
typedef struct {
    uint8_t aec_enable;                 /* 是否使能 */
    TKL_AI_CHN_E ai_chn;                /* 音频输入通道 */          	
    TKL_AUDIO_SAMPLE_E sample_rate;     /* 采样率 */
    TKL_AUDIO_DATABITS_E date_bits;    	/* 数据位 */
    TKL_AUDIO_CHANNEL_E channel;      	/* 通道数量 */

    TKL_AUDIO_SAMPLE_E spk_sample_rate; /* SPK采样率 */  
    int spk_pin;                        /* SPK放大器引脚数，<0，无放大器 */
    int spk_pin_polarity;               /* 引脚极性，0高使能，1低使能 */
} TDD_AUDIO_T5AI_T;
```

**1.1 返回值**

OPRT_OK表示成功，其他表示失败。

**2，tdl_audio_find函数**

设备查找与管理接口。

```C
TDL_AUDIO_HANDLE_T tdl_audio_find(const char *name);
```

**2.1 参数描述**

`name`：音频设备名字

**2.1 返回值**

返回音频设备句柄。

**3，tdl_audio_open函数**

用于打开并初始化音频设备，包括硬件初始化、引脚配置等操作，使设备进入可用状态。

```C
OPERATE_RET tdl_audio_open(TDL_AUDIO_HANDLE_T handle, TDL_AUDIO_MIC_CB mic_cb);
```

**3.1 参数描述**

`handle`：音频设备句柄

`mic_cb`：MIC采集回调函数

**3.1 返回值**

OPRT_OK表示成功，其他表示失败。

**4，tdl_audio_volume_set函数**

用于动态调节音频输出音量，控制扬声器功放增益。

```C
OPERATE_RET tdl_audio_volume_set(TDL_AUDIO_HANDLE_T handle, uint8_t volume);
```

**4.1 参数描述**

`handle`：音频设备句柄

`volume`：音量

**4.2 返回值**

OPRT_OK表示成功，其他表示失败。

**5，tdl_audio_play函数**

用于播放音频数据，将音频帧通过硬件接口输出到扬声器。

```C
OPERATE_RET tdl_audio_play(TDL_AUDIO_HANDLE_T handle, uint8_t *data, uint32_t len);
```

**5.1 参数描述**

`handle`：音频设备句柄

`data`：音频数据缓冲区

`len`：数据长度

**5.2 返回值：**

OPRT_OK表示成功，其他表示失败。

## 硬件设计

### 例程功能

1，测试T5自带 ADC 和 DAC 接口。长按KEY按键可持续录制当前的声音，释放KEY按键时，可播放录制的声音。最后还会保存文件到TF卡。

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

2，SPILCD

​	DC/WR: P42

​	RST: P43

​	CS: P45

​	SCK: P44

​	SDA: P46

​	BL/PWR: P9

3，SD卡

​	SD_SCK- P2

​	SD_CMD- P3

​	SD_D0- P4

​	SD_D1- P5

​	SD_D2- P36

​	SD_D3- P37

### 原理图

正点原子T5 AI开发板上MIC和SPK的连接原理图，如下图所示。

![](.\img\10.png)

## 程序设计

### 1，录音驱动代码

这里我们只讲解核心代码，详细的源码请大家参考光盘资料本实验对应的源码。录音源码包括4个文件：wav_encode.c、wav_encode.h、mic_record.c和mic_record.h。他们位于components\Middlewares\MIC_RECORD文件夹下。

mic_record.h文件是对音频相关函数声明。

```C
/* Function Declaration */
void mic_recorder();
```

mic_record.c文件是音频文件生成以及播放相关函数。

```C
#define RECORD_DURATION_MS (3 * 1000) /* Record duration in milliseconds */

typedef enum {
    RECORDER_STATUS_IDLE = 0,
    RECORDER_STATUS_START,
    RECORDER_STATUS_RECORDING,
    RECORDER_STATUS_END,
    RECORDER_STATUS_PLAYING,
} RECORDER_STATUS_E;

static RECORDER_STATUS_E sg_recorder_status = RECORDER_STATUS_IDLE;
static TDL_AUDIO_HANDLE_T sg_audio_hdl = NULL;
static TDL_AUDIO_INFO_T sg_audio_info = {0};
static TUYA_RINGBUFF_T sg_recorder_pcm_rb = NULL;

#if defined(ENABLE_BUTTON) && (ENABLE_BUTTON == 1)
/**
 * @brief       Button event callback function, controls recorder status based on button events
 * @param[in]   name    Button name string
 * @param[in]   event   Button touch event type (e.g., press down, release)
 * @param[in]   argc    Extra parameter (unused)
 * @note
 *      - On button press down (TDL_BUTTON_PRESS_DOWN), if the recorder is idle, start recording; otherwise, prompt to wait for idle status
 *      - On button release (TDL_BUTTON_PRESS_UP), if the recorder is recording, stop recording
 */
static void __button_function_cb(char *name, TDL_BUTTON_TOUCH_EVENT_E event, void *argc)
{
    argc = argc;

    switch (event) {
    case TDL_BUTTON_PRESS_DOWN: {
        PR_NOTICE("%s: single click", name);
        tftlcd_fill_rect(30, 210, 200, 16, WHITE);  /* Clear previous status */
        tftlcd_show_string(30, 210, 200, 16, 16, "Recording...", BLUE);
        if (sg_recorder_status == RECORDER_STATUS_IDLE) {
            sg_recorder_status = RECORDER_STATUS_START;
        } else {
            PR_WARN("Please wait status IDLE");
        }
    } break;

    case TDL_BUTTON_PRESS_UP: {
        PR_NOTICE("%s: release", name);
        /* Display recording done on LCD */
        tftlcd_fill_rect(30, 210, 200, 16, WHITE);  /* Clear previous status */
        tftlcd_show_string(30, 210, 200, 16, 16, "Playing...", GREEN);
        if (sg_recorder_status == RECORDER_STATUS_RECORDING) {
            sg_recorder_status = RECORDER_STATUS_END;
        }
    } break;

    default:
        break;
    }
}
#endif

/**
 * @brief       Get the next available file index by scanning RECORD directory
 * @param       None
 * @return      uint32_t Next available file index
 * @note        Scans /sdcard/RECORD/ for existing record_XXX.wav files
 *              and returns the next available index number
 */
static uint32_t __get_next_file_index(void)
{
    uint32_t max_index = 0;
    TUYA_DIR dir = NULL;
    TUYA_FILEINFO file_info;
    const char *filename = NULL;
    BOOL_T is_dir = FALSE;

    /* Open RECORD directory */
    if (tkl_dir_open("/sdcard/RECORD", &dir) != OPRT_OK) {
        PR_WARN("Failed to open RECORD directory, starting from index 0");
        return 0;
    }

    /* Scan all files in directory */
    while (tkl_dir_read(dir, &file_info) == OPRT_OK) {
        /* Get filename */
        if (tkl_dir_name(file_info, &filename) != OPRT_OK || filename == NULL) {
            continue;
        }

        /* Check if it's a file (not directory) */
        if (tkl_dir_is_directory(file_info, &is_dir) != OPRT_OK || is_dir == TRUE) {
            continue;
        }

        /* Check if filename matches pattern: record_XXX.wav */
        if (strncmp(filename, "record_", 7) == 0 && strstr(filename, ".wav") != NULL) {
            /* Extract index number */
            uint32_t index = 0;
            if (sscanf(filename, "record_%u.wav", &index) == 1) {
                if (index >= max_index) {
                    max_index = index + 1;
                }
            }
        }
    }

    /* Close directory */
    tkl_dir_close(dir);

    PR_NOTICE("Next file index: %u", max_index);
    return max_index;
}

/**
 * @brief       Save recorded audio to WAV file
 * @param       None
 * @return      OPERATE_RET OPRT_OK on success, error code otherwise
 * @note
 *      - Reads all PCM data from the recorder ring buffer
 *      - Saves the data as a WAV file to SD card
 *      - File naming: record_XXX.wav (incremental numbering)
 *      - Displays save status on LCD
 */
static OPERATE_RET __save_audio_to_wav_file(void)
{
    OPERATE_RET rt = OPRT_OK;
    uint32_t data_len = 0;
    uint8_t *pcm_buffer = NULL;
    char filename[64] = {0};
    uint32_t file_index = 0;

    if (NULL == sg_recorder_pcm_rb || 0 == sg_audio_info.frame_size) {
        PR_ERR("Invalid recorder state");
        return OPRT_INVALID_PARM;
    }

    /* Get total data length in ring buffer */
    data_len = tuya_ring_buff_used_size_get(sg_recorder_pcm_rb);
    if (data_len == 0) {
        PR_WARN("No data to save");
        tftlcd_fill_rect(30, 210, 200, 16, WHITE);
        tftlcd_show_string(30, 210, 200, 16, 16, "No data!", RED);
        tal_system_sleep(1000);
        return OPRT_OK;
    }

    PR_NOTICE("Saving %d bytes of audio data", data_len);

    /* Display saving status */
    tftlcd_fill_rect(30, 210, 200, 16, WHITE);
    tftlcd_show_string(30, 210, 200, 16, 16, "Saving...", BLUE);

    /* Allocate buffer for PCM data */
    pcm_buffer = tkl_system_psram_malloc(data_len);
    if (NULL == pcm_buffer) {
        PR_ERR("Failed to allocate memory for PCM buffer");
        tftlcd_fill_rect(30, 210, 200, 16, WHITE);
        tftlcd_show_string(30, 210, 200, 16, 16, "Save Failed!", RED);
        tal_system_sleep(1000);
        return OPRT_MALLOC_FAILED;
    }

    /* Read all data from ring buffer */
    tuya_ring_buff_read(sg_recorder_pcm_rb, pcm_buffer, data_len);

    /* Create RECORD directory if it doesn't exist */
    BOOL_T dir_exist = FALSE;
    rt = tkl_fs_is_exist("/sdcard/RECORD", &dir_exist);

    /* If directory doesn't exist or check failed, try to create it */
    if (rt != OPRT_OK || dir_exist == FALSE) {
        rt = tkl_fs_mkdir("/sdcard/RECORD");
        if (rt != OPRT_OK) {
            PR_WARN("Failed to create RECORD directory (may already exist), error: %d", rt);
            /* Continue anyway, directory might already exist */
        } else {
            PR_NOTICE("Created RECORD directory");
        }
    }

    /* Get next available file index by scanning directory */
    file_index = __get_next_file_index();

    /* Generate filename with incremental index */
    snprintf(filename, sizeof(filename), "/sdcard/RECORD/record_%03d.wav", file_index);

    /* Save to WAV file */
    rt = wav_encode_save_file(filename, pcm_buffer, data_len,
                              16000,  /* Sample rate: 16kHz */
                              16,     /* Bit depth: 16 bits */
                              1);     /* Channel: Mono */

    /* Write data back to ring buffer for playback */
    if (OPRT_OK == rt) {
        tuya_ring_buff_write(sg_recorder_pcm_rb, pcm_buffer, data_len);
    }

    /* Free buffer */
    tkl_system_psram_free(pcm_buffer);
    pcm_buffer = NULL;

    /* Display result */
    tftlcd_fill_rect(30, 210, 200, 16, WHITE);
    if (OPRT_OK == rt) {
        PR_NOTICE("Audio saved successfully: %s", filename);
        tftlcd_show_string(30, 210, 200, 16, 16, "Saved!", GREEN);

        /* Display filename on LCD */
        tftlcd_fill_rect(30, 230, 200, 16, WHITE);
        char display_name[32] = {0};
        snprintf(display_name, sizeof(display_name), "File: record_%03d.wav", file_index);
        tftlcd_show_string(30, 230, 200, 16, 16, display_name, BLUE);
    } else {
        PR_ERR("Failed to save audio file, error: %d", rt);
        tftlcd_show_string(30, 210, 200, 16, 16, "Save Failed!", RED);
    }

    tal_system_sleep(1500);

    return rt;
}

/**
 * @brief       Play audio data from the recorder ring buffer
 * @param       None
 * @return      None
 * @note
 *      - Checks if the recorder ring buffer, audio handle, and frame size are valid
 *      - Allocates a frame buffer for audio playback
 *      - Reads audio data from the recorder ring buffer and plays it frame by frame
 *      - If there is no data, playback is skipped
 *      - Frees the allocated frame buffer after playback
 */
static void play_from_recorder_rb(void)
{
    if (NULL == sg_recorder_pcm_rb || NULL == sg_audio_hdl ||\
       0 == sg_audio_info.frame_size) {
        return;
    }

    uint32_t data_len = tuya_ring_buff_used_size_get(sg_recorder_pcm_rb);
    if (data_len == 0) {
        PR_NOTICE("No data in recorder ring buffer");
        return;
    }

    uint32_t out_len = 0;
    uint8_t *frame_buf = tkl_system_psram_malloc(sg_audio_info.frame_size);
    if (NULL == frame_buf) {
        PR_ERR("tkl_system_psram_malloc failed");
        return;
    }

    do {
        memset(frame_buf, 0, sg_audio_info.frame_size);
        out_len = 0;

        data_len = tuya_ring_buff_used_size_get(sg_recorder_pcm_rb);
        if (data_len == 0) {
            PR_NOTICE("No data in recorder ring buffer");
            break;
        }

        if (data_len >sg_audio_info.frame_size) {
            tuya_ring_buff_read(sg_recorder_pcm_rb, frame_buf, sg_audio_info.frame_size);
            out_len = sg_audio_info.frame_size;
        } else {
            tuya_ring_buff_read(sg_recorder_pcm_rb, frame_buf, data_len);
            out_len = data_len;
        }

        tdl_audio_play(sg_audio_hdl, frame_buf, out_len);
    } while (1);

    if (frame_buf) {
        tkl_system_psram_free(frame_buf);
        frame_buf = NULL;
    }
}

/**
 * @brief       Handles incoming audio frames and writes them to the recorder ring buffer
 * @param[in]   type    Audio frame format type
 * @param[in]   status  Audio status
 * @param[in]   data    Pointer to audio frame data
 * @param[in]   len     Length of the audio frame data
 * @return      None
 * @note
 *      - Only writes data to the ring buffer when the recorder is in the RECORDING status
 *      - Checks if there is enough free space in the ring buffer before writing
 *      - If the buffer does not have enough space, a warning is printed and the data is not written
 */
static void get_audio_frame(TDL_AUDIO_FRAME_FORMAT_E type, TDL_AUDIO_STATUS_E status,\
                                      uint8_t *data, uint32_t len)
{
    type = type;
    status = status;
    if (RECORDER_STATUS_RECORDING != sg_recorder_status) {
        return;
    }

    if (sg_recorder_pcm_rb) {
        uint32_t free_size = tuya_ring_buff_free_size_get(sg_recorder_pcm_rb);
        if (free_size < len) {
            PR_WARN("recorder ring buffer overflow, free_size:%u, need_size:%u", free_size, len);
            return;
        }
        tuya_ring_buff_write(sg_recorder_pcm_rb, data, len);
    }

    return;
}

/**
 * @brief       Initialize audio input and output
 * @param[in]   none
 * @return      none
 * @note        Configures audio parameters:
 *              - Sample rate: 16kHz
 *              - Bit depth: 16 bits
 *              - Channel: Mono
 *              - Microphone volume: 100
 *              - Speaker volume: 80
 */
static OPERATE_RET audio_init(void)
{
    OPERATE_RET rt = OPRT_OK;
    uint32_t buf_len = 0;

    TUYA_CALL_ERR_RETURN(tdl_audio_find(AUDIO_CODEC_NAME, &sg_audio_hdl));
    TUYA_CALL_ERR_RETURN(tdl_audio_open(sg_audio_hdl, get_audio_frame));
    TUYA_CALL_ERR_RETURN(tdl_audio_get_info(sg_audio_hdl, &sg_audio_info));
    if(0 == sg_audio_info.frame_size || 0 == sg_audio_info.sample_tm_ms) {
        PR_ERR("get audio info err");
        return OPRT_INVALID_PARM;
    }

    buf_len = (RECORD_DURATION_MS / sg_audio_info.sample_tm_ms) * sg_audio_info.frame_size;
    TUYA_CALL_ERR_RETURN(tuya_ring_buff_create(buf_len, \
                                               OVERFLOW_PSRAM_STOP_TYPE, &sg_recorder_pcm_rb));

    tdl_audio_volume_set(sg_audio_hdl, 60);

    PR_NOTICE("audio_open success");

    return OPRT_OK;
}

/**
 * @brief       Main audio recorder function
 * @param[in]   none
 * @return      none
 * @note        Main loop for audio recording and playback:
 *              - Press KEY: Start recording
 *              - Release KEY: Stop recording, convert to WAV, and playback
 *              - Recordings are saved as incremental files (record_00.wav, record_01.wav, ...)
 *              - PCM files are deleted after playback to save space
 *              - Maximum recording duration: 3 seconds (limited by buffer size)
 */
void mic_recorder(void)
{
    OPERATE_RET rt = OPRT_OK;
    audio_init();

#if defined(ENABLE_BUTTON) && (ENABLE_BUTTON == 1)
    // button create
    TDL_BUTTON_CFG_T button_cfg = {.long_start_valid_time = 3000,
                                   .long_keep_timer = 1000,
                                   .button_debounce_time = 50,
                                   .button_repeat_valid_count = 0,
                                   .button_repeat_valid_time = 500};
    TDL_BUTTON_HANDLE button_hdl = NULL;

    TUYA_CALL_ERR_LOG(tdl_button_create(BUTTON_NAME, &button_cfg, &button_hdl));

    tdl_button_event_register(button_hdl, TDL_BUTTON_PRESS_DOWN, __button_function_cb);
    tdl_button_event_register(button_hdl, TDL_BUTTON_PRESS_UP, __button_function_cb);
#endif

    while(1) {
        switch(sg_recorder_status) {
            case RECORDER_STATUS_START:
                PR_NOTICE("Start recording");
                sg_recorder_status = RECORDER_STATUS_RECORDING;
                break;

            case RECORDER_STATUS_RECORDING:
                break;

            case RECORDER_STATUS_END:
                PR_NOTICE("End recording");

                /* Save audio to WAV file */
                __save_audio_to_wav_file();

                sg_recorder_status = RECORDER_STATUS_PLAYING;
                break;

            case RECORDER_STATUS_PLAYING:
                PR_NOTICE("Start playing");
                play_from_recorder_rb();
                PR_NOTICE("End playing");
                sg_recorder_status = RECORDER_STATUS_IDLE;
                break;
            case RECORDER_STATUS_IDLE:
                tuya_ring_buff_reset(sg_recorder_pcm_rb);
                break;
            default:
                break;
        }

        tal_system_sleep(10);
    }
}
```

mic_recorder函数主要实现录音以及播放效果，长按录音，松开播放，并且保存文件到TF卡中。

wav_encode.c和wav_encode.h文件内容主要是WAV文件的一些信息，这里就不罗列出来。

### 2，CMakeLists.txt文件

CMakeLists.txt文件配置内容如下。

```
# Add Components
set(module_dirs
    MIC_RECORD
)

foreach(dir ${module_dirs})
    set(MODULE_SRC_DIR ${APP_PATH}/components/Middlewares/${dir})
    aux_source_directory(${MODULE_SRC_DIR} MODULE_SRC)
    target_sources(${EXAMPLE_LIB}
        PRIVATE
            ${MODULE_SRC}
    )
    target_include_directories(${EXAMPLE_LIB}
        PRIVATE
            ${MODULE_SRC_DIR}
    )
endforeach()
```

### 3，main.c驱动代码

在main.c里面编写如下代码。

```C
#include "tkl_gpio.h"
#include "tftlcd.h"
#include "tfcard.h"
#include "mic_record.h"
#include "key.h"


/**
 * @brief       user_main
 *
 * @param[in]   none
 * @return      none
 */
void user_main(void)
{
    tal_log_init(TAL_LOG_LEVEL_DEBUG, 1024, (TAL_LOG_OUTPUT_CB)tkl_log_output); /* Initialize log output */
    tdd_disp_atk_tftlcd_register();                                             /* Register the TFTLCD LCD display */

    tftlcd_show_string(30, 50, 200, 16, 16, "T5 MINI Board", RED);
    tftlcd_show_string(30, 70, 200, 16, 16, "RECORD TEST", RED);
    tftlcd_show_string(30, 90, 200, 16, 16, "ATOM@ALIENTEK", RED);

    while (tfcard_init())
    {
        tftlcd_show_string(30, 110, 200, 16, 16, "SD Mount Failed!", RED);
        tal_system_sleep(500);
        tftlcd_show_string(30, 130, 200, 16, 16, "Please Check!", RED);
        tal_system_sleep(500);
    }

    /* Display operation instructions */
    tftlcd_show_string(30, 110, 200, 16, 16, "SD Card Ready!", GREEN);
    tftlcd_show_string(30, 130, 200, 16, 16, "Press KEY: Record", BLUE);
    tftlcd_show_string(30, 150, 200, 16, 16, "Release play", BLUE);
    /* Start audio recorder */
    mic_recorder();
}

#if OPERATING_SYSTEM == SYSTEM_LINUX

/**
 * @brief       main
 *
 * @param       argc
 * @param       argv
 * @return      none
 */
void main(int argc, char *argv[])
{
    user_main();
}
#else

/**
 * @brief       The application entry point
 * 
 * @param[in]   none
 * @return      none
 */
void tuya_app_main(void)
{
    user_main();
}
#endif
```

从user_main函数可以看到，初始化相关硬件之后，就调用mic_recorder函数去执行录音以及播放的功能。

## 运行验证

程序下载完成后，长按KEY，MIC读取音频数据，然后将音频数据转化为WAV格式文件，并存储在SD卡中。释放KEY，停止录音，并播放录音文件。
