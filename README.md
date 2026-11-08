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

## 板型适配（XiaoMiao / ESP32-C5 Mini V3 / Marauder V8）

本仓库基于官方 ESP32 Marauder 代码，合入了三套已验证的自制/改版硬件适配。三套板型共用同一份 `esp32_marauder/configs.h`，通过板型宏切换，**同一时间只能启用一套**。

> 说明：本仓库代码基线为官方 v1.12.1；三套板型的参数已在 v1.20.0 工程上实机验证，并按"板型适配最小差异"移植回本仓库。

### 板型速查表

| 板型 | 板型宏 | 编译目标（arduino-cli / Arduino IDE） | MCU | 屏幕方案 | 分区方案 | 已验证固件体积 |
|---|---|---|---|---|---|---|
| XiaoMiao（小喵掌机） | `MARAUDER_XIAOMIAO`（继承 `MARAUDER_MINI`） | `esp32:esp32:esp32:PartitionScheme=huge_app` | ESP32（WROOM-32） | ST7735 1.8"，128×160 面板 → 应用层 160×128 横屏，硬件 VSPI 10MHz | `huge_app` | 2,022,031 bytes（64%） |
| ESP32-C5 Mini V3 | `MARAUDER_MINI_V3` + `DUAL_MINI_C5` | `esp32:esp32:esp32c5:PartitionScheme=huge_app` | ESP32-C5N16R8 | ST7735 128×128 竖屏，bit-bang SPI 写 4MHz / 读 2MHz | `huge_app` | 2,277,234 bytes（72%） |
| Marauder V8（4in1） | `MARAUDER_V8` | `esp32:esp32:esp32c5:PartitionScheme=huge_app` | ESP32-C5（N16R8 模组，固件未启用 PSRAM） | ILI9341 240×320 + XPT2046 触摸，共享 SPI | `huge_app` | 2,312,788 bytes（73%） |

> 三套板型都必须使用 `huge_app`（3MB No OTA）分区。Marauder 固件本身已超过默认 APP 分区，Mini V3 / V8 的 C5 构建尤其需要。
>
> 体积说明：表中体积为 v1.20.0 工程实测值；V8 数值由参考工程 16MB 整片镜像中 app 分区解析得出（非编译日志）。本仓库 v1.12.1 基线的 XiaoMiao 构建约 1.73MB，属正常范围。

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

V8 与 XiaoMiao 移植时都踩过这个坑，修复方式就是把继承宏挪到 `BOARD TARGETS` 之后立即定义。速判：白屏且固件体积明显小于同版本正常构建（v1.20.0 约 2.0MB；本仓库 v1.12.1 约 1.7MB）时，优先检查这一条。

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

### 3. Marauder V8（ESP32-C5，4in1）

- 启用：`#define MARAUDER_V8`
- 编译目标：`esp32:esp32:esp32c5:PartitionScheme=huge_app`（参考工程为 16MB flash；其整片镜像中的分区表为 app0 3MB + spiffs 0xE0000 + coredump，与 `huge_app` 完全吻合）
- 屏幕：ILI9341 2.8" 240×320 + XPT2046 电阻触摸，TFT / 触摸 / SD 共用一条 SPI 总线（SCLK=9 / MOSI=1 / MISO=5）
- PSRAM：参考工程目录名带 `WithPSRAM8M`、模组为 N16R8，但 `configs.h` 的 V8 块里 `HAS_PSRAM` 仍是注释状态，**固件不使用 PSRAM**（`WiFiScan` / `EvilPortal` 走 `malloc` 兜底）。参考工程没有 `sketch.yaml` 或构建记录，无法确认 FQBN 是否另开了 PSRAM 选项，也无法确认 QSPI/OPI
- `HAS_ACT_LED` 未启用（`ACT_LED_PIN` 有定义但宏被注释）
- "4in1" 是硬件模块命名，参考工程没有运行期多板型选择菜单，固件仍是单一 `MARAUDER_V8` 目标
- 固件体积：约 2,312,788 bytes（app 分区，约占 3MB 的 73%，由参考工程 16MB 整片镜像解析）

