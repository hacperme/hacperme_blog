---
title: "SPI NAND Flash AS5F38G04SND-08LIN 笔记"
date: 2024-12-30T00:49:37+08:00
lastmod: 2024-12-30T00:49:37+08:00
author: ["hacper"]
tags:
    - SPI
    - NAND
    - FLASH
    - AS5F38G04SND-08LIN
categories:
    - 笔记
description: "SPI NAND Flash AS5F38G04SND-08LIN 驱动相关笔记" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: true # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
# mermaid: true
---

## AS5F38G04SND-08LIN flash 特点

- Single-Level Cell (SLC) NAND Flash
 SLC 和 MLC 的区别：SLC (Single-Level Cell)：每个存储单元只存储 1 位数据（0 或 1）。这使得 SLC 比 MLC 更可靠、更耐用。MLC (Multi-Level Cell)：每个存储单元可以存储 2 位数据（通常是 4 个电压级别，代表 00、01、10、11）。因此，MLC 的存储密度更高，但性能和耐用性略逊于 SLC。

- Clock Frequency
 Up to 120MHz (for VCC 3.3V)

- 支持 Standard, Dual and Quad SPI

- 8bit ECC for each 512bytes + 32bytes
- 容量
 Density：8Gbits
 ECC：8bit
 Page Size：4096(data storage region)+256(spare area) Bytes，data storage region 用来存储数据，spare area 用来做内存管理和数据纠错。
 Block: 64 Pages
 Device:4096 Blocks

- 功能框图
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20241230/image.70aeso4j53.webp)

Nand Flash 里面有一个缓存，读写 page 都要先操作 cache 内存，然后再从 cache 写入到flash里面或者从 cache 读出数据。 

- 内存寻址

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20241231/image.3rbax1n6vp.webp)

总共4096个block, 每个block 64 个page, 每个page 4096+256字节。
内存地址分行地址RA和列地址CA，CA 13 bit，用来寻址page里面的每个字节的位置，RA 前6 bit 表示block中地几个page的地址，RA高12 bit表示第几个block的地址。

单个page里面的内存布局，前面 4096 字节的数据存储区域，后面 256字节的 spare area。
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20241231/image.2oblm6c5n5.webp)


## 指令集

| 命令    | 命令传输方式 | 地址长度bytes | 地址传输方式 | dummy 字节 | 数据传输方式 | 备注                                                         |
| ------- | ------------ | ------------- | ------------ | ---------- | ------------ | ------------------------------------------------------------ |
| 04H     | 单线         | 0             |              | 0          |              | Write Disable                                                |
| 06H     | 单线         | 0             |              | 0          |              | Write Enable                                                 |
| D8H     | 单线         | 3             | 单线         | 0          |              | Block Erase (Block size)<br />24-bit address 由 6 dummy bits and 18  block address bits 组成<br />RA地址中的page地址会被自动忽略 |
| 02H     | 单线         | 2             | 单线         | 0          | 单线         | Program Load                                                 |
| 32H     | 单线         | 2             | 单线         | 0          | 四线         | Program Load x4 IO<br />需要设置QE bit                       |
| 10H     | 单线         | 3             | 单线         | 0          | 单线         | Program Execute                                              |
| 84H     | 单线         | 2             | 单线         |            | 单线         | Program Load Random Data                                     |
| C4H/34H | 单线         | 2             | 单线         | 0          | 四线         | Program Load Random Data x4 IO<br />需要设置QE bit           |
| 72H     | 单线         | 2             | 四线         | 0          | 四线         | Program Load Random Data Quad IO<br />需要设置QE bit         |
| 13H     | 单线         | 3             | 单线         | 0          |              | Page Read (to Cache)<br />24-bit address consists of 6 dummy bits and 18 page address bits |
| 03H/0BH | 单线         | 2             | 单线         | 0          | 单线         | Read from Cache x1 IO<br />16 bits of column address which consists of 3 wrap bits and 13 column address bits |
| 3BH     | 单线         | 2             | 单线         | 1          | 双线         | Read from Cache x2 IO                                        |
| 6BH     | 单线         | 2             | 单线         | 1          | 四线         | Read from Cache x4 IO                                        |
| BBH     | 单线         | 2             | 双线         | 1          | 双线         | Read from Cache Dual IO                                      |
| EBH     | 单线         | 2             | 四线         | 1          | 四线         | Read from Cache Quad IO<br />需要设置QE bit                  |
| 9FH     | 单线         | 1             | 单线         | 0          | 单线         | Read ID<br />address 00H： manufacturer ID<br />address 01H： device ID<br />datasheet 这两个ID是分开的地址，时序图是分开读取的，但实际驱动可以连续读2字节。 |
| FFH     | 单线         |               |              |            |              | Reset                                                        |
| 0FH     | 单线         | 1             | 单线         | 0          | 单线         | Get Feature                                                  |
| 1FH     | 单线         | 1             | 单线         | 0          | 单线         | Set Feature                                                  |

