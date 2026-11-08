# XiaoMiao 掌机 ESP32 Marauder 编译与配置说明书

> 适用版本：ESP32 Marauder v1.20.0 + 经典 ESP32 + 1.8" ST7735S
> 
> 路径：`J:\Documents\Arduino\esp32marauder_1.20.0`
> 
> 参考工程（已验证可工作）：`J:\MCU\ESP32\ESP32Marauder\esp32_marauder_mini_for_xiaomiao_esp32`（v1.12.1）
>
> ⚠️ **共享库切换提醒**：本工程两套板型共用全局 TFT_eSPI 库
> `J:\Documents\Arduino\libraries\TFT_eSPI\User_Setup.h`。
> 切换板型（ESP32-C5 Mini V3 ↔ XiaoMiao）时必须同步改这个文件。
> 当前 User_Setup.h 配置的是 **ESP32-C5 Mini V3**，切回 XiaoMiao 时按本文档第二节重新改。

---

## 一、硬件概览

| 项目 | 说明 |
|------|------|
| MCU | 经典 ESP32（WROOM-32），**不是** ESP32-C5/C3/S3 |
| 屏幕 | 1.8" ST7735S，原生 portrait 128×160，**应用层横屏 160×128** |
| TF 卡 | MicroSD，与屏幕共享 VSPI 总线 |
| 按键 | 6 键（上/下/左/右/A 确认/B 返回），GPIO34/35 输入-only，PCB 外部上拉 |
| LED | NeoPixel（GPIO25，继承自 MARAUDER_MINI） |
| 串口 | GPS 用 Serial2（可选） |

### 引脚表

| 功能 | 引脚 | 备注 |
|------|------|------|
| TFT SCLK | 18 | VSPI CLK |
| TFT MOSI | 23 | VSPI MOSI |
| TFT MISO | **-1** | ST7735 不回读，释放 GPIO19 给 SD 卡 |
| TFT CS | 5 | |
| TFT DC | 4 | |
| TFT RST | **-1** | ST7735 自带 POR 上电复位，不用 MCU 控制 |
| TFT 背光 | 直连电源 | 软件不控 |
| SD CS | 22 | |
| SD MISO | 19 | VSPI MISO |
| SD MOSI | 23 | 与 TFT 共享 |
| SD SCK | 18 | 与 TFT 共享 |
| 按键上 | 2 | |
| 按键下 | 13 | |
| 按键左 | 27 | |
| 按键右 | 35 | 输入-only，外部上拉 |
| 按键 A | 34 | 输入-only，外部上拉 |
| 按键 B | 12 | |
| NeoPixel | 25 | |

> ⚠️ **经典 ESP32 的 GPIO6–11 是 SPI Flash 专用引脚，绝对不能用作 TFT/SD/按键。**

---

## 二、修改的文件清单

### 1. `configs.h` — 板型定义

```c
// L48 BOARD TARGETS 区：取消其他板型注释，保留 XiaoMiao
#define MARAUDER_XIAOMIAO

// L59-66 BOARD TARGETS 之后：必须立即继承 MARAUDER_MINI
// —— 不能只在 xiaomiao_config.h 末尾才 define MARAUDER_MINI，
//    那样 configs.h 前面所有 #ifdef MARAUDER_MINI 块都不会生效
//    （HAS_SCREEN / HAS_MINI_SCREEN / HAS_SD / HAS_BUTTONS / 屏幕引脚 / 几何参数 等），
//    导致 Display 代码被整体编译剔除 → 背光亮但全屏白、串口正常。
#if defined(MARAUDER_XIAOMIAO) && !defined(MARAUDER_MINI)
  #define MARAUDER_MINI
#endif

// L147-148 HARDWARE NAMES 区（已存在）
#elif defined(MARAUDER_XIAOMIAO)
  #define HARDWARE_NAME "XiaoMiao Mini"

// L3616-3618 文件末尾（已存在）
#if defined(MARAUDER_XIAOMIAO)
  #include "xiaomiao_config.h"
#endif
```

### 2. `xiaomiao_config.h` — 运行时引脚与几何覆盖