| 功能 | 引脚 |
|---|---|
| TFT MISO | 5 |
| TFT MOSI | 1 |
| TFT SCLK | 9 |
| TFT CS | 24 |
| TFT DC | 10 |
| TFT RST | 0（`configs.h` 与 `User_Setup.h` 都定义为 0） |
| TOUCH_CS | 4 |
| TFT BL | 23 |
| SD CS | 6（与 TFT 共享 MISO 5 / MOSI 1 / SCLK 9） |
| I2C SDA / SCL | 8 / 26（电池电量计） |
| GPS TX / RX | 13 / 14（Serial1，**写反即 GPS Not Found**） |

> 本仓库与参考工程 `configs.h` 的 `MARAUDER_V8` 块完全一致：MISO=5 / MOSI=1 / SCLK=9 / CS=24 / DC=10 / RST=0 / TOUCH_CS=4 / BL=23 / SD_CS=6 / I2C=8/26 / GPS=13/14。不存在第二套并行的引脚数字。

- TFT_eSPI `User_Setup.h` 关键项：

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
#define SPI_TOUCH_FREQUENCY  2500000
// 参考工程未记录 SPI_READ_FREQUENCY，未定义时沿用写频率
// 不要打开 USE_HSPI_PORT
```

- 代码级要点（已在本仓库实现）：
  - `esp32_marauder.ino` 的 SD 对象必须直接用全局 `SPI`（`SDInterface(&SPI, SD_CS)`）。**不要**写 `SPIClass sharedSPI(SPI)` 拷贝构造，否则会破坏总线仲裁，表现为白屏；
  - `SDInterface::initSD()` 的 `HAS_C5_SD` 分支先执行 `_spi->begin(SD_SCK, SD_MISO, SD_MOSI, -1)` 再 `SD.begin()`；
  - C5 启动后把 CPU 提升到 240MHz，以保证 TFT 传输速度。

### TFT_eSPI `User_Setup.h` 同步清单

Arduino IDE 编译时只认全局库文件，切换板型必须逐项改完再**完整重编译**：

| 配置项 | XiaoMiao | ESP32-C5 Mini V3 | Marauder V8 |
|---|---|---|---|
| 驱动 / tab | `ST7735_DRIVER` + `ST7735_GREENTAB` | `ST7735_DRIVER` + `ST7735_GREENTAB3` | `ILI9341_DRIVER` |
| 引脚（MISO/MOSI/SCLK/CS/DC） | -1 / 23 / 18 / 5 / 4 | 24 / 0 / 26 / 25 / 23 | 5 / 1 / 9 / 24 / 10 |
| 复位 / 背光 | RST=-1，BL=32（空脚） | RST=1，BL=3 | RST=0，BL=23（`User_Setup.h` 不定义 BL） |
| 分辨率 | 128×160 | 128×128 | 240×320 |
| SPI 频率 | 10MHz | 4MHz 写 / 2MHz 读 | 10MHz 写（可试 27MHz），触摸 2.5MHz |
| 其他 | `TFT_INVERSION_OFF`、`TFT_RGB_ORDER TFT_BGR` | bit-bang，禁止 `USE_HSPI_PORT` / `USE_FSPI_PORT` | `TOUCH_CS`=4，不开 `USE_HSPI_PORT` |

### 切换板型步骤

1. 在 `esp32_marauder/configs.h` 的 `//// BOARD TARGETS` 区注释掉当前板型，启用目标板型宏；
2. 若目标板型是继承型（如 XiaoMiao），**紧接着**定义 `MARAUDER_MINI`（见上文"板型继承宏必须前置"）；V8 不继承 `MARAUDER_MINI`；
3. 同步修改全局 TFT_eSPI `User_Setup.h`：驱动、引脚、分辨率、SPI 频率、tab 变体；
4. 切换编译目标：C5 板型用 `esp32:esp32:esp32c5`，经典 ESP32 用 `esp32:esp32:esp32`；分区统一 `PartitionScheme=huge_app`；
5. 完整重编译（不要依赖增量缓存），确认固件体积与同版本正常构建一致、Display 代码已编入；
6. 烧录后检查：屏幕点亮、按键方向正确、SD 可挂载、GPS（若接线）可定位。

### 板型适配文档

- [XiaoMiao 编译与配置说明书](docs/board_adapters/XiaoMiao_编译与配置说明书.md)
- [ESP32-C5 Mini V3 编译与配置说明书](docs/board_adapters/ESP32C5_Mini_V3_编译与配置说明书.md)
- [C5 V8 移植与修改说明书](docs/board_adapters/C5_V8_移植与修改说明书.md)

# For Sale Now
You can buy the ESP32 Marauder using [this link](https://www.justcallmekokollc.com)
