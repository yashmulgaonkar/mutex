# Third-party notices

SemaphoreSign's own files (firmware source in `src/`, the OpenSCAD model, `webflasher/` except `webflasher/vendor/`, and the documentation)
are licensed under **CC BY-NC-SA 4.0**, see [LICENSE](LICENSE) and [NOTICE](NOTICE). The components below are **not** covered by that
license: each one stays under its own terms, which apply to it wherever it is redistributed. This file is informational, not legal advice.

## Included in this repository

| Component | Where | License |
|---|---|---|
| [esptool-js](https://github.com/espressif/esptool-js) 0.5.7 (Espressif) | `webflasher/vendor/esptool-js.bundle.js`, unmodified | Apache-2.0, full text in `webflasher/vendor/esptool-js.LICENSE.txt` |
| [pako](https://github.com/nodeca/pako) 2.1.0 (bundled inside esptool-js) | same file | MIT AND Zlib |

## Compiled into the published firmware

PlatformIO downloads these at build time (they are not stored in this repository). The firmware images that the web installer
([`webflasher/`](webflasher/), published on GitHub Pages) serves are built from this repository's source by GitHub Actions and include them:

| Component | License |
|---|---|
| [Arduino-ESP32 core](https://github.com/espressif/arduino-esp32) (Wire, WiFi, WebServer, DNSServer, ESPmDNS, Preferences, ...) | LGPL-2.1 |
| [ESP-IDF](https://github.com/espressif/esp-idf) and the components it bundles, as used by the Arduino core | mostly Apache-2.0; see each component's own license |
| [Adafruit LED Backpack Library](https://github.com/adafruit/Adafruit_LED_Backpack) | MIT |
| [Adafruit GFX Library](https://github.com/adafruit/Adafruit-GFX-Library) | BSD |
| [Adafruit BusIO](https://github.com/adafruit/Adafruit_BusIO) | MIT |
| [Adafruit seesaw Library](https://github.com/adafruit/Adafruit_Seesaw) | BSD (core files); `seesaw_neopixel` is LGPL, see its file headers |
| [Adafruit NeoPixel](https://github.com/adafruit/Adafruit_NeoPixel) | LGPL-3.0 |

Some of these libraries are only partly linked into the final image; the list is the conservative one.

**LGPL components:** the complete source of the firmware is public in this repository. To use a modified version of an LGPL library,
rebuild with PlatformIO (`pio run`, see `platformio.ini`; point `lib_deps` at your version) and flash the result with
`pio run -t upload`.

## Hardware data

Dimensions in the OpenSCAD enclosure model were taken from the manufacturers' public CAD data and datasheets (Waveshare ESP32-S3-Zero,
Adafruit NeoPixel Ring PID 1643, Adafruit I2C Rotary Encoder PID 5880, Adafruit 0.54" Quad Alphanumeric FeatherWing). Those models are
not included in this repository. Product names and trademarks belong to their owners; this project is not affiliated with them.
