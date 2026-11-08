# ESP32-C5 Mini V3 编译与配置说明书

> 适用版本：ESP32 Marauder v1.20.0 + ESP32-C5N16R8 + 1.44" ST7735S 128×128
> 
> 路径：`J:\Documents\Arduino\esp32marauder_1.20.0`
> 
> 参考工程（已验证可工作）：`J:\MCU\ESP32\ESP32Marauder\esp32_marauder_mini_v3-ESP32C5N16R8-final`

---

## 一、硬件概览

| 项目 | 说明 |
|------|------|
| MCU | **ESP32-C5N16R8**（RISC-V + WiFi 7 + BLE 5.0，16MB Flash） |
| 屏幕 | **1.44" ST7735S**，原生 portrait 128×128，rotation=0 |
| TF 卡 | MicroSD，与屏幕共享 SPI 总线 |
| 按键 | 5 键（左/确认/上/右/下），低电平触发，内部上拉 |
| GPS | Serial1（TX=14, RX=13），可选接 LC76G 模块 |
| CSI | ESP32-C5 内置 CSI，支持 motion detection |
| USB | USB Serial (JTAG)，GPIO19-20 |

### 引脚表

#### TFT + SD（共享 SPI 总线）

| 功能 | 引脚 | 备注 |
|------|------|------|
| TFT SCLK | 26 | bit-bang SPI（ESP32-C5 不走硬件 SPI） |
| TFT MOSI | 0 | |
| TFT MISO | 24 | |
| TFT CS | 25 | |
| TFT DC | 23 | |
| TFT RST | 1 | MCU GPIO 控制 |
| TFT BL | 3 | 背光控制 |
| TFT 背光 | — | 软件通过 GPIO3 控制 TFT_BL |
| SD CS | 4 | |
| SD MISO | **24** | = TFT_MISO，与 TFT 共享 |
| SD MOSI | **0** | = TFT_MOSI，与 TFT 共享 |
| SD SCK | **26** | = TFT_SCLK，与 TFT 共享 |
| SD_DET | 5 | 卡检测（仅 DUAL_MINI_C5 启用） |

#### 按键（DUAL_MINI_C5 映射）

| 功能 | 引脚 | 备注 |
|------|------|------|
| 按键 L（左） | 6 | |
| 按键 C（确认） | 9 | |
| 按键 U（上） | 7 | |
| 按键 R（右） | 8 | |
| 按键 D（下） | 10 | |

> **注意**：configs.h 同时定义 `MARAUDER_MINI_V3` 和 `DUAL_MINI_C5`。DUAL_MINI_C5 会在 BOARD FEATURES 区自动再 define 一次 MINI_V3，并触发 Display/Buttons 区域的 `#ifdef DUAL_MINI_C5` 分支。MINI_V3 base（无 DUAL_MINI_C5）的按键映射不同：L=10, C=9, U=6, R=7, D=8（L↔U、R↔D 交换）。

#### GPS（可选）

| 功能 | 引脚 | 备注 |
|------|------|------|
| GPS TX | 14 | Serial1 TX |
| GPS RX | 13 | Serial1 RX |

---

## 二、修改的文件清单

### 1. `configs.h` — 板型与引脚

```c
// L44-46 BOARD TARGETS 区：注释其他板型，启用 Mini V3 + DUAL_MINI_C5
#define MARAUDER_MINI_V3
#define DUAL_MINI_C5
// #define MARAUDER_XIAOMIAO   ← 切换板型时启用这个、注释上面两行

// L183-185 BOARD FEATURES 区（已存在）
#if defined(DUAL_MINI_C5)
    #define MARAUDER_MINI_V3  // 自动继承 Mini V3 所有 feature 宏
#endif

// L1092-1117 BUTTON DEFINITIONS 区：按钮引脚
//   - DUAL_MINI_C5 分支: L=6, C=9, U=7, R=8, D=10
//   - 内部上拉启用（L_PULL/C_PULL/U_PULL/R_PULL/D_PULL = true）

// L2421-2438 DISPLAY DEFINITIONS 区：TFT/SD 引脚 + SD_DET
//   TFT_MISO=24, MOSI=0, SCLK=26, CS=25, DC=23, RST=1, BL=3
//   SD_CS=4, SD_DET=5
//   TFT_WIDTH=128, TFT_HEIGHT=128, SCREEN_ORIENTATION=0

// L2958-2960 SD DEFINITIONS 区：SD_CS 必须与 Display 区一致
#define SD_CS 4    // 之前 Mini V3 是 10，已改成 4 与 TFT 定义对齐

// L3226-3229 GPS DEFINITIONS 区（已存在，不用改）
#define GPS_SERIAL_INDEX 1
#define GPS_TX 14
#define GPS_RX 13
```

