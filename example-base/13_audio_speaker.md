---
title: '音乐播放器实验'
sidebar_position: 1
---

# 音乐播放器实验

## 前言

本章实验将介绍如何使用TuyaOpen让T5对SD卡上的MP3进行解码播放。

## MP3解码库介绍

### 概述

Minimp3 是一个轻量级的 MP3 解码库，专为资源受限的嵌入式系统设计，具有简单、快速和占用内存少的特点。以下是对其功能、特点及用途的详细介绍：

------

### 一、功能

Minimp3 是一个用于解码 MP3 音频文件的组件，其核心功能包括：

1. **MP3 解码**：能够将 MP3 格式的音频文件解码为 PCM（脉冲编码调制）数据，便于后续处理或播放。
2. **支持多种采样率和比特率**：兼容常见的 MP3 比特率（如 VBR 和 CBR）以及多种采样率。
3. **跨平台运行**：可在嵌入式系统、移动设备和桌面应用等多种平台上使用。

------

### 二、特点

Minimp3 的设计目标是尽可能减少代码体积和运行时内存消耗，同时保持高效解码性能，其主要特点包括：

1. **小巧轻量**：源代码体积极小，仅几百行代码，适合资源受限的环境。
2. **高效解码**：支持 SSE 和 NEON 指令集优化，能在低功耗设备上快速解码 MP3 文件。
3. **无依赖性**：不依赖任何外部库，可独立运行。
4. **开源**：采用开源协议，开发者可以自由使用和修改。
5. **ISO 标准兼容**：遵循 ISO 标准，确保解码精度和音质准确性。

## API 描述

**1，mp3dec_init函数**

初始化MP3解码库。

```C
void mp3dec_init(mp3dec_t *dec);
```

**1.1 参数描述**

`dec`：MP3参数配置

```C
typedef struct {
    float mdct_overlap[2][9 * 32], qmf_state[15 * 2 * 32];
    int reserv, free_format_bytes;
    unsigned char header[4], reserv_buf[511];
} mp3dec_t;
```

**1.2 返回值**

无。

**2，mp3dec_decode_frame函数**

解码MP3音频数据。

```C
int mp3dec_decode_frame(mp3dec_t *dec, const uint8_t *mp3, int mp3_bytes, mp3d_sample_t *pcm, mp3dec_frame_info_t *info);
```

**2.1 参数描述**

`dec`：MP3句柄

`mp3`：待解码的数据。

`mp3_bytes`：解码大小。

`pcm`：解码后的存储区。

`info`：MP3解码信息。

**2.2 返回值**

0表示解码成功，其他表示解码失败。

## 硬件设计

### 例程功能

1，从SD卡循环播放MP3音频文件，双击下一首，长按暂停。

> [!NOTE]
>
> TuyaOpen不能读取SD卡上的中文名称文件。

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

正点原子T5 AI开发板上SD卡和音频电路的连接原理图，如下图所示。

![](.\img\08.png)

![](.\img\10.png)

## 程序设计

### 1，音频驱动代码

这里我们只讲解核心代码，详细的源码请大家参考光盘资料本实验对应的源码。音频驱动源码包括两个文件：audio_speaker.c和audio_speaker.h。他们位于components\Middlewares\MINIMP3文件夹下。

audio_speaker.h文件是对音频配置参数做了相关定义以及函数声明。