## 功能寄存器

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20241231/image.2oblmezelv.webp)

Feature 寄存器一共有 3 个：Block Lock、OTP 和 Status。

- Status 寄存器的含义

| **位 (Bit)**     | **名称 (Name)**                    | **描述 (Description)**                                       |
| ---------------- | ---------------------------------- | ------------------------------------------------------------ |
| **P_FAIL**       | 程序失败 (Program Fail)            | 表示发生了程序失败。如果用户尝试向无效地址或受保护区域（包括 OTP 区域）编程，也会设置该位。通过程序执行命令或 RESET 命令可以清除此位。 |
| **E_FAIL**       | 擦除失败 (Erase Fail)              | 表示发生了擦除失败。如果用户尝试擦除受保护的区域，也会设置该位。在块擦除命令序列开始或通过 RESET 命令时，此位被清除。 |
| **WEL**          | 写入使能锁存 (Write Enable Latch)  | 表示写入使能锁存器 (WEL) 的当前状态。必须设置 (WEL = 1) 后才能执行程序执行或块擦除命令。通过写入禁用命令 (WRITE DISABLE) 可禁用 (WEL = 0)。 |
| **OIP**          | 操作进行中 (Operation In Progress) | 在执行程序执行、页面读取、块擦除或 RESET 命令时，此位会被设置，表示设备正在忙碌。当该位为 0 时，接口处于就绪状态。 |
| **ECCS1, ECCS0** | ECC 状态 (ECC Status)              | 表示 ECC 状态： - **00b**：未检测到位错误 - **01b**：检测到位错误并已纠正 - **10b**：检测到位错误但未纠正 - **11b**：检测到位错误并已纠正，但错误位数量等于 ECC 最大值（由扩展寄存器定义）。

- Block Lock

Block Lock 寄存器提供块保护功能，可以设置整个flash、或者部分block写保护，避免被保护的block被意外擦除、编程，通过设置Block Lock 寄存器的BP0, BP1，BP2，NV, CMP and BRWD a来设置写保护。
flash 上电默认进入写保护状态。

Block Protection Bits Table：

