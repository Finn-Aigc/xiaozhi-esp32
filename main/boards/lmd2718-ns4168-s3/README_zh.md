# ESP32-S3 DevKitC + LMD2718 + NS4168（DIY）

> English: [README.md](README.md)

裸板 `ESP32-S3-DevKitC-1`（N16R8，16MB Flash / 8MB PSRAM），搭配常见的
「小智 DIY」二合一音频模块：**LMD2718** PDM 麦克风 + **NS4168** I2S 功放。

与 `bread-compact-wifi` 不同，本变体使用麦克风的 PDM 接口
（`NoAudioCodecSimplexPdm`），不需要 INMP441。

## 接线

| 音频模块引脚 | ESP32-S3 | 说明 |
|--------------|----------|------|
| LRCLK        | GPIO16   | NS4168 左右声道时钟 |
| BCLK         | GPIO15   | NS4168 位时钟 |
| SDA          | GPIO7    | NS4168 音频数据（不是 I2C SDA） |
| CLK          | GPIO5    | LMD2718 PDM 时钟 |
| DATA         | GPIO6    | LMD2718 PDM 数据 |
| VCC          | 3.3V     | 部分模块可接 5V 以获得更大音量 |
| GND          | GND      | 必须共地 |
| SPK +/−      | 喇叭     | 4Ω/8Ω，2–3W |

可选 SSD1306 OLED（128x64，地址 0x3C）：SDA → GPIO41，SCL → GPIO42。
没有 OLED 时固件会自动使用无屏模式，不影响语音功能。

按键：板载 BOOT 键（GPIO0）用于开始/结束对话，启动时按住可进入配网模式。
可选音量键：GPIO40（音量+）、GPIO39（音量−），另一端接 GND；短按调整 10%，
长按分别设为最大音量 / 静音。板载 RGB 灯位于 GPIO48。

## 麦克风增益

PDM 麦克风的增益是软件倍数，保存在 NVS（`audio/input_gain`，1 = 默认，
4 = 最大）。可在运行时通过 MCP 工具 `self.audio_microphone.get_gain` 和
`self.audio_microphone.set_gain` 调整，例如对小智说「把麦克风增益调到 2」。
超过 3 倍时大声说话可能削波。

## 编译

```bash
python scripts/build.py lmd2718-ns4168-s3
```

## 烧录

使用 CH343 USB 串口口（Windows 需安装 WCH CH343 驱动）。

```bash
idf.py -p COMx flash monitor
```

## 已知硬件问题：XMC 兼容闪存

部分 DevKitC 使用 RDID 为 `0x20 0x70 0x18`（XM25QHxxC 兼容）的闪存，
不在 ESP-IDF 的严格 XMC 型号白名单中。原版 bootloader 在上电启动流程后
校验失败，表现为反复复位重启。可对
`$IDF_PATH/components/bootloader_support/bootloader_flash/src/bootloader_flash.c`
打一个本地补丁（在 `bootloader_flash_xmc_startup` 中）：启动流程结束后，
只要厂商 ID 仍为 XMC（`0x20`）就继续启动，而不是返回 `ESP_FAIL`。
项目默认关闭 bootloader 日志，排查时可在 `idf.py menuconfig` 中临时开启
`CONFIG_BOOTLOADER_LOG_LEVEL_INFO`。