```C
#define AUDIO_INPUT_CH              TKL_AI_1
#define AUDIO_OUTPUT_CH             TKL_AO_1
#define AUDIO_CH_NUM                TKL_AUDIO_CHANNEL_MONO
#define AUDIO_TYPE                  TKL_AUDIO_TYPE_BOARD
#define AUDIO_CODEC_TYPE            TKL_CODEC_AUDIO_PCM
#define AUDIO_SAMPLE_RATE           TKL_AUDIO_SAMPLE_16K
#define AUDIO_SAMPLE_BITS           16

#define MP3_DATA_BUF_SIZE           1940

#define MAX_NGRAN                   2     /* max granules */
#define MAX_NCHAN                   2     /* max channels */
#define MAX_NSAMP                   576   /* max samples per channel, per granule */

#define PCM_SIZE_MAX (MAX_NSAMP * MAX_NCHAN * MAX_NGRAN)

#define SPEAKER_ENABLE_PIN          TUYA_GPIO_NUM_17

#define MP3_FILE_ARRAY              media_src_hello_tuya_16k
#define MP3_FILE_INTERNAL_FLASH     "/media/hello_tuya.mp3"
#define MP3_FILE_SD_CARD            "/sdcard/MUSIC/hello_tuya.mp3"
#define MP3_FILE_PATH               "/sdcard/MUSIC"

#define MAX_FILENAME_LEN            128     /* maximum filename length */

/* MP3 file list node structure */
typedef struct mp3_file_node
{
    char file_path[MAX_FILENAME_LEN];       /* full file path */
    struct mp3_file_node *next;             /* next node pointer */
} MP3_FILE_NODE;

struct speaker_mp3_ctx
{
    mp3dec_t *mp3_dec;                  /* mp3 decoder */
    mp3dec_frame_info_t mp3_frame_info; /* mp3 frame info */
    unsigned char *read_buf;            /* mp3 data buffer */
    uint32_t read_size;                 /* valid data size in read_buf */
    uint32_t mp3_offset;                /* current mp3 read position */
    short *pcm_buf;                     /* pcm data buffer */
    uint32_t current_sample_rate;       /* current hardware sample rate */
    short *resample_buf;                /* resampling output buffer */
    uint32_t total_samples;             /* total samples (updated dynamically) */
    uint32_t played_samples;            /* samples played so far */
    uint32_t file_size;                 /* MP3 file size in bytes */
    uint32_t bytes_decoded;             /* bytes decoded so far */
    BOOL_T duration_finalized;          /* TRUE when total duration is known */
};

/* Function Declaration */
void mp3_decode_init(void);
OPERATE_RET speaker_init(void);
void audio_play(void);
int audio_play_single_file(const char *mp3_file_path);
MP3_FILE_NODE* scan_mp3_folder(const char *folder_path);
void free_mp3_list(MP3_FILE_NODE *head);
void audio_play_folder_loop(const char *folder_path);
```

audio_speaker.c文件是音频文件播放相关函数。