本文件在 configs.h **所有官方 `#ifdef MARAUDER_MINI` 块执行完毕后** 才被 include，通过 `#undef` + `#define` 覆盖默认值。覆盖项：

- 按键引脚（L=27, C=34, U=2, R=35, D=13, B=12）及上拉配置
- TFT 引脚（CS=5, DC=4, RST=-1, MISO=-1, MOSI=23, SCLK=18）
- SD 引脚（CS=22, SCK=18, MISO=19, MOSI=23）
- 屏幕几何：DISP_W=160, DISP_H=128, SCREEN_ORIENTATION=3
- HAS_B 启用 B 键实例化
- 屏幕布局宏（FRAME_X/Y/W/H, 按钮坐标等）适配 160×128 横屏

### 3. `J:\Documents\Arduino\libraries\TFT_eSPI\User_Setup.h` — TFT_eSPI 库配置

> ⚠️ v1.20.0 使用**全局库**，不是 sketch 内副本。路径固定为 `J:\Documents\Arduino\libraries\TFT_eSPI\User_Setup.h`。

```c
// 驱动选择
#define ST7735_DRIVER           // 1.8" ST7735S
#define ST7735_GREENTAB         // tab 变体：rotation=3 时 rowstart=3，避免上边缘雪花
#define TFT_INVERSION_OFF       // 禁止像素反转
#define TFT_RGB_ORDER TFT_BGR   // 颜色顺序

// 分辨率（portrait 原生）
#define TFT_WIDTH  128
#define TFT_HEIGHT 160

// 引脚
#define TFT_MISO -1             // ST7735 不回读
#define TFT_MOSI 23
#define TFT_SCLK 18
#define TFT_CS   5
#define TFT_DC   4
#define TFT_RST  -1

// SPI 频率
#define SPI_FREQUENCY        10000000
#define SPI_READ_FREQUENCY   10000000
```

### ST7735 Tab 变体选择指南（边缘雪花杂色调参）

ST7735 每个 tab 变体在不同 rotation 下有硬编码的 `colstart`/`rowstart`（像素偏移）。屏幕玻璃因厂家切分略有差异，需要选最匹配的。以下是 rotation=3（本板使用）的值：

| Tab 变体 | colstart | rowstart | 适用场景 |
|----------|----------|----------|----------|
| `ST7735_GREENTAB` | 1 | **3** | ✅ 当前选择，rowstart 最大 |
| `ST7735_GREENTAB2` | 1 | 2 | rowstart 中等 |
| `ST7735_GREENTAB3` | 1 | 2 | rowstart 中等（参考工程用的） |
| `ST7735_REDTAB` / `BLACKTAB` | 0 | 0 | 无偏移，可能有更多边缘雪花 |

**如果换 tab 后还有杂色**，可尝试同时改 rotation：
- 把 `xiaomiao_config.h` 里 `SCREEN_ORIENTATION` 从 3 改成 1
- rotation=1 时 `ST7735_GREENTAB3` 的 colstart=3（比 rotation=3 多 2px），能大幅修正左边雪花
- 但方向可能翻转，需实测

**切换步骤**：编辑 `User_Setup.h` 里的 `#define ST7735_xxx` 行 → 重新编译烧录。每次只换一个 tab 变体。

---

## 三、编译步骤

### 前提条件
- Arduino IDE 或 arduino-cli 已安装
- ESP32 Arduino Core 2.x 已安装（`esp32:esp32:esp32`）
- 全局 TFT_eSPI 库 v2.5.43 已安装（`J:\Documents\Arduino\libraries\TFT_eSPI\`）

### Arduino IDE 方式

| 设置项 | 值 |
|--------|----|
| Board | **ESP32 Dev Module** |
| Partition Scheme | **Huge APP (3MB No OTA/1MB SPIFFS)** |
| Flash Size | 4MB |
| Upload Speed | 921600 |

> ⚠️ **Partition Scheme 必须选 Huge APP**。固件约 2.02MB，默认分区不够。

### arduino-cli 方式

```powershell
# 编译
arduino-cli compile --fqbn esp32:esp32:esp32:PartitionScheme=huge_app 'J:\Documents\Arduino\esp32marauder_1.20.0'

