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
- The board is sunk into a pocket in the **left cap's spigot face** (spigot 7 mm, PCB top 1 mm below the face),
  components facing into the capsule, USB-C toward the back. Held by two fixed lips at the USB end (beside the
  receptacle), a snap hook at the antenna end (upright 1 mm plate rooted 0.5 mm from the cap's shoulder,
  flexing toward the front; barb at `mcu_latch_z`, between the antenna and pad 14) with rigid stops beside it
  for USB plug force, and wire channels along both long edges. Hook the USB end under the lips, press the
  antenna end down until it clicks. The spigot is cut back to `win_seat` (5) along the window strip so the
  acrylic blank is unchanged.
- Two display stoppers stand `disp_stop_l` = 5.5 mm off the cap face (measured gap on the real build; the model
  has 6.25), above and below the display modules, on the first display PCB's left edge, so the displays can't
  slide toward the cap. Because they rise from the face, the cap now prints **dome-down with supports under
  the dome**; the face-side features (lips, hook, pocket) print without supports in that orientation.
- The lips/latch proved too small on the first print, so a separate `mcu_brace` part clamps the board: a bar
  across it at `mcu_brace_y` (12.2 mm from the antenna end, over pad 2, clear of the buttons), two fingers on
  the PCB's bare 2 mm edge strips (0.1 mm preload), feet screwed to the cap face with 2 M2 x 6 flat heads,
  bar spanning the wire channels. It sits wholly inside the capsule; the body is unaffected (verified).
- Window: `acr_clr` = 0.3 mm thickness clearance between the front wall and the display faces (the display
  PCB rails moved back 0.3).
- The cap is fixed with 3 M2 x 6 flat-head screws through countersunk holes in the body wall (top, bottom,
  back-lower at 320 deg; straight back is the USB opening), 3 mm into the spigot (`cap_screw_x`), same module
  as the carrier screws (`radial_screws`).
- The USB-C cable exits through a 12.5 × 8.5 mm opening in the **back wall** of the body (cut through the
  cap spigot too). On the first fit the real port sat 1-2 mm further toward the displays than the modelled
  `usb_cx` (7.62), so the opening is shifted by `usb_x_adj` = 1.5 (centre x = 9.12); tune after measuring.
  Re-checked against the user's STEP (`enclosure/capsule/stl/ESP32-S3-Zero_v2-*.stp`, local only): the port
  mouth centre is 1.67 above the PCB top (model 1.62), so the offset is not in the CAD data - it comes from the
  print or the assembly. No board parts sit under the cap's lips or latch barb. Displays start `antenna_gap` (3 mm) after the board's tallest part.
- Dimensions came from Waveshare's STEP model (`ESP32-S3-Zero_v2`), resolved with an assembly-transform parser.

## Enclosure: NeoPixel ring at the seam
- Adafruit 12-LED NeoPixel Ring (PID 1643): 36.82 OD / 23.36 ID / 2.97 mm thick (from Adafruit's STEP model).
- It snaps into a 2.3 mm seat in the encoder carrier's outer face (the carrier's outer band is round): three
  arc snap latches at 30/150/270 deg, barbs flush with the face so the carrier still prints face down; the hub
  inside the ring's ID centres it. The seat adds 2.3 mm to the carrier (`plate_t` = `bulkhead` + `neo_seat`).
- The round `light_ring` diffuser (`neo_ring = true`) is 8.3 mm (5.3 + `diff_extra` 3): an open light-mixing
  chamber over the LEDs, 2 mm front and outer walls; print white/clear PETG. A 1 mm centring spigot on its
  back drops into a counterbore in the carrier hub, so it stays concentric while gluing.
- Its 3 wires (DIN, GND, 5V) go through dedicated slots in the seat floor at `neo_pad_a` (59/301/240 deg,
  measured from Adafruit's photo), which also clocks the ring; a 1 mm groove under the pad circle clears
  the solder joints. Firmware support is not written yet.
- Shaft engagement in the knob is 5.6 mm with the default 22 mm shaft (min 4 enforced by an assert).

## Enclosure: encoder carrier (serviceable right end)
- The body is an open tube. The encoder mounts on a separate `encoder_carrier` (end plate + 4 standoffs):
  screw the encoder to it on the bench (M2.5 x 5-6 self-tappers from the encoder side; pilots are blind so
  they can't reach the NeoPixel ring), slide the carrier into the body's right end (spigot keyed by the flat
  front), fix it with 3 M2 x 6 flat-head screws through countersunk holes in the body wall (back, top,
  bottom; heads flush). The top one goes into the display PCB stop block.
- Then snap the NeoPixel ring into the carrier's seat, glue the diffuser on, then fit the knob.
- Heat-set bosses moved behind the display PCB (`boss_y = 5`): in front of it they blocked the 27.94 mm PCB's
  rails. `front_offset = -4` keeps the clip screws over the monitor top. An assert guards the boss/rail clash.

## Conventions
- Units mm; enclosure axes: X along the capsule, −Y = front (display side), Z up.
- `mutex_capsule.scad` is the single source of truth for the enclosure. After changing a default, re-export the STLs/3MF.