```C
/* Global flag to skip to next track */
static BOOL_T sg_skip_to_next = FALSE;

/* Global flag for pause/resume state */
static BOOL_T sg_is_paused = FALSE;

struct speaker_mp3_ctx  sg_mp3_ctx = {
    .mp3_dec = NULL,
    .read_buf = NULL,
    .read_size = 0,
    .mp3_offset = 0,
    .pcm_buf = NULL,
    .current_sample_rate = 16000,  /* default 16kHz */
    .resample_buf = NULL,
    .total_samples = 0,
    .played_samples = 0,
    .file_size = 0,
    .bytes_decoded = 0,
    .duration_finalized = FALSE,
};

/**
 * @brief       Format time in seconds to MM:SS string
 *
 * @param[in]   seconds: time in seconds
 * @param[out]  buf: output buffer (min 6 bytes)
 * @return      none
 */
static void format_time(uint32_t seconds, char *buf)
{
    uint32_t minutes = seconds / 60;
    uint32_t secs = seconds % 60;
    snprintf(buf, 6, "%02d:%02d", minutes, secs);
}

/**
 * @brief       Estimate MP3 duration based on file size and average bitrate
 *
 * @param[in]   file_size: MP3 file size in bytes
 * @param[in]   avg_bitrate_kbps: average bitrate in kbps
 * @param[in]   sample_rate: sample rate in Hz
 * @return      estimated total samples
 */
static uint32_t estimate_mp3_duration(uint32_t file_size, int avg_bitrate_kbps, int sample_rate)
{
    if (avg_bitrate_kbps == 0 || sample_rate == 0)
    {
        return 0;
    }

    /* duration_seconds = file_size_bits / bitrate = (file_size * 8) / (bitrate * 1000) */
    /* total_samples = duration_seconds * sample_rate */
    uint32_t duration_seconds = (file_size * 8) / (avg_bitrate_kbps * 1000);
    return duration_seconds * sample_rate;
}

/**
 * @brief       Resample PCM data using linear interpolation
 *
 * @param[in]   input: input PCM buffer
 * @param[in]   input_samples: number of input samples (mono or stereo counted as pairs)
 * @param[in]   input_rate: input sample rate in Hz
 * @param[out]  output: output PCM buffer
 * @param[in]   output_rate: output sample rate in Hz (typically 16000)
 * @param[in]   channels: number of channels (1=mono, 2=stereo)
 * @return      number of output samples
 */
static int resample_pcm_linear(const short *input, int input_samples, int input_rate,
                                short *output, int output_rate, int channels)
{
    static BOOL_T first_call = TRUE;

    if (input_rate == output_rate)
    {
        /* No resampling needed, direct copy */
        memcpy(output, input, input_samples * channels * sizeof(short));
        return input_samples;
    }

    float ratio = (float)input_rate / (float)output_rate;
    int output_samples = (int)(input_samples / ratio);
    int i, ch;

    /* Debug: log resampling info on first call */
    if (first_call)
    {
        PR_NOTICE("Resampling: %d Hz -> %d Hz, ratio=%.3f, in_samples=%d, out_samples=%d, channels=%d",
                  input_rate, output_rate, ratio, input_samples, output_samples, channels);
        first_call = FALSE;
    }

    /* Perform linear interpolation resampling */
    for (i = 0; i < output_samples; i++)
    {
        float src_pos = i * ratio;
        int src_index = (int)src_pos;
        float frac = src_pos - src_index;

        /* Clamp to valid range */
        if (src_index < 0)
        {
            src_index = 0;
            frac = 0.0f;
        }
        if (src_index >= input_samples - 1)
        {
            src_index = input_samples - 2;
            frac = 1.0f;
        }

        /* Safety check for very small buffers */
        if (src_index < 0 || input_samples < 2)
        {
            /* Just copy first sample if buffer too small */
            for (ch = 0; ch < channels; ch++)
            {
                output[i * channels + ch] = input[ch];
            }
            continue;
        }

        /* Linear interpolation for each channel */
        for (ch = 0; ch < channels; ch++)
        {
            int idx0 = src_index * channels + ch;
            int idx1 = (src_index + 1) * channels + ch;
            int sample0 = input[idx0];
            int sample1 = input[idx1];
            /* Use integer arithmetic to prevent floating point errors */
            int interpolated = sample0 + (int)((sample1 - sample0) * frac);

            output[i * channels + ch] = (short)interpolated;
        }
    }

    return output_samples;
}

/**
 * @brief       Convert stereo to mono by averaging left and right channels
 *              Uses safe arithmetic to prevent overflow
 *
 * @param[in]   input: stereo PCM buffer (interleaved LRLRLR format)
 * @param[in]   samples: number of samples per channel
 * @param[out]  output: mono PCM buffer
 * @return      number of mono samples
 */
static int stereo_to_mono(const short *input, int samples, short *output)
{
    /* Safety check */
    if (samples <= 0 || input == NULL || output == NULL)
    {
        PR_ERR("Invalid parameters in stereo_to_mono");
        return 0;
    }

    for (int i = 0; i < samples; i++)
    {
        int left = input[i * 2];
        int right = input[i * 2 + 1];
        /* Safe average: divide each channel first to prevent overflow */
        int avg = (left / 2) + (right / 2);

        /* Clamp to valid range to prevent any potential overflow */
        if (avg > 32767) avg = 32767;
        if (avg < -32768) avg = -32768;

        output[i] = (short)avg;
    }
    return samples;
}

/**
 * @brief       Check if filename contains Chinese characters
 *
 * @param[in]   filename: filename string to check
 * @return      TRUE if contains Chinese, FALSE otherwise
 */
static BOOL_T is_chinese_filename(const char *filename)
{
    if (filename == NULL)
    {
        return FALSE;
    }

    /* Check each byte in the filename */
    while (*filename)
    {
        unsigned char c = (unsigned char)*filename;

        /* Chinese characters in UTF-8 are typically 3 bytes starting with 0xE0-0xEF
         * Or any non-ASCII character (> 0x7F) */
        if (c > 0x7F)
        {
            return TRUE;  /* Found non-ASCII character, likely Chinese */
        }
        filename++;
    }

    return FALSE;
}

/**
 * @brief       mp3 decode init
 *
 * @param[in]   none
 * @return      none
 */
void mp3_decode_init(void)
{
    sg_mp3_ctx.read_buf = tkl_system_psram_malloc(MP3_DATA_BUF_SIZE);
    if (sg_mp3_ctx.read_buf == NULL) 
    {
        PR_ERR("mp3 read buf malloc failed!");
        return;
    }

    sg_mp3_ctx.pcm_buf = tkl_system_psram_malloc(PCM_SIZE_MAX * 2);
    if (sg_mp3_ctx.pcm_buf == NULL) 
    {
        PR_ERR("pcm_buf malloc failed!");
        return;
    }

    sg_mp3_ctx.mp3_dec = (mp3dec_t *)tkl_system_psram_malloc(sizeof(mp3dec_t));
    if (NULL == sg_mp3_ctx.mp3_dec)
    {
        PR_ERR("malloc mp3dec_t failed");
        return;
    }

    sg_mp3_ctx.resample_buf = tkl_system_psram_malloc(PCM_SIZE_MAX * 2);
    if (sg_mp3_ctx.resample_buf == NULL)
    {
        PR_ERR("resample_buf malloc failed!");
        return;
    }

    mp3dec_init(sg_mp3_ctx.mp3_dec);    /* init mp3 decoder */

    return;
}


/**
 * @brief       audio frame put callback
 *
 * @param[in]   pframe : audio frame info
 * @return      int
 */
int _audio_frame_put(TKL_AUDIO_FRAME_INFO_T *pframe)
{
    (void)pframe;
    return 0;
}

/**
 * @brief       speaker init
 *
 * @param[in]   none
 * @return      none
 */
OPERATE_RET speaker_init(void)
{
    OPERATE_RET rt = OPRT_OK;
    TKL_AUDIO_CONFIG_T config = {0};

    config.enable = true;
    config.card = AUDIO_TYPE;
    config.ai_chn = AUDIO_INPUT_CH;
    config.sample = AUDIO_SAMPLE_RATE;
    config.datebits = AUDIO_SAMPLE_BITS;
    config.channel = AUDIO_CH_NUM;
    config.codectype = AUDIO_CODEC_TYPE;
    config.put_cb = _audio_frame_put;

    config.fps = 25;            /* frame per second，suggest 25 */
    config.mic_volume = 0x2d;
    config.spk_volume = 0x2d;

    config.spk_gpio_polarity = 0;
    config.spk_sample = AUDIO_SAMPLE_RATE;
    config.spk_gpio = SPEAKER_ENABLE_PIN;

    TUYA_CALL_ERR_RETURN(tkl_ai_init(&config, 1));                                  /* init ai */
    TUYA_CALL_ERR_RETURN(tkl_ai_start(AUDIO_TYPE, AUDIO_INPUT_CH));                 /* start ai */
    TUYA_CALL_ERR_RETURN(tkl_ai_set_vol(AUDIO_TYPE, AUDIO_INPUT_CH, 80));           /* set ai volume */
    TUYA_CALL_ERR_RETURN(tkl_ao_set_vol(AUDIO_TYPE, AUDIO_OUTPUT_CH, NULL, 60));    /* set ao volume */

    sg_mp3_ctx.current_sample_rate = 16000;  /* record current sample rate */

    return rt;
}

/**
 * @brief       audio_play_single_file - play a single mp3 file with automatic resampling
 *
 * @param[in]   mp3_file_path: path to the mp3 file
 * @return      0 on success, -1 on error
 */
int audio_play_single_file(const char *mp3_file_path)
{
    unsigned char *mp3_frame_head = NULL;
    uint32_t decode_size_remain = 0;
    uint32_t read_size_remain = 0;
    BOOL_T sample_rate_checked = FALSE;
    char info_buf[64];

    if (sg_mp3_ctx.mp3_dec == NULL || sg_mp3_ctx.read_buf == NULL || sg_mp3_ctx.pcm_buf == NULL)
    {
        PR_ERR("MP3Decoder init fail!");
        return -1;
    }

    memset(sg_mp3_ctx.read_buf, 0, MP3_DATA_BUF_SIZE);
    memset(sg_mp3_ctx.pcm_buf, 0, PCM_SIZE_MAX * 2);
    sg_mp3_ctx.read_size = 0;
    sg_mp3_ctx.mp3_offset = 0;
    sg_mp3_ctx.played_samples = 0;
    sg_mp3_ctx.total_samples = 0;
    sg_mp3_ctx.file_size = 0;
    sg_mp3_ctx.bytes_decoded = 0;
    sg_mp3_ctx.duration_finalized = FALSE;

    /* Extract and display filename on LCD */
    const char *filename = strrchr(mp3_file_path, '/');
    if (filename == NULL)
    {
        filename = mp3_file_path;  /* No path separator found, use entire string */
    }
    else
    {
        filename++;  /* Skip the '/' character */
    }

    /* Clear previous filename display area (only one line) */
    tftlcd_fill_rect(30, 130, 210, 16, WHITE);

    /* Check if filename contains Chinese characters */
    if (is_chinese_filename(filename))
    {
        /* Display generic name for Chinese songs to avoid garbled text */
        tftlcd_show_string(30, 130, 180, 16, 16, "Chinese Song", BLUE);
        PR_NOTICE("Detected Chinese filename: %s", filename);
    }
    else
    {
        /* Display actual filename for English/ASCII names */
        tftlcd_show_string(30, 130, 180, 16, 16, (char *)filename, BLUE);
    }

    BOOL_T is_exist = FALSE;
    tkl_fs_is_exist(mp3_file_path, &is_exist);
    if (is_exist == FALSE)
    {
        PR_ERR("mp3 file not exist!");
        return -1;
    }

    TUYA_FILE mp3_file = tkl_fopen(mp3_file_path, "r");
    if (NULL == mp3_file)
    {
        PR_ERR("open mp3 file %s failed!", mp3_file_path);
        return -1;
    }

    /* Get file size */
    tkl_fseek(mp3_file, 0, SEEK_END);
    sg_mp3_ctx.file_size = tkl_ftell(mp3_file);
    tkl_fseek(mp3_file, 0, SEEK_SET);
    PR_NOTICE("MP3 file size: %d bytes", sg_mp3_ctx.file_size);

    /* Frame counter for progress update */
    uint32_t frame_count = 0;

    /* Reset skip and pause flags at start of playback */
    sg_skip_to_next = FALSE;
    sg_is_paused = FALSE;

    do {
        /* Check key status every frame */
        KEY_STATUS_E key_status = get_key_status();

        /* Double click: skip to next track */
        if (key_status == KEY_STATUS_DOUBLE_CLICK)
        {
            PR_NOTICE("*** Key double clicked - skipping to next track ***");
            reset_key_status();
            sg_skip_to_next = TRUE;
            break;  /* Exit playback loop immediately */
        }

        /* Long press: toggle pause/resume */
        if (key_status == KEY_STATUS_LONG_PRESS_START)
        {
            sg_is_paused = !sg_is_paused;  /* Toggle pause state */
            reset_key_status();

            if (sg_is_paused)
            {
                PR_NOTICE("*** Playback PAUSED ***");
                /* Display PAUSED next to progress info, not blocking button hints */
                tftlcd_show_string(140, 150, 90, 16, 16, "[PAUSED]", RED);
            }
            else
            {
                PR_NOTICE("*** Playback RESUMED ***");
                /* Clear only the PAUSED text area (width=80, height=16) */
                tftlcd_fill_rect(140, 150, 80, 16, WHITE);
            }
        }

        /* If paused, wait and continue checking for resume */
        if (sg_is_paused)
        {
            tal_system_sleep(100);  /* Sleep 100ms while paused */
            continue;  /* Skip audio processing */
        }

        /* 1. read mp3 data */
        if (mp3_frame_head != NULL && decode_size_remain > 0) 
        {
            memmove(sg_mp3_ctx.read_buf, mp3_frame_head, decode_size_remain);
            sg_mp3_ctx.read_size = decode_size_remain;
        }

        read_size_remain = MP3_DATA_BUF_SIZE - sg_mp3_ctx.read_size;
        int fs_read_len = tkl_fread(sg_mp3_ctx.read_buf + sg_mp3_ctx.read_size, read_size_remain, mp3_file);
        if (fs_read_len <= 0) 
        {
            if (decode_size_remain == 0) 
            {   /* last frame data decoding and playback completed */
                PR_NOTICE("mp3 play finish!");
                break;
            } 
            else
            {
                goto __MP3_DECODE;
            }
        } 
        else 
        {
            sg_mp3_ctx.read_size += fs_read_len;
        }

        __MP3_DECODE:
        /* 2. decode mp3 data */
        mp3_frame_head = sg_mp3_ctx.read_buf;
        int samples = mp3dec_decode_frame(sg_mp3_ctx.mp3_dec, mp3_frame_head, sg_mp3_ctx.read_size, sg_mp3_ctx.pcm_buf, &sg_mp3_ctx.mp3_frame_info);

        if (samples == 0)
        {
            decode_size_remain += 64;
            PR_ERR("mp3dec_decode_frame failed!");
            break;
        }

        /* Check sample rate on first successful decode */
        if (!sample_rate_checked && sg_mp3_ctx.mp3_frame_info.hz != 0)
        {
            sample_rate_checked = TRUE;
            PR_NOTICE("MP3 sample rate: %d Hz, channels: %d, bitrate: %d kbps",
                      sg_mp3_ctx.mp3_frame_info.hz, sg_mp3_ctx.mp3_frame_info.channels,
                      sg_mp3_ctx.mp3_frame_info.bitrate_kbps);

            /* Notify if stereo will be converted to mono */
            if (sg_mp3_ctx.mp3_frame_info.channels == 2)
            {
                PR_NOTICE("Stereo MP3 detected - will convert to mono for playback");
            }

            /* Estimate total duration */
            sg_mp3_ctx.total_samples = estimate_mp3_duration(
                sg_mp3_ctx.file_size,
                sg_mp3_ctx.mp3_frame_info.bitrate_kbps,
                sg_mp3_ctx.mp3_frame_info.hz
            );

            uint32_t total_seconds = sg_mp3_ctx.total_samples / sg_mp3_ctx.mp3_frame_info.hz;
            PR_NOTICE("Estimated duration: %d seconds (%d samples)",
                      total_seconds, sg_mp3_ctx.total_samples);

            /* Display sample rate info if resampling is needed */
            if ((uint32_t)sg_mp3_ctx.mp3_frame_info.hz != sg_mp3_ctx.current_sample_rate)
            {
                PR_NOTICE("Resampling from %d Hz to %d Hz",
                          sg_mp3_ctx.mp3_frame_info.hz, sg_mp3_ctx.current_sample_rate);
            }
        }

        mp3_frame_head += sg_mp3_ctx.mp3_frame_info.frame_bytes;
        decode_size_remain = sg_mp3_ctx.read_size - sg_mp3_ctx.mp3_frame_info.frame_bytes;

        /* 3. Resample if needed, then play pcm data */
        TKL_AUDIO_FRAME_INFO_T frame;
        int output_samples;
        short *playback_buf;

        if ((uint32_t)sg_mp3_ctx.mp3_frame_info.hz != sg_mp3_ctx.current_sample_rate)
        {
            /* Need resampling */
            output_samples = resample_pcm_linear(
                sg_mp3_ctx.pcm_buf,
                samples,
                sg_mp3_ctx.mp3_frame_info.hz,
                sg_mp3_ctx.resample_buf,
                sg_mp3_ctx.current_sample_rate,
                sg_mp3_ctx.mp3_frame_info.channels
            );
            playback_buf = sg_mp3_ctx.resample_buf;
        }
        else
        {
            /* No resampling needed */
            output_samples = samples;
            playback_buf = sg_mp3_ctx.pcm_buf;
        }

        /* 4. Convert stereo to mono if needed (hardware is mono) */
        if (sg_mp3_ctx.mp3_frame_info.channels == 2)
        {
            /* Stereo to mono conversion (in-place) */
            stereo_to_mono(playback_buf, output_samples, playback_buf);
            frame.pbuf = (char *)playback_buf;
            frame.used_size = output_samples * 2;  /* mono: samples * 2 bytes */
        }
        else
        {
            /* Already mono */
            frame.pbuf = (char *)playback_buf;
            frame.used_size = output_samples * 2;  /* mono: samples * 2 bytes */
        }

        tkl_ao_put_frame(0, 0, NULL, &frame);

        /* Update played samples and bytes decoded counters */
        sg_mp3_ctx.played_samples += samples;
        sg_mp3_ctx.bytes_decoded += sg_mp3_ctx.mp3_frame_info.frame_bytes;
        frame_count++;

        /* Dynamically update total samples based on decode progress */
        if (!sg_mp3_ctx.duration_finalized && sg_mp3_ctx.bytes_decoded > 0)
        {
            /* Estimate total samples based on current progress:
             * total_samples = played_samples * file_size / bytes_decoded */
            if (sg_mp3_ctx.bytes_decoded < sg_mp3_ctx.file_size)
            {
                /* Still decoding, estimate remaining */
                uint64_t estimated_total = ((uint64_t)sg_mp3_ctx.played_samples * sg_mp3_ctx.file_size) / sg_mp3_ctx.bytes_decoded;
                sg_mp3_ctx.total_samples = (uint32_t)estimated_total;
            }
            else
            {
                /* Finished decoding, this is the exact duration */
                sg_mp3_ctx.total_samples = sg_mp3_ctx.played_samples;
                sg_mp3_ctx.duration_finalized = TRUE;
                PR_NOTICE("Final duration: %d seconds (%d samples)",
                          sg_mp3_ctx.total_samples / sg_mp3_ctx.mp3_frame_info.hz,
                          sg_mp3_ctx.total_samples);
            }
        }

        /* Update progress display every 200 frames (~8 seconds) to reduce LCD/SD conflict */
        if (frame_count % 200 == 0 && sg_mp3_ctx.total_samples > 0)
        {
            uint32_t current_seconds = sg_mp3_ctx.played_samples / sg_mp3_ctx.mp3_frame_info.hz;
            uint32_t total_seconds = sg_mp3_ctx.total_samples / sg_mp3_ctx.mp3_frame_info.hz;

            char time_current[6], time_total[6];
            format_time(current_seconds, time_current);
            format_time(total_seconds, time_total);

            /* Display progress on LCD - no fill_rect to save time */
            snprintf(info_buf, sizeof(info_buf), "%s/%s", time_current, time_total);
            tftlcd_show_string(30, 150, 90, 16, 16, info_buf, GREEN);
        }
    } while (1);

    if (mp3_file != NULL)
    {
        tkl_fclose(mp3_file);
        mp3_file = NULL;
    }

    return 0;
}

/**
 * @brief       audio_play - play the default mp3 file (for backward compatibility)
 *
 * @param[in]   none
 * @return      none
 */
void audio_play(void)
{
    audio_play_single_file(MP3_FILE_SD_CARD);
}

/**
 * @brief       scan_mp3_folder - scan folder for MP3 files
 *
 * @param[in]   folder_path: path to the folder
 * @return      pointer to the head of the MP3 file list, NULL if failed
 */
MP3_FILE_NODE* scan_mp3_folder(const char *folder_path)
{
    TUYA_DIR dir;
    TUYA_FILEINFO info;
    const char *name;
    BOOL_T is_regular;
    MP3_FILE_NODE *head = NULL, *tail = NULL;
    char file_path[MAX_FILENAME_LEN];
    int file_count = 0;

    /* Check if folder exists */
    BOOL_T is_exist = FALSE;
    tkl_fs_is_exist(folder_path, &is_exist);
    if (is_exist == FALSE)
    {
        PR_ERR("Folder %s not exist!", folder_path);
        return NULL;
    }

    /* Open directory */
    if (tkl_dir_open(folder_path, &dir) != 0)
    {
        PR_ERR("Open directory %s failed!", folder_path);
        return NULL;
    }

    /* Read all entries in directory */
    while (tkl_dir_read(dir, &info) == 0)
    {
        /* Check if entry is a regular file */
        if (tkl_dir_is_regular(info, &is_regular) == 0 && is_regular)
        {
            /* Get the entry name */
            if (tkl_dir_name(info, &name) == 0)
            {
                /* Check if file has .mp3 extension */
                const char *ext = strrchr(name, '.');
                if (ext && (strcmp(ext, ".mp3") == 0 || strcmp(ext, ".MP3") == 0))
                {
                    /* Create new node */
                    MP3_FILE_NODE *new_node = (MP3_FILE_NODE *)tkl_system_malloc(sizeof(MP3_FILE_NODE));
                    if (new_node == NULL)
                    {
                        PR_ERR("Malloc failed for MP3 file node!");
                        tkl_dir_close(dir);
                        free_mp3_list(head);
                        return NULL;
                    }

                    /* Build full file path */
                    snprintf(file_path, MAX_FILENAME_LEN, "%s/%s", folder_path, name);
                    strncpy(new_node->file_path, file_path, MAX_FILENAME_LEN - 1);
                    new_node->file_path[MAX_FILENAME_LEN - 1] = '\0';
                    new_node->next = NULL;

                    /* Add to list */
                    if (head == NULL)
                    {
                        head = tail = new_node;
                    }
                    else
                    {
                        tail->next = new_node;
                        tail = new_node;
                    }

                    file_count++;
                    PR_NOTICE("Found MP3: %s", new_node->file_path);
                }
            }
        }
    }

    /* Close directory */
    tkl_dir_close(dir);

    PR_NOTICE("Total %d MP3 files found in %s", file_count, folder_path);
    return head;
}

/**
 * @brief       free_mp3_list - free MP3 file list
 *
 * @param[in]   head: pointer to the head of the list
 * @return      none
 */
void free_mp3_list(MP3_FILE_NODE *head)
{
    MP3_FILE_NODE *current = head;
    MP3_FILE_NODE *next;

    while (current != NULL)
    {
        next = current->next;
        tkl_system_free(current);
        current = next;
    }
}

/**
 * @brief       audio_play_folder_loop - loop play all MP3 files in a folder
 *
 * @param[in]   folder_path: path to the folder
 * @return      none
 */
void audio_play_folder_loop(const char *folder_path)
{
    MP3_FILE_NODE *mp3_list = NULL;
    MP3_FILE_NODE *current = NULL;
    int total_files = 0;
    char info_buf[32];

    /* Display scanning status */
    tftlcd_show_string(30, 130, 200, 16, 16, "Scanning folder...", BLUE);

    /* Scan folder for MP3 files */
    mp3_list = scan_mp3_folder(folder_path);
    if (mp3_list == NULL)
    {
        PR_ERR("No MP3 files found in folder %s", folder_path);
        tftlcd_show_string(30, 130, 200, 16, 16, "No MP3 files!", RED);
        return;
    }

    /* Count total files */
    current = mp3_list;
    while (current != NULL)
    {
        total_files++;
        current = current->next;
    }

    /* Display file count (no fill_rect to reduce operations) */
    snprintf(info_buf, sizeof(info_buf), "Found %d files", total_files);
    tftlcd_show_string(30, 130, 180, 16, 16, info_buf, GREEN);
    tal_system_sleep(1000);  /* Show count for 1 second */

    /* Display button hints after scanning */
    tftlcd_show_string(30, 170, 200, 16, 16, "KEY: 2x=Next", GREEN);
    tftlcd_show_string(30, 190, 200, 16, 16, "     Long=Pause", GREEN);

    /* Loop play all MP3 files */
    current = mp3_list;
    int current_index = 1;
    
    while (1)
    {
        /* If reached end of list, restart from beginning */
        if (current == NULL)
        {
            current = mp3_list;
            current_index = 1;
        }

        /* Clear and display track number (only happens on track change, infrequent) */
        tftlcd_fill_rect(30, 150, 100, 16, WHITE);  /* Only clear one line, not 166 pixels */
        snprintf(info_buf, sizeof(info_buf), "[%d/%d]", current_index, total_files);
        tftlcd_show_string(30, 150, 70, 16, 16, info_buf, RED);

        /* Play current MP3 file */
        PR_NOTICE("Playing [%d/%d]: %s", current_index, total_files, current->file_path);
        audio_play_single_file(current->file_path);

        /* Move to next file */
        current = current->next;
        current_index++;

        /* Small delay between tracks */
        tal_system_sleep(1000);
    }

    /* Free the list (this code won't be reached in infinite loop) */
    free_mp3_list(mp3_list);
}
```