| **CMP** | **INV** | **BP2** | **BP1** | **BP0** | **Protect Rows**     |
| ------- | ------- | ------- | ------- | ------- | -------------------- |
| X       | X       | 0       | 0       | 0       | All unlocked         |
| 0       | 0       | 0       | 0       | 1       | Upper 1/64 locked    |
| 0       | 0       | 0       | 1       | 0       | Upper 1/32 locked    |
| 0       | 0       | 0       | 1       | 1       | Upper 1/16 locked    |
| 0       | 0       | 1       | 0       | 0       | Upper 1/8 locked     |
| 0       | 0       | 1       | 0       | 1       | Upper 1/4 locked     |
| 0       | 0       | 1       | 1       | 0       | Upper 1/2 locked     |
| X       | X       | 1       | 1       | 1       | All locked (Default) |
| 0       | 1       | 0       | 0       | 1       | Lower 1/64 locked    |
| 0       | 1       | 0       | 1       | 0       | Lower 1/32 locked    |
| 0       | 1       | 0       | 1       | 1       | Lower 1/16 locked    |
| 0       | 1       | 1       | 0       | 0       | Lower 1/8 locked     |
| 0       | 1       | 1       | 0       | 1       | Lower 1/4 locked     |
| 0       | 1       | 1       | 1       | 0       | Lower 1/2 locked     |
| 1       | 0       | 0       | 0       | 1       | Lower 63/64 locked   |
| 1       | 0       | 0       | 1       | 0       | Lower 31/32 locked   |
| 1       | 0       | 0       | 1       | 1       | Lower 15/16 locked   |
| 1       | 0       | 1       | 0       | 0       | Lower 7/8 locked     |
| 1       | 0       | 1       | 0       | 1       | Lower 3/4 locked     |
| 1       | 0       | 1       | 1       | 0       | Block 0              |
| 1       | 1       | 0       | 0       | 1       | Upper 63/64 locked   |
| 1       | 1       | 0       | 1       | 0       | Upper 31/32 locked   |
| 1       | 1       | 0       | 1       | 1       | Upper 15/16 locked   |
| 1       | 1       | 1       | 0       | 0       | Upper 7/8 locked     |
| 1       | 1       | 1       | 0       | 1       | Upper 3/4 locked     |
| 1       | 1       | 1       | 1       | 1       | Block 0              |


- OTP

该 flash 64 page 大小的 OTP 区域，该区域只能编程一次。通过OTP的 OTP_PRT OTP_EN bit操作 OTP 区域。
OTP 状态 (OTP State)

| **OTP_PRT** | **OTP_EN** | **状态 (State)**                                             |
| ----------- | ---------- | ------------------------------------------------------------ |
| X           | 0          | 正常操作，无法访问 OTP 区域。                                |
| 0           | 1          | 允许访问 OTP 区域。可以执行 PAGE READ 和 PAGE PROGRAM 操作。 |
| 1           | 1          | 当设备上电时，OTP_PRT 有两种情况：                           |
|             |            | 1. 当设备上电时，OTP_PRT 为 0：用户可以通过 SET FEATURE 命令设置 OTP_PRT 和 OTP_EN 为 1，然后通过 PROGRAM EXECUTE (10H) 命令锁定 OTP 区域。一旦 OTP 区域被锁定，OTP_PRT 将永久变为 1。 |
|             |            | 2. 当设备上电时，OTP_PRT 为 1：用户只能读取 OTP 区域的数据。 |

OTP 页面定义 (OTP Page Definition)

| **页面地址 (Page Address)** | **页面名称 (Page Name)**  | **描述 (Description)**                       | **数据长度 (Data Length)** | **备注 (Notes)** |
| --------------------------- | ------------------------- | -------------------------------------------- | -------------------------- | ---------------- |
| 00h                         | 参数页面 (Parameter Page) | 出厂已编程，只读。                           | 256 字节 × 4               |                  |
| OTP 页面 [0]                | OTP Page [0]              |                                              |                            |                  |
| 01h ~ 3Fh                   | OTP 页面 [1:63]           | 当 OTP_PRT=0 时可读写；当 OTP_PRT=1 时只读。 | 2,176 字节                 |                  |
|                             |                           |                                              | 2,112 字节                 |                  |
|                             |                           |                                              | 4,352 字节                 |                  |

另外 ECC 使能和 QE bit 也在 OTP 寄存器设置。

## 操作时序

### 读 page

1. 13H (Page Read to Cache)
2. 0FH (GET FEATURE command to read the status)
   检查 OIP bit in status register (C0H) 是否设置 0;读ECCS1 ECCS0 判断是否存在ECC问题
3. Read from Cache memory
    - 03H or 0BH (Read from Cache x1 IO) / 3BH (Read from Cache x2 IO) / 6BH (Read from Cache x4 IO)
    - BBH (Read from Cache Dual IO) / EBH (Read from Cache Quad IO)