# 烧录（把 COMx 改成实际端口）
arduino-cli upload -p COM5 --fqbn esp32:esp32:esp32:PartitionScheme=huge_app 'J:\Documents\Arduino\esp32marauder_1.20.0'

# 串口监控
arduino-cli monitor -p COM5 -c baudrate=115200
```

**编译成功标志**：
```
Sketch uses 2022031 bytes (64%) of program storage space. Maximum is 3145728 bytes.
Global variables use 79096 bytes (24%) of dynamic memory, leaving 248584 bytes for local variables. Maximum is 327680 bytes.
```

固件体积约 2.02MB（64%）。之前白屏时体积只有 1.25MB（95%），说明当时 Display/菜单代码被编译剔除了。

---

## 四、常见故障排查

### 故障 1：烧录后背光常亮、全屏白、串口正常

**根因**：`MARAUDER_MINI` 宏定义太晚，导致 configs.h 前面所有 `#ifdef MARAUDER_MINI` 块不生效，Display 类被 `#ifdef HAS_SCREEN` 整体排除。

**检查**：
1. 打开 configs.h，L59-66 是否有：
   ```c
   #if defined(MARAUDER_XIAOMIAO) && !defined(MARAUDER_MINI)
     #define MARAUDER_MINI
   #endif
   ```
2. 重新编译后固件体积是否 ≥ 2MB（Display 代码编入后的体积）

### 故障 2：屏幕能显示但边缘有雪花杂色

**根因**：ST7735 tab 变体的 colstart/rowstart 偏移与屏幕玻璃不匹配。

**修复**：参考上文 Tab 变体选择指南，依次尝试：
1. 先换 `ST7735_GREENTAB`（rowstart 最大）
2. 不行再换 `ST7735_GREENTAB2` / `ST7735_GREENTAB3`
3. 还是不行就改 SCREEN_ORIENTATION（rotation=1 配 GREENTAB3 能给更大的 colstart）

### 故障 3：颜色红蓝互换

**根因**：BGR/RGB 顺序不对。

**修复**：在 `User_Setup.h` 里切换：
```c
#define TFT_RGB_ORDER TFT_BGR  // 当前选择
// #define TFT_RGB_ORDER TFT_RGB  // 如果红蓝互换就换这个
```

### 故障 4：屏幕显示但按键没反应

**检查**：
1. `xiaomiao_config.h` 里按键引脚是否正确
2. GPIO34/35 必须是外部上拉，内部上拉对 input-only 引脚无效
3. Switches.cpp 里 input-only 引脚会退化成纯 INPUT，确认 pullup=true

### 故障 5：TF 卡不识别

**检查**：
1. SD_MISO=19, SD_MOSI=23, SD_SCK=18, SD_CS=22 是否正确
2. 与 TFT 共享 VSPI，所以 TFT_MISO 必须是 -1
3. configs.h 里 HAS_SD / USE_SD 是否由 MARAUDER_MINI 继承生效

### 故障 6：固件太大（>1.3MB）编不过

**根因**：Partition Scheme 不是 huge_app。

**修复**：Arduino IDE 里 Board → Partition Scheme → **Huge APP (3MB No OTA/1MB SPIFFS)**

---

## 五、参考资源

- 可工作参考工程：`J:\MCU\ESP32\ESP32Marauder\esp32_marauder_mini_for_xiaomiao_esp32`（v1.12.1，sketch 内带 TFT_eSPI 副本 + User_Setup_marauder_mini.h）
- TFT_eSPI 库：`J:\Documents\Arduino\libraries\TFT_eSPI\`（v2.5.43）
- TFT_eSPI ST7735 偏移调整注释：参考工程 `User_Setup_marauder_mini.h` 顶部
- 本工程 ST7735 tab 偏移数据：`J:\Documents\Arduino\libraries\TFT_eSPI\TFT_Drivers\ST7735_Rotation.h`