上述源码中，audio_play_folder_loop函数用来遍历播放整个文件夹的歌曲。

speaker_init函数主要对音频参数进行配置。

mp3_decode_init函数主要就是对音乐文件进行解码。

### 2，CMakeLists.txt文件

CMakeLists.txt文件配置内容如下。

```
# Add Components
set(module_dirs
    MINIMP3
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
#include "tal_api.h"
#include "tkl_output.h"
#include "tkl_memory.h"
#include "tftlcd.h"
#include "tfcard.h"
#include "audio_speaker.h"
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
    tftlcd_show_string(30, 70, 200, 16, 16, "MUSIC TEST", RED);
    tftlcd_show_string(30, 90, 200, 16, 16, "ATOM@ALIENTEK", RED);

    while (tfcard_init())
    {
        tftlcd_show_string(30, 110, 200, 16, 16, "SD Mount Failed!", RED);
        tal_system_sleep(500);
        tftlcd_show_string(30, 130, 200, 16, 16, "Please Check!", RED);
        tal_system_sleep(500);
    }

    key_init();             /* init key for next track control */
    mp3_decode_init();      /* init mp3 decode */
    speaker_init();         /* init speaker */

    tftlcd_show_string(30, 110, 200, 16, 16, "Now Playing:", RED);

    /* Display button hints (wait for folder scan to complete) */
    audio_play_folder_loop(MP3_FILE_PATH);  /* This function loops internally */
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

从user_main函数可以看到，初始化相关硬件之后，就初始化mp3解码以及音频解码初始化，最后调用audio_play_folder_loop函数去遍历播放整个音乐文件夹内容。

## 运行验证

程序下载完成后，必须检查SD卡根目录下存在MP3文件，然后系统循环播放MP3文件。
