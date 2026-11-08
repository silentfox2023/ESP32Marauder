<!---[![License: MIT](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/justcallmekoko/ESP32Marauder/blob/master/LICENSE)--->
<!---[![Gitter](https://badges.gitter.im/justcallmekoko/ESP32Marauder.png)](https://gitter.im/justcallmekoko/ESP32Marauder)--->
<!---[![Build Status](https://travis-ci.com/justcallmekoko/ESP32Marauder.svg?branch=master)](https://travis-ci.com/justcallmekoko/ESP32Marauder)--->
<!---Shields/Badges https://shields.io/--->

# ESP32 Marauder
<p align="center"><img alt="Marauder logo" src="https://github.com/justcallmekoko/ESP32Marauder/blob/master/pictures/marauder_skull_patch_04_full_final.png?raw=true" width="300"></p>
<p align="center">
  <b>A suite of WiFi/Bluetooth offensive and defensive tools for the ESP32</b>
  <br><br>
  <a href="https://github.com/justcallmekoko/ESP32Marauder/blob/master/LICENSE"><img alt="License" src="https://img.shields.io/github/license/mashape/apistatus.svg"></a>
  <a href="https://gitter.im/justcallmekoko/ESP32Marauder"><img alt="Gitter" src="https://badges.gitter.im/justcallmekoko/ESP32Marauder.png"/></a>
  <br>
  <a href="https://twitter.com/intent/follow?screen_name=jcmkyoutube"><img src="https://img.shields.io/twitter/follow/jcmkyoutube?style=social&logo=twitter" alt="Twitter"></a>
  <a href="https://www.instagram.com/just.call.me.koko"><img src="https://img.shields.io/badge/Follow%20Me-Instagram-orange" alt="Instagram"/></a>
  <br><br>
</p>
    
[![Build and Push](https://github.com/justcallmekoko/ESP32Marauder/actions/workflows/build_push.yml/badge.svg)](https://github.com/justcallmekoko/ESP32Marauder/actions/workflows/build_push.yml)
[![Firmware coverage](https://codecov.io/gh/justcallmekoko/ESP32Marauder/branch/develop/graph/badge.svg?flag=firmware-unit-tests)](https://app.codecov.io/gh/justcallmekoko/ESP32Marauder/tree/develop)

## Getting Started
Download the [latest release](https://github.com/justcallmekoko/ESP32Marauder/releases/latest) of the firmware.  

Check out the project [wiki](https://github.com/justcallmekoko/ESP32Marauder/wiki) for a full overview of the ESP32 Marauder

## 板型适配（XiaoMiao / ESP32-C5 Mini V3 / Marauder V8 官方版 / Marauder V8 修改版）

本仓库基于官方 ESP32 Marauder 代码，合入了四套已验证的自制/改版硬件适配。四套板型共用同一份 `esp32_marauder/configs.h`，通过板型宏切换，**同一时间只能启用一套**。

另外还有两套只参考引脚、不合并代码的工程（`marauderC5` 1.18.0、`ESP32_LDDB` 1.19.0），见第 5 节。

> 说明：本仓库代码基线为官方 v1.12.1；各板型参数来自对应的 v1.20.0 / 1.18.0 / 1.19.0 参考工程，并按"板型适配最小差异"移植回本仓库。

### 板型速查表

| 板型 | 板型宏 | 编译目标（arduino-cli / Arduino IDE） | MCU | 屏幕方案 | 分区方案 | 已验证固件体积 |
|---|---|---|---|---|---|---|
| XiaoMiao（小喵掌机） | `MARAUDER_XIAOMIAO`（继承 `MARAUDER_MINI`） | `esp32:esp32:esp32:PartitionScheme=huge_app` | ESP32（WROOM-32） | ST7735 1.8"，128×160 面板 → 应用层 160×128 横屏，硬件 VSPI 10MHz | `huge_app` | 2,022,031 bytes（64%） |
| ESP32-C5 Mini V3 | `MARAUDER_MINI_V3` + `DUAL_MINI_C5` | `esp32:esp32:esp32c5:PartitionScheme=huge_app` | ESP32-C5N16R8 | ST7735 128×128 竖屏，bit-bang SPI 写 4MHz / 读 2MHz | `huge_app` | 2,277,234 bytes（72%） |
| Marauder V8 官方版（4in1，硬件 SPI） | `MARAUDER_V8` | `esp32:esp32:esp32c5:PartitionScheme=huge_app,FlashSize=8M` | ESP32-C5（固件未启用 PSRAM） | ILI9341 240×320 + XPT2046，TFT/SD 走 SPI2 原生 IOMUX | `huge_app` | 约 2,312,816 bytes（73%） |
| Marauder V8 修改版 | `MARAUDER_V8_MODIFIED` | `esp32:esp32:esp32c5:PartitionScheme=huge_app,FlashSize=16M` | ESP32-C5 N16R8（固件未启用 PSRAM） | ILI9341 240×320 + XPT2046，TFT 引脚经 GPIO 矩阵映射 | `huge_app` | 约 2,312,744 bytes（73%） |

> 四套板型都必须使用 `huge_app`（3MB No OTA）分区。Marauder 固件本身已超过默认 APP 分区，C5 构建尤其需要。
>
> 体积说明：表中体积为对应参考工程镜像的实测值；两套 V8 数值由整片镜像中 app 分区解析得出（24 字节镜像头 + 各段，不含末尾校验与 SHA-256），非编译日志。参考镜像分别为 8MB / 16MB 整片，故 `FlashSize` 建议值不同。本仓库 v1.12.1 基线的 XiaoMiao 构建约 1.73MB，属正常范围。

### 通用前提

1. Arduino ESP32 core ≥ 3.3.4（ESP-IDF 5.5），C5 板型必须；
2. 使用支持 ESP32-C5 的专用 TFT_eSPI（含 `Processors/TFT_eSPI_ESP32_C5.h/.c` 与 C5 触摸补丁）；
3. **TFT_eSPI 的 `User_Setup.h` 不在本仓库内**。Arduino IDE 实际编译的是全局库里的那一份（本机为 `j:\Documents\Arduino\libraries\TFT_eSPI\User_Setup.h`），仓库内同名文件不参与编译。每次切换板型都必须手工同步这一份文件。

### ⚠️ 必读：板型继承宏必须前置

自定义板型如果依赖 `MARAUDER_MINI` 的既有配置（目前只有 XiaoMiao），必须在 `configs.h` 的 `//// BOARD TARGETS` 区**紧跟着**定义：

```c
#if defined(MARAUDER_XIAOMIAO) && !defined(MARAUDER_MINI)
  #define MARAUDER_MINI
#endif
```

不要把 `#define MARAUDER_MINI` 放到文件末尾或单独的 `*_config.h` 里。原因：`configs.h` 前部的 `#ifdef MARAUDER_MINI` 块（`HAS_SCREEN`、显示驱动、按键、SD 等）在预处理器第一次扫描时就已经展开，之后才定义的宏不会回溯生效。

典型症状：

- 串口输出正常、WiFi/CLI 可用，但**屏幕全白**；
- 固件体积明显变小（Display / 菜单代码被 `#ifdef HAS_SCREEN` 整体剔除）。

这条经验在 XiaoMiao 移植时踩过，修复方式就是把继承宏挪到 `BOARD TARGETS` 之后立即定义。**V8 官方版与修改版都是独立板型，不继承 `MARAUDER_MINI`，不受这一条影响。** 速判：白屏且固件体积明显小于同版本正常构建（v1.20.0 约 2.0MB；本仓库 v1.12.1 约 1.7MB）时，优先检查这一条。

### 1. XiaoMiao（经典 ESP32）

- 启用：`#define MARAUDER_XIAOMIAO`，并在其后立即 `#define MARAUDER_MINI`（见上文）
- 板型头文件：`esp32_marauder/xiaomiao_config.h`，在 `configs.h` 末尾 `#include`，用 `#undef` + `#define` 覆盖引脚与屏幕布局
- 编译目标：`esp32:esp32:esp32:PartitionScheme=huge_app`

| 功能 | 引脚 |
|---|---|
| TFT SCLK | 18 |
| TFT MOSI | 23 |
| TFT MISO | -1（ST7735 不回读） |
| TFT CS | 5 |
| TFT DC | 4 |
| TFT RST | -1（使用 ST7735 上电复位） |
| TFT BL | 背光直连电源，软件不控制（User_Setup 里的 TFT_BL=32 为空脚） |
| SD SCK / MOSI / MISO / CS | 18 / 23 / 19 / 22（与 TFT 共享 VSPI） |
| 按键 上 / 下 / 左 / 右 | 2 / 13 / 27 / 35 |
| 按键 A（确认） / B（返回） | 34 / 12 |
| NeoPixel | 25（继承 Mini） |

- 屏幕：ST7735S，1.8"，面板分辨率 128×160；`SCREEN_ORIENTATION=3`，应用层按 `DISP_W=160`、`DISP_H=128` 横屏布局
- 按键注意：GPIO34/35 为输入-only，无内部上拉，依赖 PCB 外部上拉电阻；`Switches.cpp` 对 34–39 自动使用纯 `INPUT`
- TFT_eSPI `User_Setup.h` 关键项：

```c
#define ST7735_DRIVER
#define ST7735_GREENTAB       // rotation=3 时 rowstart 最大，可避免上边缘雪花
#define TFT_INVERSION_OFF
#define TFT_RGB_ORDER TFT_BGR
#define TFT_MISO -1
#define TFT_MOSI 23
#define TFT_SCLK 18
#define TFT_CS   5
#define TFT_DC   4
#define TFT_RST  -1
#define TFT_BL   32
#define TFT_WIDTH  128
#define TFT_HEIGHT 160
#define SPI_FREQUENCY 10000000
```

### 2. ESP32-C5 Mini V3

- 启用：`#define MARAUDER_MINI_V3` + `#define DUAL_MINI_C5`
- 编译目标：`esp32:esp32:esp32c5:PartitionScheme=huge_app`
- 屏幕：ST7735，128×128 竖屏，`SCREEN_ORIENTATION=0`。C5 走 **bit-bang SPI**（`TFT_eSPI_ESP32_C5.c`），不能使用硬件 SPI 端口，SPI 频率必须压低

| 功能 | 引脚 |
|---|---|
| TFT SCLK | 26 |
| TFT MOSI | 0 |
| TFT MISO | 24 |
| TFT CS | 25 |
| TFT DC | 23 |
| TFT RST | 1 |
| TFT BL | 3 |
| TOUCH_CS | -1（无触摸） |
| SD SCK / MOSI / MISO / CS | 26 / 0 / 24 / 4（与 TFT 共享总线） |
| SD_DET | 5（仅 `DUAL_MINI_C5`） |
| 按键 L / C / U / R / D | `DUAL_MINI_C5`：6 / 9 / 7 / 8 / 10；基础 Mini V3：10 / 9 / 6 / 7 / 8 |
| GPS TX / RX | 14 / 13（Serial1） |

- TFT_eSPI `User_Setup.h` 关键项：

```c
#define ST7735_DRIVER
#define ST7735_GREENTAB3
#undef  USE_HSPI_PORT       // C5 使用 bit-bang，不走硬件 SPI 端口
#undef  USE_FSPI_PORT
#define TFT_MISO 24
#define TFT_MOSI 0
#define TFT_SCLK 26
#define TFT_CS   25
#define TFT_DC   23
#define TFT_RST  1
#define TFT_BL   3
#define TFT_WIDTH  128
#define TFT_HEIGHT 128
#define SPI_FREQUENCY        4000000   // 4MHz 写
#define SPI_READ_FREQUENCY   2000000   // 2MHz 读
```

### 3. Marauder V8 官方版（4in1，硬件 SPI）

- 启用：`#define MARAUDER_V8`
- 编译目标：`esp32:esp32:esp32c5:PartitionScheme=huge_app,FlashSize=8M`（参考工程整片镜像为 8MB；分区表为 app0 3MB + spiffs 0xE0000 + coredump，与 `huge_app` 完全吻合）
- 屏幕：ILI9341 2.8" 240×320 + XPT2046 电阻触摸，TFT / 触摸 / SD 共用一条 SPI 总线
- 硬件 SPI：TFT/SD 使用 ESP32-C5 的 **SPI2 原生 IOMUX** 引脚（MISO=2 / SCLK=6 / MOSI=7）；`TFT_CS`、`TOUCH_CS`、`SD_CS` 由普通 GPIO 控制
- PSRAM：参考工程 `configs.h` 的 V8 块里 `HAS_PSRAM` 是注释状态，**固件不使用 PSRAM**。该工程没有 `sketch.yaml` 或构建记录，无法确认 FQBN 是否另开了 PSRAM 选项，也无法确认 QSPI/OPI；其整片镜像为 8MB，工程内没有模组型号标注（注意：带 `WithPSRAM8M` 的是第 4 节的修改版工程）
- `HAS_ACT_LED` 未启用
- "4in1" 是硬件模块命名，参考工程没有运行期多板型选择菜单，固件仍是单一 `MARAUDER_V8` 目标
- 固件体积：约 2,312,816 bytes（app 分区，约占 3MB 的 73%，由 8MB 整片镜像解析）

| 功能 | 引脚 |
|---|---|
| TFT MISO | 2 |
| TFT MOSI | 7 |
| TFT SCLK | 6 |
| TFT CS | 23 |
| TFT DC | 24 |
| TFT RST | 0（参考工程在 `TFT_eSPI/User_Setup.h` 中定义；本仓库同时在 `configs.h` 中保留同名定义） |
| TOUCH_CS | 3 |
| TFT BL | 8 |
| SD CS | 10（与 TFT 共享 MISO 2 / MOSI 7 / SCLK 6） |
| I2C SDA / SCL | 5 / 4（电池电量计） |
| GPS TX / RX | 13 / 14（Serial1，**写反即 GPS Not Found**） |

- TFT_eSPI `User_Setup.h` 关键项（参考工程自带 `TFT_eSPI/User_Setup.h`）：

```c
#define ILI9341_DRIVER
#define TFT_MISO 2
#define TFT_MOSI 7
#define TFT_SCLK 6
#define TFT_CS   23
#define TFT_DC   24
#define TFT_RST  0
#define TOUCH_CS 3
// 背光不写进 User_Setup.h，由 configs.h 的 TFT_BL 8 控制
#define TFT_WIDTH  240
#define TFT_HEIGHT 320
#define SPI_FREQUENCY        10000000
#define SPI_READ_FREQUENCY   20000000
#define SPI_TOUCH_FREQUENCY   2500000
```

### 4. Marauder V8 修改版

- 启用：`#define MARAUDER_V8_MODIFIED`
- 编译目标：`esp32:esp32:esp32c5:PartitionScheme=huge_app,FlashSize=16M`（参考工程整片镜像为 16MB，分区表同为 `huge_app`）
- 与官方版的关系：**同一套固件代码，只有引脚不同**。实机比对两份工程的顶层源码，除 `configs.h` 的引脚块（约 23 行）外，其余 `.cpp/.h/.ino` 内容完全一致；因此本仓库用两个独立板型宏承载，互不影响，也不会自动继承对方
- 屏幕与触摸：与官方版相同（ILI9341 240×320 + XPT2046）；区别是 TFT 引脚经 **GPIO 矩阵**映射，不是 IOMUX 原生脚
- PSRAM：`HAS_PSRAM` 同样为注释状态，**固件不使用 PSRAM**
- 固件体积：约 2,312,744 bytes（app 分区，约占 3MB 的 73%，由 16MB 整片镜像解析）

| 功能 | 引脚 |
|---|---|
| TFT MISO | 5 |
| TFT MOSI | 1 |
| TFT SCLK | 9 |
| TFT CS | 24 |
| TFT DC | 10 |
| TFT RST | 0 |
| TOUCH_CS | 4 |
| TFT BL | 23 |
| SD CS | 6（与 TFT 共享 MISO 5 / MOSI 1 / SCLK 9） |
| I2C SDA / SCL | 8 / 26（电池电量计） |
| GPS TX / RX | 13 / 14（Serial1） |

- TFT_eSPI `User_Setup.h` 关键项（参考工程未自带库，使用全局库）：

```c
#define ILI9341_DRIVER
#define TFT_MISO 5
#define TFT_MOSI 1
#define TFT_SCLK 9
#define TFT_CS   24
#define TFT_DC   10
#define TFT_RST  0
#define TOUCH_CS 4
// 背光不写进 User_Setup.h，由 configs.h 的 TFT_BL 23 控制
#define TFT_WIDTH  240
#define TFT_HEIGHT 320
#define SPI_FREQUENCY        10000000   // 稳定值；可再试 27MHz
#define SPI_READ_FREQUENCY   20000000
#define SPI_TOUCH_FREQUENCY   2500000
// 不要打开 USE_HSPI_PORT
```

> 两个 V8 宏的引脚完全不同：官方版 2/7/6/23/24/3/8/10，修改版 5/1/9/24/10/4/23/6。混用会白屏或 SD 挂载失败。选宏前先确认手上板子的丝印 / 走线属于哪一版。

- V8 两版共同的代码级要点（已在本仓库实现）：
  - `esp32_marauder.ino` 的 SD 对象必须直接用全局 `SPI`（`SDInterface(&SPI, SD_CS)`）。**不要**写 `SPIClass sharedSPI(SPI)` 拷贝构造，否则会破坏总线仲裁，表现为白屏；
  - `SDInterface::initSD()` 的 `HAS_C5_SD` 分支先执行 `_spi->begin(SD_SCK, SD_MISO, SD_MOSI, -1)` 再 `SD.begin()`；
  - C5 启动后把 CPU 提升到 240MHz，以保证 TFT 传输速度。

### 5. 仅参考引脚的两套工程（代码未合并）

以下两套参考工程与本仓库基线**版本不同**，只提取引脚定义供接线参考，**没有合并任何代码**。直接照抄配置会有版本差异风险（宏、函数、菜单结构都可能不存在或已变化）。

| 参考工程 | 版本 / 板型宏 | 与本仓库基线的关系 | 说明 |
|---|---|---|---|
| `esp32_marauder_3in1_1.18.0-modified-ESP32C5_marauderC5` | 1.18.0 / `MARAUDER_C5` | 基线 v1.12.1，版本不同 | ESP32-C5；无屏幕（`HAS_SCREEN` 未启用）、无 GPS、无电池 |
| `esp32_marauder_3in1_1.19.0-modified_ESP32_LDDB-1.19.0` | 1.19.0 / `ESP32_LDDB` | 基线 v1.12.1，版本不同 | 经典 ESP32；无屏幕、无 GPS、无电池 |

`marauderC5`（1.18.0）已定义的引脚：

| 功能 | 引脚 |
|---|---|
| SD CS | 10 |
| SD MISO / MOSI / SCK | 2 / 7 / 6（SPI2 原生 IOMUX） |
| GPS TX / RX | 14 / 13（Serial1） |
| NeoPixel | 27（`HAS_NEOPIXEL_LED` 未启用） |

`ESP32_LDDB`（1.19.0）已定义的引脚：

| 功能 | 引脚 |
|---|---|
| SD CS | 4（需用 ESP-IDF 底层 API 释放 flash pad） |
| SD MISO / MOSI / SCK | 17 / 4 / 16 |
| NeoPixel | 17（默认与 SD MISO 冲突，`HAS_NEOPIXEL_LED` 未启用） |

> 注意：`ESP32_LDDB` 工程里 `SD_CS` 与 `SD_MOSI` 都写作 GPIO4，按定义存在同脚冲突；本仓库只如实转述该工程的引脚，实际接线请以硬件为准。

> 这两套工程的屏幕 / 触摸 / 电池 / GPS 均未启用，因此没有对应的 TFT 引脚可参考；不要从它们的 `User_Setup.h` 反推屏幕接线。

### TFT_eSPI `User_Setup.h` 同步清单

Arduino IDE 编译时只认全局库文件，切换板型必须逐项改完再**完整重编译**：

| 配置项 | XiaoMiao | ESP32-C5 Mini V3 | V8 官方版 | V8 修改版 |
|---|---|---|---|---|
| 驱动 / tab | `ST7735_DRIVER` + `ST7735_GREENTAB` | `ST7735_DRIVER` + `ST7735_GREENTAB3` | `ILI9341_DRIVER` | `ILI9341_DRIVER` |
| 引脚（MISO/MOSI/SCLK/CS/DC） | -1 / 23 / 18 / 5 / 4 | 24 / 0 / 26 / 25 / 23 | 2 / 7 / 6 / 23 / 24 | 5 / 1 / 9 / 24 / 10 |
| 复位 / 背光 | RST=-1，BL=32（空脚） | RST=1，BL=3 | RST=0，BL=8（`User_Setup.h` 不定义 BL） | RST=0，BL=23（`User_Setup.h` 不定义 BL） |
| 分辨率 | 128×160 | 128×128 | 240×320 | 240×320 |
| SPI 频率 | 10MHz | 4MHz 写 / 2MHz 读 | 10MHz 写 / 20MHz 读，触摸 2.5MHz | 10MHz 写（可试 27MHz）/ 20MHz 读，触摸 2.5MHz |
| 其他 | `TFT_INVERSION_OFF`、`TFT_RGB_ORDER TFT_BGR` | bit-bang，禁止 `USE_HSPI_PORT` / `USE_FSPI_PORT` | `TOUCH_CS`=3，不开 `USE_HSPI_PORT` | `TOUCH_CS`=4，不开 `USE_HSPI_PORT` |

### 切换板型步骤

1. 在 `esp32_marauder/configs.h` 的 `//// BOARD TARGETS` 区注释掉当前板型，启用目标板型宏；两套 V8 宏 `MARAUDER_V8` / `MARAUDER_V8_MODIFIED` 互斥，只启用其中一个；
2. 若目标板型是继承型（如 XiaoMiao），**紧接着**定义 `MARAUDER_MINI`（见上文"板型继承宏必须前置"）；V8 不继承 `MARAUDER_MINI`；
3. 同步修改全局 TFT_eSPI `User_Setup.h`：驱动、引脚、分辨率、SPI 频率、tab 变体；
4. 切换编译目标：C5 板型用 `esp32:esp32:esp32c5`，经典 ESP32 用 `esp32:esp32:esp32`；分区统一 `PartitionScheme=huge_app`；
5. 完整重编译（不要依赖增量缓存），确认固件体积与同版本正常构建一致、Display 代码已编入；
6. 烧录后检查：屏幕点亮、按键方向正确、SD 可挂载、GPS（若接线）可定位。

### 新增命令与功能（gpioscan / tftbench / touchraw）

> **来源与版本**：这三条命令属于 **1.20.0 系列固件**。用法依据是本机在用的 1.20.0 工程
> `J:\Documents\Arduino\esp32marauder_1.20.0` 的 `CommandLine.h` / `CommandLine.cpp`。
> 两套 v1.20.0 V8 参考工程（官方版 / 修改版）也包含 `tftbench` 和 `touchraw`，但其中的 `gpioscan` 只有"无参数全扫描"一种用法。
>
> **本仓库基线 v1.12.1 没有合并这些命令**，1.18.0 的 `marauderC5`、1.19.0 的 `ESP32_LDDB` 也没有。在 `help` / 菜单里看不到它们属于预期行为；需要时请到 1.20.0 工程使用或自行移植。

| 命令 | 参数 | 适用构建 | 实际行为 |
|---|---|---|---|
| `gpioscan` | 无 | 全部 | 全脚扫描：`0 … NUM_DIGITAL_PINS-1`，逐个打印 `GPIO<n> : RESERVED (原因)`、`GPIO<n> : IN USE [固件占用]`、`GPIO<n> : HIGH|LOW (电量)` 三种之一；结尾 `Scan complete: N GPIOs, M in use, K reserved` + `Floating (unconnected) pins may read HIGH or LOW.` |
| `gpioscan -p <pin>` | 单引脚 | 全部 | 只查一个引脚：`[GPIO] GPIO<n>  level=<0|1>  mode=<input|output|input-only|flash-reserved>`；经典 ESP32 的 flash 脚改打印 `level=n/a  mode=flash-reserved (skipped)` |
| `gpioscan -p <pin> -l <0\|1>` | 输出电平 | 全部 | 先从 LEDC 摘除（`ledcDetach`）再设为 `OUTPUT` 并写入电平，打印 `[GPIO] pin <n> -> OUTPUT level <0|1> (held)`；命令返回后引脚**保持该电平**，直到重启或再次操作 |
| `gpioscan -p <pin> -t <sec>` | 翻转测试 | 全部 | 打印 `[GPIO] pin <n> toggling 1Hz for <sec> s`，每 500ms 翻转并打印 `[GPIO] pin <n> = 0|1`，持续 `sec` 秒（1–600），结束拉低并打印 `[GPIO] toggle finished, pin left at level 0` |
| `tftbench` | 无 | `HAS_SCREEN` + ESP32-C5 | 执行一次 `fillScreen(TFT_BLACK)` 并计时，打印 `tftbench: <us> us for <px> px, <KB/s> KB/s, CPU <MHz> MHz`；非 C5 屏幕构建只打印 `tftbench requires the ESP32-C5 screen build` |
| `touchraw` | 无 | `HAS_TOUCH` + ESP32-C5 | 用 4 条 XPT2046 命令（`X=0xD3`、`Y=0x93`、`Z1=0xB3`、`Z2=0xC3`）各读一次，打印 `raw` 3 字节与 12 位 `value`；非 C5 触摸构建只打印 `touchraw requires the ESP32-C5 touch build` |

- 参数约束：`-l` / `-t` 必须配合 `-p`；`-p` 的范围是 `0 … SOC_GPIO_PIN_COUNT-1`；`-l` 只接受 `0`/`1`；`-t` 只接受 `1–600` 秒；`-l` 与 `-t` 同时给出时按 `-t` 执行（源码里 `-t` 分支先返回）。参数顺序不限，非法值会打印 `[GPIO] invalid pin ...`、`[GPIO] -l requires 0 or 1` 等提示。
- 保留脚判定按目标芯片内置：ESP32-C5 为 GPIO16–18 / 20–22（flash）和 GPIO11–12（UART0）；经典 ESP32 为 GPIO6–11（flash）和 GPIO1/3（UART0）。全扫描里这些脚显示 `RESERVED (原因)`。
- 写保护范围：`-l` / `-t` 只对**经典 ESP32** 生效——GPIO6–11 是 SPI Flash 脚、GPIO34–39 只能输入，两者都会被拒绝驱动并提示 `reserved for SPI flash` / `input-only`。ESP32-C5 上这两个保护函数直接返回 false，因此不做该拦截。
- `mode` 字段（`input` / `output` / `input-only` / `flash-reserved`）只在经典 ESP32 上真实有效；其他目标（含 C5）的方向寄存器布局不同，`-p` 查询一律输出 `mode=input`。
- `gpioscan` 的 `IN USE` 标签来自当前编译的板型宏：`TFT_MOSI/MISO/SCLK/CS/DC/RST/BL`、`SD_CS/SCK/MISO/MOSI/DET`、`GPS_TX/RX`（需 `HAS_GPS`）、`I2C_SDA/SCL`（需 `HAS_BATTERY`）、`ACT_LED`、`BATTERY_ADC`、`NEOPIXEL`（需 `HAS_NEOPIXEL_LED`）。换板型后输出自动跟着变。注意 `TOUCH_CS` 不在该标签表里，触摸引脚只会显示普通电平。
- `touchraw` 的 `value` 是 `((raw[1] << 8) | raw[2]) >> 3`，即 XPT2046 的 12 位结果；左右两侧的 `CS_H` / `T_CS_L` 由 TFT_eSPI 的 C5 补丁提供。
- 单引脚查询的 flash 跳过逻辑只在经典 ESP32 上触发；C5 的保留脚只在"无参数全扫描"里标注，用 `-p` 查询时仍会打印电平。
- `tftbench` / `touchraw` 依赖 C5 专用 TFT_eSPI 补丁（`tft_c5_touch_dump` 等）；换回官方原版 TFT_eSPI 后这两个命令不会被编入。`gpioscan` 只用 Arduino 基础 API，不依赖 TFT_eSPI。

### 板型适配文档

- [XiaoMiao 编译与配置说明书](docs/board_adapters/XiaoMiao_编译与配置说明书.md)
- [ESP32-C5 Mini V3 编译与配置说明书](docs/board_adapters/ESP32C5_Mini_V3_编译与配置说明书.md)
- [C5 V8 移植与修改说明书](docs/board_adapters/C5_V8_移植与修改说明书.md)（同时覆盖 V8 官方版与修改版）

# For Sale Now
You can buy the ESP32 Marauder using [this link](https://www.justcallmekokollc.com)
