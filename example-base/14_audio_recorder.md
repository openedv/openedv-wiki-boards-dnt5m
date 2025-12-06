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

## API 描述

**1，tkl_ai_init函数**

音频初始化。

```C
OPERATE_RET tkl_ai_init(TKL_AUDIO_CONFIG_T *pconfig, int32_t count);
```

**1.1 参数描述**

`pconfig`：解码文件路径

```C
typedef struct {
    uint32_t enable;                  	/* 是否使能 */
    uint32_t card;                    	/* 声卡编号 */
    TKL_AI_CHN_E ai_chn;              	/* 音频输入通道 */
    TKL_AUDIO_SAMPLE_E sample;        	/* 采样率 */
    TKL_AUDIO_DATABITS_E datebits;    	/* 数据位 */
    TKL_AUDIO_CHANNEL_E channel;      	/* 通道数量 */
    TKL_MEDIA_CODEC_TYPE_E codectype; 	/* 编码类型 */
    int32_t is_softcodec;             	/* 1、软编码，0、硬件编码 */
    uint32_t fps;                       /* 每秒帧数，建议25帧 */
    int32_t mic_volume;                 /* MIC增益 */
    int32_t spk_volume;                 /* SPK音量 */
    int32_t spk_volume_offset;          /* spk音量偏移[0,100] */
    int32_t spk_gpio;                   /* SPK放大器引脚数，<0，无放大器 */
    int32_t spk_gpio_polarity;          /* 引脚极性，0高使能，1低使能 */
    void *padta;
    TKL_FRAME_PUT_CB put_cb;            /*  音频回调函数 */
} TKL_AUDIO_CONFIG_T;
```

`count`：配置统计。

**1.2 返回值**

OPRT_OK表示成功，其他表示失败。

**2，tkl_ai_set_vol函数**

设置MIC增益。

```C
OPERATE_RET tkl_ai_set_vol(int32_t card, TKL_AI_CHN_E chn, int32_t vol);
```

**2.1 参数描述**

`card`：声卡编号

`chn`：通道数。

```C
typedef enum {
    TKL_AI_0 = 0,
    TKL_AI_1,
    TKL_AI_2,
    TKL_AI_3,
    TKL_AI_MAX,
} TKL_AI_CHN_E; // audio input channel
```

`vol`：屏增益。

**2.2 返回值**

OPRT_OK表示成功，其他表示失败。

**3，tkl_ai_start函数**

音频开始。

```C
OPERATE_RET tkl_ai_start(int32_t card, TKL_AI_CHN_E chn);
```

**3.1 参数描述**

`card`：声卡编号

`chn`：通道数。

```C
typedef enum {
    TKL_AI_0 = 0,
    TKL_AI_1,
    TKL_AI_2,
    TKL_AI_3,
    TKL_AI_MAX,
} TKL_AI_CHN_E; // audio input channel
```

**3.2 返回值：**

OPRT_OK表示成功，其他表示失败。

**4，tkl_ao_set_vol函数**

音频开始。

```C
OPERATE_RET tkl_ao_set_vol(int32_t card, TKL_AO_CHN_E chn, void *handle, int32_t vol);
```

**4.1 参数描述**

`card`：声卡编号

`chn`：通道数

```C
typedef enum {
    TKL_AI_0 = 0,
    TKL_AI_1,
    TKL_AI_2,
    TKL_AI_3,
    TKL_AI_MAX,
} TKL_AI_CHN_E; // audio input channel
```

`handle`：音频句柄

`vol`：MIC增益

**4.2 返回值：**

OPRT_OK表示成功，其他表示失败。

## 硬件设计

### 例程功能

1，测试T5自带 ADC 和 DAC 接口。长按KEY按键可持续录制当前的声音，释放KEY按键时，可播放录制的声音。

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

正点原子T5 AI开发板上MIC和SPK卡的连接原理图，如下图所示。

![](.\img\10.png)

## 程序设计

### 1，录音驱动代码

这里我们只讲解核心代码，详细的源码请大家参考光盘资料本实验对应的源码。录音源码包括4个文件：wav_encode.c、wav_encode.h、wav_record.c和wav_record.h。他们位于components\Middlewares\WAV文件夹下。