### 2. `J:\Documents\Arduino\libraries\TFT_eSPI\User_Setup.h` — TFT_eSPI 库配置

> ⚠️ **TFT_eSPI User_Setup.h 是全局库文件，Mini V3 和 XiaoMiao 共用同一份。切换板型必须同步改。**

```c
#define USER_SETUP_LOADED
#undef USE_HSPI_PORT
#undef USE_FSPI_PORT     // ESP32-C5 用 bit-bang SPI，不走硬件 SPI 端口

// 驱动
#define ST7735_DRIVER
#define ST7735_GREENTAB3
#define TFT_RGB_ORDER TFT_BGR   // 颜色顺序

// 分辨率（portrait 原生）
#define TFT_WIDTH  128
#define TFT_HEIGHT 128

// 引脚
#define TFT_MISO 24
#define TFT_MOSI 0
#define TFT_SCLK 26
#define TFT_CS   25
#define TFT_DC   23
#define TFT_RST  1
#define TFT_BL   3
#define TOUCH_CS -1

// SPI 频率 — ESP32-C5 用 bit-bang SPI，频率必须压低
#define SPI_FREQUENCY        4000000   // 4MHz 写
#define SPI_READ_FREQUENCY   2000000   // 2MHz 读
#define SPI_TOUCH_FREQUENCY  2500000
```

### ESP32-C5 bit-bang SPI 原理

TFT_eSPI 库 v2.5.43 包含 `TFT_eSPI_ESP32_C5.c`，文件头注释明确说明：

```
Bitbang SPI processor for ESP32C5 / C3 / C6 / H2
```

当目标是 ESP32-C5 时，`CONFIG_IDF_TARGET_ESP32C5` 宏会触发 C5 专用代码路径，**跳过**经典 ESP32 的硬件 SPI `spi.begin()` 调用，改为逐 bit 翻转 GPIO 实现 SPI 时序。因此：
- SPI 频率必须低（参考工程用 4MHz/2MHz）
- `USE_HSPI_PORT` / `USE_FSPI_PORT` 必须 undefined
- `#if !defined(CONFIG_IDF_TARGET_ESP32C5)` 守卫会让经典 ESP32 的 SPI 初始化代码不执行

---

## 三、编译步骤

### 前提条件
- Arduino Core ESP32 3.1.2 或更高（支持 ESP32-C5）
- `esp32:esp32:esp32c5` 开发板包已安装
- 全局 TFT_eSPI 库 v2.5.43：`J:\Documents\Arduino\libraries\TFT_eSPI\`

### Arduino IDE 方式

| 设置项 | 值 |
|--------|----|
| Board | **ESP32C5 Dev Module** |
| Partition Scheme | **Huge APP (3MB No OTA)** |
| Flash Size | 16MB |
| PSRAM | OPI PSRAM (2MB) |
| Upload Speed | 921600 |

> ⚠️ **Partition Scheme 必须选 Huge APP**。固件约 2.28MB，默认分区不够。
> 
> ⚠️ **PSRAM 必须选 OPI PSRAM (2MB)**。ESP32-C5 16MB Flash / 2MB PSRAM 是硬件配置，选错会启动失败。

### arduino-cli 方式

```powershell
# 编译
arduino-cli compile --fqbn esp32:esp32:esp32c5:PartitionScheme=huge_app 'J:\Documents\Arduino\esp32marauder_1.20.0'