### 写 page

1. 06H (WRITE ENABLE when WEL bit is 0)
2. PROGRAM LOAD
    - 02H (PROGRAM LOAD) / 32H (PROGRAM LOAD x4)
3. 10H (PROGRAM EXECUTE)
4. FH (GET FEATURE command to read the status)
    OIP bit 看是否编程完成，P_FAIL 看是否编程出错

### 内部数据移动

1. 13H (Page Read to cache)
2. 0FH (GET FEATURE command to read the status)
3. Optional 84H/C4H/34H/72H **(PROGRAM LOAD RANDOM DATA. The command of Program load random data can be operated several times in this step.)
4. 06H (WRITE ENABLE)
5. 10H (PROGRAM EXECUTE)
6. 0FH (GET FEATURE command to read the status)
    - 84H/C4H/34H/72H commands are only available in Internal Data Move operation
P_FAIL 看是否编程出错

### 块擦除

1. 06H (WRITE ENABLE command)
2. D8H (BLOCK ERASE command)
3. FH (GET FEATURE command to read the status register)
E_FAIL bit 指示擦除是否成功


## 驱动

### 驱动接口

```c
struct spi_nand_transaction_t {
    uint8_t command;
    uint8_t address_bytes;
    uint32_t address;
    uint32_t mosi_len;
    const uint8_t *mosi_data;
    uint32_t miso_len;
    uint8_t *miso_data;
    uint32_t dummy_bits;
    uint32_t flags;
};

typedef struct spi_nand_transaction_t spi_nand_transaction_t;

#define CMD_SET_REGISTER    0x1F
#define CMD_READ_REGISTER   0x0F
#define CMD_WRITE_ENABLE    0x06
#define CMD_READ_ID         0x9F
#define CMD_PAGE_READ       0x13
#define CMD_PROGRAM_EXECUTE 0x10
#define CMD_PROGRAM_LOAD    0x84
#define CMD_PROGRAM_LOAD_X4 0x34
#define CMD_READ_FAST       0x0B
#define CMD_READ_X2         0x3B
#define CMD_READ_X4         0x6B
#define CMD_ERASE_BLOCK     0xD8

#define REG_PROTECT         0xA0
#define REG_CONFIG          0xB0
#define REG_STATUS          0xC0

#define STAT_BUSY           1 << 0
#define STAT_WRITE_ENABLED  1 << 1
#define STAT_ERASE_FAILED   1 << 2
#define STAT_PROGRAM_FAILED 1 << 3
#define STAT_ECC0           1 << 4
#define STAT_ECC1           1 << 5
#define STAT_ECC2           1 << 6

esp_err_t spi_nand_execute_transaction(spi_device_handle_t device, spi_nand_transaction_t *transaction);

esp_err_t spi_nand_read_register(spi_device_handle_t device, uint8_t reg, uint8_t *val);
esp_err_t spi_nand_write_register(spi_device_handle_t device, uint8_t reg, uint8_t val);
esp_err_t spi_nand_write_enable(spi_device_handle_t device);
esp_err_t spi_nand_read_page(spi_device_handle_t device, uint32_t page);
esp_err_t spi_nand_read(spi_device_handle_t device, uint8_t *data, uint16_t column, uint16_t length);
esp_err_t spi_nand_program_execute(spi_device_handle_t device, uint32_t page);
esp_err_t spi_nand_program_load(spi_device_handle_t device, const uint8_t *data, uint16_t column, uint16_t length);
esp_err_t spi_nand_erase_block(spi_device_handle_t device, uint32_t page);


esp_err_t nand_wrap_is_bad(spi_nand_flash_device_t *handle, uint32_t b, bool *is_bad_status);
esp_err_t nand_wrap_mark_bad(spi_nand_flash_device_t *handle, uint32_t b);
esp_err_t nand_wrap_erase_chip(spi_nand_flash_device_t *handle);
esp_err_t nand_wrap_erase_block(spi_nand_flash_device_t *handle, uint32_t b);
esp_err_t nand_wrap_prog(spi_nand_flash_device_t *handle, uint32_t p, const uint8_t *data);
esp_err_t nand_wrap_is_free(spi_nand_flash_device_t *handle, uint32_t p, bool *is_free_status);
esp_err_t nand_wrap_read(spi_nand_flash_device_t *handle, uint32_t p, size_t offset, size_t length, uint8_t *data);
esp_err_t nand_wrap_copy(spi_nand_flash_device_t *handle, uint32_t src, uint32_t dst);
```

