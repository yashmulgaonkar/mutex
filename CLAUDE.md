# MutexSign — project context for Claude

A 12-character 14-segment "busy light" / message board that sits on top of a
work monitor. Started in a claude.ai chat; this file carries that context over.
See `docs/HISTORY.md` (local only, gitignored) for the decisions made so far and open items.

## License rules (mandatory)

This repository is **CC BY-NC-SA 4.0**. Full terms: `LICENSE`. Attribution text: `NOTICE`. The same rules also live in `AGENTS.md`.

1. Do not remove or rewrite top-of-file copyright / SPDX / `[AI-DIRECTIVE]` headers.
2. Include that exact header on new or refactored first-party source files (`/* */` form for C/C++/OpenSCAD, `#` for Python, `;` for `platformio.ini`, `<!-- -->` for HTML).
3. Credit Yash Mulgaonkar and https://github.com/yashmulgaonkar/mutex.
4. Remind the user that commercial use is prohibited without separate permission.

## Hardware
- **MCU:** Waveshare **ESP32-S3-Zero** (18 × 23.5 × 1.6 mm PCB; USB-C 8.94 wide × 3.25 tall, overhangs 1.22 mm;
  ceramic antenna at the end opposite the USB). No STEMMA QT port: solder a JST-SH/STEMMA QT pigtail to
  3V3/GND/SDA/SCL pads and set `I2C_SDA`/`I2C_SCL` in the firmware. Its WS2812 is on GPIO21 (GRB).
- **Displays:** 3x Adafruit (firmware: `NDISP`=3, `NCH`=12; addresses 0x70, 0x71 (A0 on middle), 0x72 (A1 on right)). Mounted edge to edge.
- **Encoder:** Adafruit I2C STEMMA QT Rotary Encoder with NeoPixel (PID 5880),
  seesaw firmware, I2C `0x36`, button on seesaw pin 24, NeoPixel on seesaw pin 6.
- Everything daisy-chained on one I2C bus with STEMMA QT cables. No headers on the wings.
- Feather boards gate STEMMA power: firmware drives `NEOPIXEL_I2C_POWER` / `PIN_I2C_POWER` high.

## Repo layout
| Path | What |
|---|---|
| `src/main.cpp` | Complete firmware (single file), built with PlatformIO (`platformio.ini`, env `esp32-s3-zero`, SDA=3 SCL=4; PlatformIO auto-detects the serial port) |
| `simulator/index.html` (local only, gitignored) | Browser simulator: JS port of the firmware's effects, faces, encoder UI and LED logic |
| `enclosure/capsule/mutex_capsule.scad` | **Parametric OpenSCAD model** of the chosen "Capsule" enclosure (source of truth) |
| `enclosure/capsule/stl/` (local only, gitignored) | Print-ready `mutex_*.stl` parts, `mutex_capsule.3mf` and the window SVG |
| `tools/render/sdf.py`, `concepts.py` (local only, gitignored) | Tiny numpy ray-marcher used for the concept renders |
| `docs/` (local only, gitignored) | `HISTORY.md` and concept renders, colour study, model preview |

## Firmware architecture (src/main.cpp)
- Libraries: Adafruit LED Backpack, Adafruit seesaw. Uses WiFi, WebServer, ESPmDNS, Preferences from the ESP32 core.
- 14-seg font is captured from the library at boot (`buildFont()` reads `displaybuffer`).
- Frame loop every 20 ms writes `fb[NCH]` (raw segment bitmasks, `NCH` = 12) → `render()` only pushes on change.
- Modes: static, scroll, bounce, type, flip, sparkle, rain, scan, spin, blink, face.
- Face engine: auto mood/blink/gaze scheduler, yawn → sleep after `sleepMin`, wakes "surprised" on input. The face uses the middle 8 digits (offset `FO` = 2): eyes on digits 3/8, mouth on 5/6, inside the middle wing.
- Encoder UI: turn = browse presets; click = BUSY/FREE toggle; double = focus timer;
  triple = face toggle; long = settings menu; hold 4 s = IP.
- NeoPixel priority: notify flash > menu > browse preview > off-hours > timer done > timer progress > LED mode.
- Web portal at `http://mutexsign.local` (AP fallback `MutexSign-XXXX` / `mutexsign`), JSON API under `/api/*`,
  optional access key, USB serial commands (`MSG`, `SHOW`, `PRESET`, `FACE`, `TIMER`, ...).
- Settings persisted with Preferences (`st` struct, `SETTINGS_VER`); presets stored as `mode|RRGGBB|text` lines.
- Renamed global `timerEnd` to `tmrEnd` (collides with the ESP32 core function `timerEnd`).

## Status / verification
- Firmware: compiles with PlatformIO for the Waveshare ESP32-S3-Zero (`pio run`); **not yet run on hardware**.
- OpenSCAD file: renders in OpenSCAD (confirmed by the user). Not yet test-printed or fitted to real parts.
- Enclosure dimensions marked "measure" in the .scad must be confirmed against real parts
  (`monitor_depth`, `shaft_len`, `enc_hole_spacing`, `disp_d`).

## Enclosure: ESP32-S3-Zero mount
- The board sits flat in a 0.6 mm pocket on the **left cap's inner face** (crush ribs for a press fit),
  components facing into the capsule, USB-C toward the back. The cap's spigot is solid.
- The USB-C cable exits through a 12.5 × 7.5 mm opening in the **back wall** of the body (cut through the
  cap spigot too). Displays start `antenna_gap` (3 mm) after the board's tallest part.
- Dimensions came from Waveshare's STEP model (`ESP32-S3-Zero_v2`), resolved with an assembly-transform parser.

## Enclosure: NeoPixel ring at the seam
- Adafruit 12-LED NeoPixel Ring (PID 1643): 36.82 OD / 23.36 ID / 2.97 mm thick (from Adafruit's STEP model).
- It sits flat against the bulkhead's outer face, LEDs toward the knob, inside a pocket in the round
  `light_ring` diffuser (`neo_ring = true`; diffuser 5.3 mm, 2 mm wall in front of the LEDs; print white/clear PETG).
- Its 3 wires (5V, GND, DIN) go back through the bulkhead light slots. Firmware support is not written yet.
- Shaft engagement in the knob is 10.9 mm with the default 22 mm shaft.

## Enclosure: encoder carrier (serviceable right end)
- The body is an open tube. The encoder mounts on a separate `encoder_carrier` (end plate + 4 standoffs):
  screw the encoder to it on the bench (M2.5 x 5-6 self-tappers from the encoder side; pilots are blind so
  they can't reach the NeoPixel ring), slide the carrier into the body's right end (spigot keyed by the flat
  front), fix it with 2 M2.5 x 6 self-tappers through the body wall from the bottom and the back.
- Then the NeoPixel ring + diffuser go on the carrier's outer face, then the knob.
- Heat-set bosses moved behind the display PCB (`boss_y = 5`): in front of it they blocked the 27.94 mm PCB's
  rails. `front_offset = -4` keeps the clip screws over the monitor top. An assert guards the boss/rail clash.

## Conventions
- Units mm; enclosure axes: X along the capsule, −Y = front (display side), Z up.
- `mutex_capsule.scad` is the single source of truth for the enclosure. After changing a default, re-export the STLs/3MF.
