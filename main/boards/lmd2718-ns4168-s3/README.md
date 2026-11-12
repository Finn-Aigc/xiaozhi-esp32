# ESP32-S3 DevKitC + LMD2718 + NS4168 (DIY)

> 中文版: [README_zh.md](README_zh.md)

Bare `ESP32-S3-DevKitC-1` (N16R8, 16 MB flash / 8 MB PSRAM) with the common
"小智 DIY" two-in-one audio module: **LMD2718** PDM microphone + **NS4168**
I2S class-D amplifier.

Unlike the `bread-compact-wifi` board this variant uses the PDM interface of
the microphone (`NoAudioCodecSimplexPdm`), so no INMP441 is required.

## Wiring

| Module pin | ESP32-S3 | Notes |
|------------|----------|-------|
| LRCLK      | GPIO16   | NS4168 word select |
| BCLK       | GPIO15   | NS4168 bit clock |
| SDA        | GPIO7    | NS4168 audio data |
| CLK        | GPIO5    | LMD2718 PDM clock |
| DATA       | GPIO6    | LMD2718 PDM data |
| VCC        | 3.3V     | 5V is possible on some modules for louder output |
| GND        | GND      | Common ground required |
| SPK +/-    | Speaker  | 4Ω/8Ω 2–3 W speaker |

Optional SSD1306 OLED (128x64, address 0x3C): SDA → GPIO41, SCL → GPIO42.
The firmware falls back to a headless display when no OLED answers.

Buttons: the on-board BOOT key (GPIO0) toggles the conversation and enters
Wi-Fi configuration during startup. Optional volume keys: GPIO40 (volume up)
and GPIO39 (volume down) to GND — short press changes the volume by 10%, long
press sets maximum / mutes. The on-board RGB LED is on GPIO48.

## Microphone gain

The PDM microphone gain is a software multiplier persisted in NVS
(`audio/input_gain`, 1 = default, 4 = maximum). It can be changed at runtime
through the MCP tools `self.audio_microphone.get_gain` and
`self.audio_microphone.set_gain`; for example say "set the microphone gain to
2" to the assistant. Values above 3 may clip loud speech.

## Build

```bash
python scripts/build.py lmd2718-ns4168-s3
```

## Flash

Use the CH343 USB-UART port (install the WCH CH343 driver on Windows).

```bash
idf.py -p COMx flash monitor
```

## Known hardware quirk: XMC-compatible flash

Some DevKitC boards ship a flash chip reporting RDID `0x20 0x70 0x18`
(XM25QHxxC-compatible) that is missing from ESP-IDF's strict XMC model table.
With the stock bootloader the chip startup flow fails the re-check and the
board reset-loops. Apply this local patch to
`$IDF_PATH/components/bootloader_support/bootloader_flash/src/bootloader_flash.c`
(inside `bootloader_flash_xmc_startup`): after the startup flow, accept the
chip if the XMC vendor ID (`0x20`) is still present instead of returning
`ESP_FAIL`. The project disables bootloader logging by default; enable
`CONFIG_BOOTLOADER_LOG_LEVEL_INFO` temporarily with `idf.py menuconfig` when
diagnosing such failures.