### Dhara 介绍

[Dhara: NAND flash translation layer for small MCUs](https://github.com/dlbeer/dhara) 是一个 nand flash 管理组件，提供以下功能：
- 磨损均衡
- 掉电保护
- 逻辑块内存管理等

Dhara 适配接口实现

```c
typedef struct {
    struct dhara_nand dhara_nand;
    struct dhara_map dhara_map;
    spi_nand_flash_device_t *parent_handle;
} spi_nand_flash_dhara_priv_data_t;

static esp_err_t dhara_init(spi_nand_flash_device_t *handle)
{
    // create a holder structure for dhara context
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = calloc(1, sizeof(spi_nand_flash_dhara_priv_data_t));
    // save the holder inside the device structure
    handle->ops_priv_data = dhara_priv_data;
    // store the pointer back to device structure in the holder stucture
    dhara_priv_data->parent_handle = handle;

    dhara_priv_data->dhara_nand.log2_page_size = handle->chip.log2_page_size;
    dhara_priv_data->dhara_nand.log2_ppb = handle->chip.log2_ppb;
    dhara_priv_data->dhara_nand.num_blocks = handle->chip.num_blocks;

    dhara_map_init(&dhara_priv_data->dhara_map, &dhara_priv_data->dhara_nand, handle->work_buffer, handle->config.gc_factor);
    dhara_error_t ignored;
    dhara_map_resume(&dhara_priv_data->dhara_map, &ignored);

    return ESP_OK;
}

static esp_err_t dhara_deinit(spi_nand_flash_device_t *handle)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = (spi_nand_flash_dhara_priv_data_t *)handle->ops_priv_data;
    // clear dhara map
    dhara_map_init(&dhara_priv_data->dhara_map, &dhara_priv_data->dhara_nand, handle->work_buffer, handle->config.gc_factor);
    dhara_map_clear(&dhara_priv_data->dhara_map);
    return ESP_OK;
}

static esp_err_t dhara_read(spi_nand_flash_device_t *handle, uint8_t *buffer, dhara_sector_t sector_id)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = (spi_nand_flash_dhara_priv_data_t *)handle->ops_priv_data;
    dhara_error_t err;
    if (dhara_map_read(&dhara_priv_data->dhara_map, sector_id, handle->read_buffer, &err)) {
        return ESP_ERR_FLASH_BASE + err;
    }
    memcpy(buffer, handle->read_buffer, handle->chip.page_size);
    return ESP_OK;
}

static esp_err_t dhara_write(spi_nand_flash_device_t *handle, const uint8_t *buffer, dhara_sector_t sector_id)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = (spi_nand_flash_dhara_priv_data_t *)handle->ops_priv_data;
    dhara_error_t err;
    if (dhara_map_write(&dhara_priv_data->dhara_map, sector_id, buffer, &err)) {
        return ESP_ERR_FLASH_BASE + err;
    }
    return ESP_OK;
}

static esp_err_t dhara_copy_sector(spi_nand_flash_device_t *handle, dhara_sector_t src_sec, dhara_sector_t dst_sec)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = (spi_nand_flash_dhara_priv_data_t *)handle->ops_priv_data;
    dhara_error_t err;
    if (dhara_map_copy_sector(&dhara_priv_data->dhara_map, src_sec, dst_sec, &err)) {
        return ESP_ERR_FLASH_BASE + err;
    }
    return ESP_OK;
}

static esp_err_t dhara_trim(spi_nand_flash_device_t *handle, dhara_sector_t sector_id)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = (spi_nand_flash_dhara_priv_data_t *)handle->ops_priv_data;
    dhara_error_t err;
    if (dhara_map_trim(&dhara_priv_data->dhara_map, sector_id, &err)) {
        return ESP_ERR_FLASH_BASE + err;
    }
    return ESP_OK;
}

static esp_err_t dhara_sync(spi_nand_flash_device_t *handle)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = (spi_nand_flash_dhara_priv_data_t *)handle->ops_priv_data;
    dhara_error_t err;
    if (dhara_map_sync(&dhara_priv_data->dhara_map, &err)) {
        return ESP_ERR_FLASH_BASE + err;
    }
    return ESP_OK;
}

static esp_err_t dhara_get_capacity(spi_nand_flash_device_t *handle, dhara_sector_t *number_of_sectors)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = (spi_nand_flash_dhara_priv_data_t *)handle->ops_priv_data;
    *number_of_sectors = dhara_map_capacity(&dhara_priv_data->dhara_map);
    return ESP_OK;
}

static esp_err_t dhara_erase_chip(spi_nand_flash_device_t *handle)
{
    return nand_erase_chip(handle);
}

static esp_err_t dhara_erase_block(spi_nand_flash_device_t *handle, uint32_t block)
{
    return nand_erase_block(handle, block);
}


const spi_nand_ops dhara_nand_ops = {
    .init = &dhara_init,
    .deinit = &dhara_deinit,
    .read = &dhara_read,
    .write = &dhara_write,
    .erase_chip = &dhara_erase_chip,
    .erase_block = &dhara_erase_block,
    .trim = &dhara_trim,
    .sync = &dhara_sync,
    .copy_sector = &dhara_copy_sector,
    .get_capacity = &dhara_get_capacity,
};

esp_err_t nand_register_dev(spi_nand_flash_device_t *handle)
{
    if (handle == NULL) {
        return ESP_ERR_INVALID_ARG;
    }
    handle->ops = &dhara_nand_ops;
    return ESP_OK;
}

esp_err_t nand_unregister_dev(spi_nand_flash_device_t *handle)
{
    free(handle->ops_priv_data);
    handle->ops = NULL;
    return ESP_OK;
}

/*------------------------------------------------------------------------------------------------------*/


// The following APIs are implementations required by the Dhara library.
// Please refer to the header file dhara/nand.h for details.

int dhara_nand_is_bad(const struct dhara_nand *n, dhara_block_t b)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = __containerof(n, spi_nand_flash_dhara_priv_data_t, dhara_nand);
    spi_nand_flash_device_t *dev_handle = dhara_priv_data->parent_handle;
    bool is_bad_status = false;
    if (nand_is_bad(dev_handle, b, &is_bad_status)) {
        return 1;
    }
    if (is_bad_status == true) {
        return 1;
    }
    return 0;
}

void dhara_nand_mark_bad(const struct dhara_nand *n, dhara_block_t b)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = __containerof(n, spi_nand_flash_dhara_priv_data_t, dhara_nand);
    spi_nand_flash_device_t *dev_handle = dhara_priv_data->parent_handle;
    nand_mark_bad(dev_handle, b);
    return;
}

int dhara_nand_erase(const struct dhara_nand *n, dhara_block_t b, dhara_error_t *err)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = __containerof(n, spi_nand_flash_dhara_priv_data_t, dhara_nand);
    spi_nand_flash_device_t *dev_handle = dhara_priv_data->parent_handle;
    esp_err_t ret = nand_erase_block(dev_handle, b);
    if (ret) {
        if (ret == ESP_ERR_NOT_FINISHED) {
            dhara_set_error(err, DHARA_E_BAD_BLOCK);
        }
        return -1;
    }
    return 0;
}

int dhara_nand_prog(const struct dhara_nand *n, dhara_page_t p, const uint8_t *data, dhara_error_t *err)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = __containerof(n, spi_nand_flash_dhara_priv_data_t, dhara_nand);
    spi_nand_flash_device_t *dev_handle = dhara_priv_data->parent_handle;
    esp_err_t ret = nand_prog(dev_handle, p, data);
    if (ret) {
        if (ret == ESP_ERR_NOT_FINISHED) {
            dhara_set_error(err, DHARA_E_BAD_BLOCK);
        }
        return -1;
    }
    return 0;
}

int dhara_nand_is_free(const struct dhara_nand *n, dhara_page_t p)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = __containerof(n, spi_nand_flash_dhara_priv_data_t, dhara_nand);
    spi_nand_flash_device_t *dev_handle = dhara_priv_data->parent_handle;
    bool is_free_status = true;
    if (nand_is_free(dev_handle, p, &is_free_status)) {
        return 0;
    }
    if (is_free_status == true) {
        return 1;
    }
    return 0;
}

int dhara_nand_read(const struct dhara_nand *n, dhara_page_t p, size_t offset, size_t length,
                    uint8_t *data, dhara_error_t *err)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = __containerof(n, spi_nand_flash_dhara_priv_data_t, dhara_nand);
    spi_nand_flash_device_t *dev_handle = dhara_priv_data->parent_handle;
    if (nand_read(dev_handle, p, offset, length, data)) {
        if (dev_handle->chip.ecc_data.ecc_corrected_bits_status == STAT_ECC_NOT_CORRECTED) {
            dhara_set_error(err, DHARA_E_ECC);
        }
        return -1;
    }
    return 0;
}

int dhara_nand_copy(const struct dhara_nand *n, dhara_page_t src, dhara_page_t dst, dhara_error_t *err)
{
    spi_nand_flash_dhara_priv_data_t *dhara_priv_data = __containerof(n, spi_nand_flash_dhara_priv_data_t, dhara_nand);
    spi_nand_flash_device_t *dev_handle = dhara_priv_data->parent_handle;
    esp_err_t ret = nand_copy(dev_handle, src, dst);
    if (ret) {
        if (dev_handle->chip.ecc_data.ecc_corrected_bits_status == STAT_ECC_NOT_CORRECTED) {
            dhara_set_error(err, DHARA_E_ECC);
        }
        if (ret == ESP_ERR_NOT_FINISHED) {
            dhara_set_error(err, DHARA_E_BAD_BLOCK);
        }
        return -1;
    }
    return 0;
}
```

### Dhara 存在的问题

- [Performance Tuning](https://github.com/dlbeer/dhara/issues/21)
    相比较于直接读写 flash 的速率, 通过 Dhara 读写速率下降厉害，而且读取速率比写入速率慢

在这个case 上面的测试数据
```bash
# nand_write_read

..............................
Write 3840.0 KB at 986.988 KB/s
..............................
Read 3840.0 KB at 1998.049 KB/s
Done. Marked 0 this run, 0 total

# dhara_write_read

................................
Write 4000.0 KB at 620.230 KB/s
................................
Read 4000.0 KB at 271.348 KB/s
# 
```

自己测试的数据
```bash
# 直接读写flash
msh />flashspeed 0 102400 4096

flash write speed: 1019000 byte/s
total_length: 419430400 bytes, cost: 411 s

flash read speed: 1365000 byte/s
total_length: 419430400 bytes, cost: 307 s

# 经过Dhara，挂载 fat32 文件系统读写文件速率

msh />writespeed test 104857600 4096
File write speed: 748000 byte/s
msh />readspeed test  4096
File read speed: 305000 byte/s

```



## 参考资料

- [SPI NAND Flash Driver](https://github.com/espressif/idf-extra-components/tree/57c8917e0204f0058863b3bc67517183cd8ae71e/spi_nand_flash)
- [AllianceMemory_SPI_NAND_Flash_July2020_Rev1_0-1893515.pdf](https://www.mouser.com/datasheet/2/12/AllianceMemory_SPI_NAND_Flash_July2020_Rev1_0-1893515.pdf)
- [Dhara: NAND flash translation layer for small MCUs](https://github.com/dlbeer/dhara)