wav_record.h文件是对音频配置参数做了相关定义以及函数声明。

```C
/* Only use SD card for recording */
#define RECORDER_FILE_DIR      "/sdcard/RECORD"

#define SPEAKER_ENABLE_PIN TUYA_GPIO_NUM_17


#define MIC_SAMPLE_RATE TKL_AUDIO_SAMPLE_16K    /* MIC sample rate */
#define MIC_SAMPLE_BITS TKL_AUDIO_DATABITS_16   /* MIC sample bits */
#define MIC_CHANNEL TKL_AUDIO_CHANNEL_MONO      /* MIC channel */

#define MIC_RECORD_DURATION_MS (3 * 1000)       /* assist in determining the buffer size */
#define PCM_BUF_SIZE    (MIC_RECORD_DURATION_MS * MIC_SAMPLE_RATE * MIC_SAMPLE_BITS * MIC_CHANNEL / 8 / 1000)   /* RINGBUF size */
#define PCM_FRAME_SIZE  (10 * MIC_SAMPLE_RATE * MIC_SAMPLE_BITS * MIC_CHANNEL / 8 / 1000)                       /* 10ms PCM data size */

struct recorder_ctx
{
    TUYA_RINGBUFF_T pcm_buf;
    BOOL_T recording;           /* Recording status */
    BOOL_T playing;             /* Playing status */
    TUYA_FILE file_hdl;         /* Recording file handle */
    char current_pcm_path[128]; /* Current PCM file path */
    char current_wav_path[128]; /* Current WAV file path */
    uint32_t file_index;        /* File index counter */
};

/* Function Declaration */
void wav_recorder();
```

wav_record.c文件是音频文件生成以及播放相关函数。