# 烧录（把 COMx 改成实际端口）
arduino-cli upload -p COM5 --fqbn esp32:esp32:esp32c5:PartitionScheme=huge_app 'J:\Documents\Arduino\esp32marauder_1.20.0'

# 串口监控
arduino-cli monitor -p COM5 -c baudrate=115200
```

**编译成功标志**：
```
Sketch uses 2277234 bytes (72%) of program storage space. Maximum is 3145728 bytes.
Global variables use 89972 bytes (27%) of dynamic memory, leaving 237708 bytes for local variables. Maximum is 327680 bytes.
```

---

## 四、与 XiaoMiao（经典 ESP32）的切换操作

两套硬件共用同一工程，切换步骤：

| 步骤 | ESP32-C5 Mini V3 → XiaoMiao | XiaoMiao → ESP32-C5 Mini V3 |
|------|------|------|
| configs.h L44-48 | 注释 `MARAUDER_MINI_V3`+`DUAL_MINI_C5`，启用 `MARAUDER_XIAOMIAO` | 注释 `MARAUDER_XIAOMIAO`，启用 `MARAUDER_MINI_V3`+`DUAL_MINI_C5` |
| 编译目标 | `esp32:esp32:esp32` | `esp32:esp32:esp32c5` |
| User_Setup.h 驱动 | ST7735 + GREENTAB + BGR + INVERSION_OFF | ST7735 + GREENTAB3 + BGR |
| User_Setup.h 尺寸 | 128×160 | 128×128 |
| User_Setup.h 引脚 | MISO=-1/MOSI=23/SCLK=18/CS=5/DC=4/RST=-1 | MISO=24/MOSI=0/SCLK=26/CS=25/DC=23/RST=1 |
| User_Setup.h SPI | 10MHz（硬件 SPI） | 4MHz write / 2MHz read（bit-bang） |

---

## 五、常见故障排查

### 故障 1：编译错误 "SPI.h: No such file or directory" 或 SPI 相关函数 undefined

**根因**：编译目标不是 ESP32-C5，或者 ESP32 Core 版本太旧不支持 C5。

**检查**：
- `arduino-cli board listall | findstr c5` — 应该看到 `ESP32C5 Dev Module`
- Arduino IDE → Boards Manager → 搜索 esp32 → 最新版应为 3.1.2+

### 故障 2：编译通过但烧录后黑屏

**检查顺序**：
1. `User_Setup.h` 引脚是否正确（MISO=24, MOSI=0, SCLK=26, CS=25, DC=23, RST=1, BL=3）
2. `configs.h` L44-46 是否同时启用了 MARAUDER_MINI_V3 + DUAL_MINI_C5
3. 固件体积是否 ≥ 2MB（证明 Display 代码编入了）
4. 串口输出是否正常（115200 baud）

### 故障 3：屏幕有显示但边缘雪花杂色

**根因**：ST7735 GREENTAB3 偏移与玻璃不匹配。

**修复**：参考 XiaoMiao 说明书里的 Tab 变体选择指南，在 `User_Setup.h` 里依次尝试：
1. `ST7735_GREENTAB`（rotation=0 时 colstart=2, rowstart=3）
2. `ST7735_GREENTAB2`（rotation=0 时 colstart=2, rowstart=1）
3. `ST7735_REDTAB` / `BLACKTAB`（rotation=0 时 colstart=2, rowstart=1）

> 提示：Mini V3 当前 SCREEN_ORIENTATION=0，参考工程用 GREENTAB3。如果还有雪花，换 tab 变体先试，必要时同时调整 SCREEN_ORIENTATION。

### 故障 4：屏幕显示但按键没反应

**检查**：
1. `configs.h` L1102-1106 DUAL_MINI_C5 按钮引脚是否正确（L=6, C=9, U=7, R=8, D=10）
2. 如果用的是 base Mini V3（无 DUAL_MINI_C5），按钮映射是 L=10, C=9, U=6, R=7, D=8
3. 内部上拉是否启用（L_PULL=true 等）

### 故障 5：TF 卡不识别

**检查**：
1. `configs.h` L2958-2960 SD_CS 是否为 4（与 Display 区一致）
2. SD MISO=24 / MOSI=0 / SCK=26（与 TFT 共享总线）
3. `HAS_C5_SD` / `HAS_SD` / `USE_SD` 是否由 MINI_V3 继承生效

### 故障 6：颜色红蓝互换

**根因**：BGR/RGB 顺序不对。

**修复**：在 `User_Setup.h` 里切换：
```c
#define TFT_RGB_ORDER TFT_BGR  // 当前选择
// #define TFT_RGB_ORDER TFT_RGB  // 如果红蓝互换就换这个
```

### 故障 7：固件太大（>3MB）编不过

**修复**：分区方案选 **Huge APP (3MB No OTA/1MB SPIFFS)**。

### 故障 8：烧录后进入 Recovery 模式

**检查**：PSRAM 设置是否正确。ESP32-C5N16R8 硬件有 OPI PSRAM 2MB，IDE 里要选 OPI PSRAM (2MB)，不是 QSPI PSRAM。

---

## 六、硬件注意事项

### ESP32-C5 vs 经典 ESP32 的关键差异

| 特性 | ESP32-C5 | 经典 ESP32 |
|------|----------|-----------|
| 架构 | RISC-V 32-bit | Xtensa LX6 双核 |
| WiFi | WiFi 7 (802.11be) | WiFi 4 (802.11n) |
| SPI | **bit-bang** (TFT_eSPI_ESP32_C5.c) | 硬件 SPI (VSPI/HSPI) |
| SPI 频率 | ≤ 4MHz | 10-27MHz |
| 板型 feature 宏 | `HAS_C5_SD`, `HAS_PSRAM` | 无 C5 专属宏 |
| Core 版本要求 | ≥ 3.1.2 | ≥ 2.0.x |

### ESP32-C5 可用 GPIO 范围

ESP32-C5 的 GPIO 范围更大（0-48），但需要注意：
- GPIO19-20：USB D-/D+，USB 模式下不能用
- GPIO11-16：Strapping pins，启动时决定 Flash/PSRAM 模式，上电时有特殊电平要求
- GPIO33+：部分为 SPI Flash / PSRAM 专用（取决于 Flash 规格），不能作通用 IO

---

## 七、参考资源

- 可工作参考工程：`J:\MCU\ESP32\ESP32Marauder\esp32_marauder_mini_v3-ESP32C5N16R8-final`（v1.12.1，含 sketch 内 TFT_eSPI 副本）
- TFT_eSPI 全局库：`J:\Documents\Arduino\libraries\TFT_eSPI\`（v2.5.43）
- TFT_eSPI C5 专用实现：`J:\Documents\Arduino\libraries\TFT_eSPI\TFT_eSPI_ESP32_C5.c`
- TFT_eSPI ST7735 偏移数据：`J:\Documents\Arduino\libraries\TFT_eSPI\TFT_Drivers\ST7735_Rotation.h`
- Mini V3 参考 User_Setup：参考工程根目录 `User_Setup.h`（105 字节转发壳）+ `User_Setup_marauder_mini_v3.h`
- ESP32-C5 Arduino Core 文档：https://docs.espressif.com/projects/arduino-esp32/en/latest/boards/esp32-c5-devkitc-1.html

---

## 八、文档索引

| 文档 | 适用硬件 | 核心 MCU |
|------|---------|---------|
| 本文件（ESP32C5_Mini_V3_编译与配置说明书.md） | Mini V3 | ESP32-C5 |
| `XiaoMiao_编译与配置说明书.md` | XiaoMiao | 经典 ESP32 |
| `C5_V8_移植与修改说明书.md` | V8 官方版 + V8 修改版（两套独立板型宏） | ESP32-C5 |