```C
struct recorder_ctx sg_recorder_ctx =
{
    .pcm_buf = NULL,
    .recording = FALSE,
    .playing = FALSE,
    .file_hdl = NULL,
    .current_pcm_path = {0},
    .current_wav_path = {0},
    .file_index = 0,
};

/**
 * @brief       Audio frame callback function
 * @param[in]   pframe: Audio frame information
 * @return      Used size of the audio frame
 * @note        This callback is called by the audio driver when audio data is available
 *              It writes audio data to the ring buffer when recording is active
 */
static int _audio_frame_put(TKL_AUDIO_FRAME_INFO_T *pframe)
{
    if (NULL == sg_recorder_ctx.pcm_buf)
    {
        return pframe->used_size;
    }

    if (sg_recorder_ctx.recording)
    {
        tuya_ring_buff_write(sg_recorder_ctx.pcm_buf, pframe->pbuf, pframe->used_size);
    }

    return pframe->used_size;
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
static void audio_init(void)
{
    int ret = 0;
    TKL_AUDIO_CONFIG_T config ={0};

    config.enable = false;
    config.card = TKL_AUDIO_TYPE_BOARD;
    config.ai_chn = TKL_AI_0;
    config.spk_gpio_polarity= 0;                /* sample */
    config.sample = MIC_SAMPLE_RATE;            /* sample */
    config.datebits = MIC_SAMPLE_BITS;          /* datebit */
    config.channel = TKL_AUDIO_CHANNEL_MONO;    /* channel num */
    config.codectype = TKL_CODEC_AUDIO_PCM;     /* codec type */

    config.spk_gpio = SPEAKER_ENABLE_PIN;
    config.put_cb = _audio_frame_put;

    ret = tkl_ai_init(&config, 1);

    /* Set microphone input volume (0-100, higher = louder recording) */
    ret |= tkl_ai_set_vol(TKL_AUDIO_TYPE_BOARD, 0, 100);

    ret |= tkl_ai_start(TKL_AUDIO_TYPE_BOARD,0);

    /* Set speaker output volume (0-100, higher = louder playback) */
    tkl_ao_set_vol(TKL_AUDIO_TYPE_BOARD, 0, NULL, 80);

    return;
}

/**
 * @brief       Scan directory and find the next available file index
 * @param[in]   none
 * @return      Next available file index (0-99)
 * @note        Only scans WAV files since PCM files are deleted after playback
 *              Stops scanning after finding 10 consecutive empty slots for optimization
 *              Returns 0 if directory doesn't exist or no files found
 */
static uint32_t get_next_file_index(void)
{
    uint32_t max_index = 0;
    BOOL_T is_exist = FALSE;

    /* Check if directory exists */
    tkl_fs_is_exist(RECORDER_FILE_DIR, &is_exist);
    if (is_exist == FALSE)
    {
        PR_DEBUG("Directory does not exist, starting from index 0");
        return 0;
    }

    /* Open directory */
    TUYA_DIR dir_hdl;
    OPERATE_RET rt = tkl_dir_open(RECORDER_FILE_DIR, &dir_hdl);
    if (OPRT_OK != rt)
    {
        PR_ERR("Failed to open directory: %s", RECORDER_FILE_DIR);
        return 0;
    }

    /* Scan all files in directory */
    char full_path[300];
    uint32_t file_count = 0;
    uint32_t empty_count = 0;

    PR_NOTICE("Scanning directory: %s", RECORDER_FILE_DIR);

    /* Manual scan: try each possible index from 0 to 99 */
    /* Only check WAV files since PCM files are deleted after playback */
    for (int i = 0; i < 100; i++)
    {
        /* Check if record_XX.wav exists */
        snprintf(full_path, sizeof(full_path), "%s/record_%02d.wav", RECORDER_FILE_DIR, i);
        BOOL_T wav_exist = FALSE;
        tkl_fs_is_exist(full_path, &wav_exist);

        if (wav_exist)
        {
            PR_NOTICE("Found recording file: record_%02d.wav", i);
            max_index = (uint32_t)i;
            file_count++;
            empty_count = 0;  /* Reset empty counter */
        }
        else
        {
            empty_count++;
            /* Stop scanning if we found 10 consecutive empty slots */
            if (empty_count >= 10)
            {
                PR_NOTICE("Found 10 consecutive empty slots, stopping scan at index %d", i);
                break;
            }
        }
    }

    PR_NOTICE("Total recordings found: %d, max_index: %d", file_count, max_index);

    tkl_dir_close(dir_hdl);

    /* Return next available index */
    uint32_t next_index = max_index + 1;
    PR_DEBUG("Found max index %d, next index will be %d", max_index, next_index);

    return next_index;
}

/**
 * @brief       Read audio data from ring buffer and write to file
 * @param[in]   none
 * @return      none
 * @note        Called periodically in main loop to save recorded audio data
 *              Reads data from ring buffer and writes to PCM file
 */
static void mic_record(void)
{
    if (NULL == sg_recorder_ctx.file_hdl)
    {
        return;
    }

    uint32_t data_len = 0;
    data_len = tuya_ring_buff_used_size_get(sg_recorder_ctx.pcm_buf);
    if (data_len == 0)
    {
        return;
    }

    char *read_buf = tkl_system_psram_malloc(data_len);
    if (NULL == read_buf)
    {
        PR_ERR("tkl_system_psram_malloc failed");
        return;
    }

    /* Write to file */
    tuya_ring_buff_read(sg_recorder_ctx.pcm_buf, read_buf, data_len);
    uint32_t rt_len = tkl_fwrite(read_buf, data_len, sg_recorder_ctx.file_hdl);
    if (rt_len != data_len)
    {
        PR_ERR("write file failed, maybe disk full");
        PR_ERR("write len %d, data len %d", rt_len, data_len);
    }

    if (read_buf)
    {
        tkl_system_psram_free(read_buf);
        read_buf = NULL;
    }

    return;
}

/**
 * @brief       Play recorded audio from PCM file
 * @param[in]   none
 * @return      none
 * @note        Reads PCM file in chunks and sends to audio output
 *              Uses 10ms frame size for smooth playback
 */
static void app_recode_play_from_flash(void)
{
    /* Read file */
    TUYA_FILE file_hdl = tkl_fopen(sg_recorder_ctx.current_pcm_path, "r");
    if (NULL == file_hdl)
    {
        PR_ERR("open file failed: %s", sg_recorder_ctx.current_pcm_path);
        return;
    }

    uint32_t data_len = 0;
    char *read_buf = tkl_system_psram_malloc(PCM_FRAME_SIZE);
    if (NULL == read_buf)
    {
        PR_ERR("tkl_system_psram_malloc failed");
        return;
    }

    do {
        memset(read_buf, 0, PCM_FRAME_SIZE);
        data_len = tkl_fread(read_buf, PCM_FRAME_SIZE, file_hdl);
        if (data_len == 0)
        {
            break;
        }

        TKL_AUDIO_FRAME_INFO_T frame_info;
        frame_info.pbuf = read_buf;
        frame_info.used_size = data_len;
        tkl_ao_put_frame(0, 0, NULL, &frame_info);
    } while (1);

    if (read_buf)
    {
        tkl_system_psram_free(read_buf);
        read_buf = NULL;
    }

    if (file_hdl)
    {
        tkl_fclose(file_hdl);
        file_hdl = NULL;
    }
}

/**
 * @brief       Play recorded audio file
 * @param[in]   none
 * @return      none
 * @note        Wrapper function for playback functionality
 */
static void app_recode_play(void)
{
    app_recode_play_from_flash();
    return;
}

/**
 * @brief       Convert PCM file to WAV format
 * @param[in]   pcm_file: Path to the PCM file
 * @return      OPRT_OK on success, error code otherwise
 * @note        Creates WAV file by adding WAV header to PCM data
 *              Output WAV file path is stored in sg_recorder_ctx.current_wav_path
 */
static OPERATE_RET app_pcm_to_wav(char *pcm_file)
{
    OPERATE_RET rt = OPRT_OK;
    uint8_t *read_buf = NULL;
    uint8_t head[WAV_HEAD_LEN] = {0};
    uint32_t pcm_len = 0;
    uint32_t sample_rate = MIC_SAMPLE_RATE;
    uint16_t bit_depth = MIC_SAMPLE_BITS;
    uint16_t channel = MIC_CHANNEL;

    /* Get pcm file length */
    TUYA_FILE pcm_hdl = tkl_fopen(pcm_file, "r");
    if (NULL == pcm_hdl) 
    {
        PR_ERR("open file failed");
        return OPRT_FILE_OPEN_FAILED;
    }
    tkl_fseek(pcm_hdl, 0, 2);
    pcm_len = tkl_ftell(pcm_hdl);

    tkl_fclose(pcm_hdl);
    pcm_hdl = NULL;

    PR_DEBUG("pcm file len %d", pcm_len);
    if (pcm_len == 0) 
    {
        PR_ERR("pcm file is empty");
        return OPRT_COM_ERROR;
    }

    /* Get wav head */
    rt = app_get_wav_head(pcm_len, 1, sample_rate, bit_depth, channel, head);
    if (OPRT_OK != rt) 
    {
        PR_ERR("app_get_wav_head failed, rt = %d", rt);
        return rt;
    }

    /* TAL_PR_HEXDUMP_DEBUG("wav head", head, WAV_HEAD_LEN); */

    /* Create wav file */
    TUYA_FILE wav_hdl = tkl_fopen(sg_recorder_ctx.current_wav_path, "w");
    if (NULL == wav_hdl)
    {
        PR_ERR("open file: %s failed", sg_recorder_ctx.current_wav_path);
        rt = OPRT_FILE_OPEN_FAILED;
        goto __EXIT;
    }

    /* Write wav head */
    tkl_fwrite(head, WAV_HEAD_LEN, wav_hdl);

    /* Read pcm file */
    read_buf = tkl_system_psram_malloc(PCM_FRAME_SIZE);
    if (NULL == read_buf) 
    {
        PR_ERR("tkl_system_psram_malloc failed");
        /* return OPRT_COM_ERROR; */
        rt = OPRT_MALLOC_FAILED;
        goto __EXIT;
    }

    PR_DEBUG("pcm file len %d", pcm_len);
    pcm_hdl = tkl_fopen(pcm_file, "r");
    if (NULL == pcm_hdl) 
    {
        PR_ERR("open file failed");
        rt = OPRT_FILE_OPEN_FAILED;
        goto __EXIT;
    }

    tkl_fseek(pcm_hdl, WAV_HEAD_LEN, 0);

    /* Write wav data */
    do {
        memset(read_buf, 0, PCM_FRAME_SIZE);
        uint32_t read_len = tkl_fread(read_buf, PCM_FRAME_SIZE, pcm_hdl);
        if (read_len == 0) 
        {
            break;
        }

        tkl_fwrite(read_buf, read_len, wav_hdl);
    } while (1);

__EXIT:
    if (pcm_hdl) 
    {
        tkl_fclose(pcm_hdl);
        pcm_hdl = NULL;
    }

    if (wav_hdl) 
    {
        tkl_fclose(wav_hdl);
        wav_hdl = NULL;
    }

    if (NULL != read_buf) 
    {
        tkl_system_psram_free(read_buf);
        read_buf = NULL;
    }

    return rt;
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
void wav_recorder(void)
{
    OPERATE_RET rt = OPRT_OK;

    /* Create PCM buffer */
    if (NULL == sg_recorder_ctx.pcm_buf)
    {
        PR_DEBUG("create pcm buffer size %d", PCM_BUF_SIZE);
        rt = tuya_ring_buff_create(PCM_BUF_SIZE, OVERFLOW_PSRAM_STOP_TYPE, &sg_recorder_ctx.pcm_buf);
        if (OPRT_OK != rt)
        {
            PR_ERR("tuya_ring_buff_create failed, rt = %d", rt);
            return;
        }
    }

    audio_init();

    /* Initialize file index by scanning existing files */
    sg_recorder_ctx.file_index = get_next_file_index();
    PR_NOTICE("Starting with file index: %d", sg_recorder_ctx.file_index);

    for (;;)
    {
        mic_record();

        /* Get key status */
        KEY_STATUS_E key_status = get_key_status();

        /* Check if key is released - stop recording */
        if (key_status == KEY_STATUS_PRESS_UP)
        {
            reset_key_status();  /* Reset key status after handling */

            /* End recording */
            if (TRUE == sg_recorder_ctx.recording) 
            {
                sg_recorder_ctx.recording = FALSE;
                sg_recorder_ctx.playing = TRUE;

                if (sg_recorder_ctx.file_hdl)
                {
                    tkl_fclose(sg_recorder_ctx.file_hdl);
                    sg_recorder_ctx.file_hdl = NULL;
                }

                /* Convert pcm to wav */
                OPERATE_RET rt = app_pcm_to_wav(sg_recorder_ctx.current_pcm_path);
                if (OPRT_OK != rt)
                {
                    PR_ERR("app_pcm_to_wav failed, rt = %d", rt);
                }

                PR_DEBUG("stop recording");

                /* Display recording done on LCD */
                tftlcd_fill_rect(30, 210, 200, 16, WHITE);  /* Clear previous status */
                tftlcd_show_string(30, 210, 200, 16, 16, "Recording Done!", GREEN);
                tal_system_sleep(1000);
            }

            /* Start playing */
            if (TRUE == sg_recorder_ctx.playing) 
            {
                PR_DEBUG("start playing");

                /* Display playing status on LCD */
                tftlcd_fill_rect(30, 210, 200, 16, WHITE);  /* Clear previous status */
                tftlcd_show_string(30, 210, 200, 16, 16, "Playing...", BLUE);

                sg_recorder_ctx.playing = FALSE;
                app_recode_play();
                PR_DEBUG("stop playing");

                /* Delete PCM file after playing (keep only WAV) */
                BOOL_T is_exist = FALSE;
                tkl_fs_is_exist(sg_recorder_ctx.current_pcm_path, &is_exist);
                if (is_exist == TRUE)
                {
                    OPERATE_RET rt = tkl_fs_remove(sg_recorder_ctx.current_pcm_path);
                    if (OPRT_OK == rt)
                    {
                        PR_NOTICE("Deleted PCM file: %s", sg_recorder_ctx.current_pcm_path);
                    }
                    else
                    {
                        PR_ERR("Failed to delete PCM file: %s", sg_recorder_ctx.current_pcm_path);
                    }
                }

                /* Display playing done on LCD */
                tftlcd_fill_rect(30, 210, 200, 16, WHITE);  /* Clear previous status */
                tftlcd_show_string(30, 210, 200, 16, 16, "Play Done!", GREEN);

                /* Clear filename display */
                tftlcd_fill_rect(30, 230, 200, 16, WHITE);  /* Clear filename */
            }

            tal_system_sleep(100);
            continue;
        }

        /* Check if key is pressed down - start recording */
        if (key_status == KEY_STATUS_PRESS_DOWN)
        {
            reset_key_status();  /* Reset key status after handling */

            /* Start recording */
            if (FALSE == sg_recorder_ctx.recording)
            {
                /* Ensure RECORD directory exists */
                BOOL_T is_exist = FALSE;
                tkl_fs_is_exist(RECORDER_FILE_DIR, &is_exist);
                if (is_exist == FALSE)
                {
                    OPERATE_RET rt = tkl_fs_mkdir(RECORDER_FILE_DIR);
                    if (OPRT_OK == rt)
                    {
                        PR_DEBUG("Created directory: %s", RECORDER_FILE_DIR);
                    }
                    else
                    {
                        PR_ERR("Failed to create directory: %s", RECORDER_FILE_DIR);
                    }
                }

                /* Generate unique filename with incremental index */
                snprintf(sg_recorder_ctx.current_pcm_path, sizeof(sg_recorder_ctx.current_pcm_path),
                         "%s/record_%02d.pcm", RECORDER_FILE_DIR, sg_recorder_ctx.file_index);
                snprintf(sg_recorder_ctx.current_wav_path, sizeof(sg_recorder_ctx.current_wav_path),
                         "%s/record_%02d.wav", RECORDER_FILE_DIR, sg_recorder_ctx.file_index);

                PR_DEBUG("New recording file: %s (index %d)", sg_recorder_ctx.current_pcm_path, sg_recorder_ctx.file_index);

                /* Increment file index for next recording */
                sg_recorder_ctx.file_index++;

                /* Create recording file */
                sg_recorder_ctx.file_hdl = tkl_fopen(sg_recorder_ctx.current_pcm_path, "w");
                if (NULL == sg_recorder_ctx.file_hdl)
                {
                    PR_ERR("open file failed");
                    continue;
                }
                PR_DEBUG("open file %s success", sg_recorder_ctx.current_pcm_path);

                tuya_ring_buff_reset(sg_recorder_ctx.pcm_buf);
                sg_recorder_ctx.recording = TRUE;
                sg_recorder_ctx.playing = FALSE;
                PR_DEBUG("start recording");

                /* Display recording status on LCD */
                tftlcd_fill_rect(30, 210, 200, 16, WHITE);  /* Clear previous status */
                tftlcd_show_string(30, 210, 200, 16, 16, "Recording...", RED);

                /* Display current recording filename on LCD */
                char filename_display[32];
                snprintf(filename_display, sizeof(filename_display), "File: record_%02d.wav", sg_recorder_ctx.file_index - 1);
                tftlcd_fill_rect(30, 230, 200, 16, WHITE);  /* Clear previous filename */
                tftlcd_show_string(30, 230, 200, 16, 16, filename_display, BLUE);
            }
        }
        else
        {
            /* No key event, just continue */
            tal_system_sleep(10);
        }
    }
}
```

wav_recorder函数主要实现录音以及播放效果，长按录音，松开播放，并且保存文件到TF卡中。

wav_encode.c和wav_encode.h文件内容主要是WAV文件的一些信息，这里就不罗列出来。

### 2，CMakeLists.txt文件

CMakeLists.txt文件配置内容如下。

```
# Add Components
set(module_dirs
    MINIMP3
    WAV
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
#include "wav_record.h"
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
    tftlcd_show_string(30, 150, 200, 16, 16, "Release KEY: Stop", BLUE);
    tftlcd_show_string(30, 190, 200, 16, 16, "Save: SD Card", GREEN);

    tal_system_sleep(2000);  /* Show instructions for 2 seconds */

    /* Initialize key before starting recorder */
    key_init();

    /* Start audio recorder */
    wav_recorder();
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

从user_main函数可以看到，初始化相关硬件之后，就调用wav_recorder函数去执行录音以及播放的功能。

## 运行验证

程序下载完成后，长按KEY，MIC读取音频数据，然后将音频数据转化为PCM和WAV格式文件，并存储在SD卡中。释放KEY，停止录音，并播放录音文件。
