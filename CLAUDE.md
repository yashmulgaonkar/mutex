# MutexSign — project context for Claude

A 12-character 14-segment "busy light" / message board that sits on top of a
work monitor. Started in a claude.ai chat; this file carries that context over.
The author also keeps private working notes (`docs/HISTORY.md`), a browser simulator and render tools locally; they are **not in this repository**
(gitignored), so a fresh clone will not have `docs/`, `simulator/` or `tools/`. Printable STL/3MF files are published on MakerWorld, not in git.

## License rules (mandatory)

This repository is **CC BY-NC-SA 4.0**. Full terms: `LICENSE`. Attribution text: `NOTICE`. The same rules also live in `AGENTS.md`.

1. Do not remove or rewrite top-of-file copyright / SPDX / `[AI-DIRECTIVE]` headers.
2. Include that exact header on new or refactored first-party source files (`/* */` form for C/C++/OpenSCAD/JavaScript/CSS, `#` for Python and YAML, `;` for `platformio.ini`, `<!-- -->` for HTML).
3. Credit Yash Mulgaonkar and https://github.com/yashmulgaonkar/mutex.
4. Remind the user that commercial use is prohibited without separate permission.
5. Third-party code keeps its own license: when you add, remove or upgrade a vendored file or a PlatformIO `lib_deps` entry, update
   `THIRD_PARTY_NOTICES.md` (and `NOTICE`/README if the summary changes). Never relicense third-party files or strip their license text.
6. `LICENSE` is the verbatim CC BY-NC-SA 4.0 legal code; do not edit it. The project copyright line lives in `NOTICE`.

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
| `webflasher/` | Browser firmware installer for GitHub Pages (esptool-js over Web Serial): `index.html`, `style.css`, `flasher.js`, `build_bundle.py` (PlatformIO output -> merged + app images + `manifest.json`), `vendor/` (esptool-js 0.5.7, Apache-2.0, third party) |
| `.github/workflows/pages.yml` | Builds the firmware (`pio run`), runs `build_bundle.py`, deploys `webflasher/` + `firmware/` to Pages. Needs Settings > Pages > Source: GitHub Actions |
| `THIRD_PARTY_NOTICES.md`, `NOTICE`, `LICENSE` | Third-party licenses, attribution text, and the verbatim CC BY-NC-SA 4.0 legal code |
| `simulator/index.html` (author's local copy, not in the repo) | Browser simulator: JS port of the firmware's effects, faces, encoder UI and LED logic |
| `enclosure/capsule/mutex_capsule.scad` | **Parametric OpenSCAD model** of the chosen "Capsule" enclosure (source of truth) |
| `enclosure/capsule/stl/` (author's local copy, not in the repo; published on MakerWorld) | Print-ready `mutex_*.stl` parts, `mutex_capsule.3mf` and the window SVG, exported from the SCAD |
| `tools/render/sdf.py`, `concepts.py` (author's local copy, not in the repo) | Tiny numpy ray-marcher used for the concept renders |
| `docs/` (author's local copy, not in the repo) | `HISTORY.md` and concept renders, colour study, model preview |

## Firmware architecture (src/main.cpp)
- Libraries: Adafruit LED Backpack, Adafruit seesaw. Uses WiFi, WebServer, ESPmDNS, Preferences from the ESP32 core.
- 14-seg font is captured from the library at boot (`buildFont()` reads `displaybuffer`).
- Frame loop every 20 ms writes `fb[NCH]` (raw segment bitmasks, `NCH` = 12) → `render()` only pushes on change.
- Modes: static, scroll, bounce, type, flip, sparkle, rain, scan, spin, blink, face.
- Face engine: auto mood/blink/gaze scheduler, yawn → sleep after `sleepMin`, wakes "surprised" on input. The face uses the middle 8 digits (offset `FO` = 2): eyes on digits 3/8, mouth on 5/6, inside the middle wing.
- Encoder UI: turn = browse presets; click = BUSY/FREE toggle; double = focus timer;
  triple = face toggle; long = settings menu; hold 4 s = IP.
- NeoPixel priority: notify flash > menu > browse preview > off-hours > timer done > timer progress > LED mode.
- Web portal at `http://mutexsign.local` (AP fallback `MUTEXSIGN-XXXX` / `MUTEXSIGN`), JSON API under `/api/*`,
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
  components facing into the capsule, USB-C toward the back. The pocket has 0.6 mm extra length at the antenna
  end (`mcu_ant_clr`; 0.1 printed too tight). Held by two fixed lips (0.6 mm reach; the real board has small parts further in) at the USB end (beside the
  receptacle), the solid pocket wall at the antenna end (takes USB plug force), wire channels along both long
  edges, and the `mcu_brace` below. (The snap latch/hook at the antenna end was dropped: it can't print in
  the cap's print orientation and the brace does the clamping.) The spigot is cut back to `win_seat` (5)
  along the window strip so the acrylic blank is unchanged.
- Two display stoppers stand `disp_stop_l` = 5.5 mm off the cap face (measured gap on the real build; the model
  has 6.25), above and below the display modules, on the first display PCB's left edge, so the displays can't
  slide toward the cap. Each has a 45 deg gusset on its front side.
- The cap prints **standing on its flat front face, USB end up** (stoppers horizontal, gussets underneath);
  add a support under the spigot's front flat (2.2 mm above the bed, 5 mm out from the dome).
- The lips/latch proved too small on the first print, so a separate `mcu_brace` part clamps the board: a bar
  across it at `mcu_brace_y` (12.2 mm from the antenna end, over pad 2, clear of the buttons), two fingers on
  the PCB's bare 2 mm edge strips (0.1 mm preload), feet screwed to the cap face with 2 M2 x 6 flat heads,
  bar spanning the wire channels. It sits wholly inside the capsule; the body is unaffected (verified).
- Window: `acr_clr` = 0.3 mm thickness clearance between the front wall and the display faces (the display
  PCB rails moved back 0.3). The window opening is shifted `win_dx` = -1 mm toward the left cap
  (`win_open_cx`); the bosses and clip stay on `win_cx`.
- Window blank is **159 × 24 × 3 mm, 1 mm corners** (`acr_len`): on the real build the encoder PCB's connector
  stops it before the body's end stop. It overlaps the opening 8 mm at the left and only just reaches the
  right edge (-0.05 mm, accepted by the user); the asserts allow that.
- All 6 body countersinks (cap + carrier screws) are sunk `car_csk_deep` = 1 mm into the 2 mm wall with a
  4 mm recess above, because the printed heads stood proud.
- The cap is fixed with 3 M2 x 6 flat-head screws through countersunk holes in the body wall (top, bottom,
  back-lower at 320 deg; straight back is the USB opening), 3 mm into the spigot (`cap_screw_x`), same module
  as the carrier screws (`radial_screws`).
- The USB-C cable exits through a 12.5 × 8.5 mm opening in the **back wall** of the body (cut through the
  cap spigot too), centred at x = 7.12 (`usb_x_adj` = -0.5 from the modelled port `usb_cx` = 7.62), fitted on
  the braced board. An earlier +1.5 shift came from the board sitting proud before the brace existed. The
  user's STEP (`enclosure/capsule/stl/ESP32-S3-Zero_v2-*.stp`, local only) puts the port mouth centre 1.67
  above the PCB top (model 1.62). No board parts sit under the cap's lips. Displays start `antenna_gap` (3 mm) after the board's tallest part.
- Dimensions came from Waveshare's STEP model (`ESP32-S3-Zero_v2`), resolved with an assembly-transform parser.

## Enclosure: NeoPixel ring at the seam
- Adafruit 12-LED NeoPixel Ring (PID 1643): 36.82 OD / 23.36 ID / 2.97 mm thick (from Adafruit's STEP model).
- It snaps into a 2.3 mm seat in the encoder carrier's outer face (the carrier's end plate and outer band are
  round, so the seat has a floor where the ring overhangs the flat front): three
  arc snap latches at 30/150/270 deg, barbs flush with the face so the carrier still prints face down; the hub
  inside the ring's ID centres it. The seat adds 2.3 mm to the carrier (`plate_t` = `bulkhead` + `neo_seat`).
- The round `light_ring` diffuser (`neo_ring = true`) is 7.5 mm (5.3 + `diff_extra` 2.2; it was 8.3, shortened
  0.8 so the encoder button can be pressed): an open light-mixing chamber over the LEDs, 2 mm front and outer
  walls; print in translucent PETG (the knob is opaque orange). It is **joined to the knob, not the carrier**: a
  plug ring of 8 arc segments on its front face (`diff_plug`: `plug_h` 2.5, `fit_clr` 0.2, crush ribs `fit_crush`
  0.35, slots straddling the knob's 8 spokes) pushes into the knob's skirt bore. The pair then friction-fits onto
  the encoder shaft and rides with it; `knob_gap` (1.6 mm) is the clearance between the carrier face and the
  diffuser's back = the push-click travel (the knob stays where it was). The 1 mm spigot on its back only guides
  it into the carrier hub's counterbore at the end of a push. **Print back face down** (spigot on the bed, plug up):
  needs build-plate supports under the outer wall and a bridged 2 mm chamber roof.
- Its 3 wires (DIN, GND, 5V) go through dedicated slots in the seat floor at `neo_pad_a` (59/301/240 deg,
  measured from Adafruit's photo), which also clocks the ring; a 1 mm groove under the pad circle clears
  the solder joints. Firmware: ring DIN on ESP32 GPIO6 (`RING_PIN`), Adafruit NeoPixel library; `RING_OFFSET` /
  `RING_REVERSE` set the 12 o'clock LED and direction. Ring mode `status` (default) follows the display
  (effect-synced animations, face mood/blink/gaze, timer arc, menu gauges, new-message burst); the other ring
  modes are standalone effects (`RING <name>` serial, `/api/ring?m=&b=`, menu RING / RING LVL, web "Light ring").
- Shaft engagement in the knob is 5.6 mm with the default 22 mm shaft (min 4 enforced by an assert).

## Enclosure: encoder carrier (serviceable right end)
- The body is an open tube. The encoder mounts on a separate `encoder_carrier` (end plate + 4 standoffs):
  screw the encoder to it on the bench (M2.5 x 5-6 self-tappers from the encoder side; pilots are blind so
  they can't reach the NeoPixel ring), slide the carrier into the body's right end (spigot keyed by the flat
  front), fix it with 3 M2 x 6 flat-head screws through countersunk holes in the body wall (back, top,
  bottom; heads flush). The top one goes into the display PCB stop block.
- Then snap the NeoPixel ring into the carrier's seat, press the diffuser onto the knob, and press that pair onto the shaft.
- The display PCB end stops on the carrier (top/bottom) stay 0.4 mm outside the encoder PCB (no overhang, so
  the encoder drops straight onto its standoffs) and catch the outer 0.87 mm of the last display PCB's edges.
- Heat-set bosses moved behind the display PCB (`boss_y = 5`): in front of it they blocked the 27.94 mm PCB's
  rails. `front_offset = -4` keeps the clip screws over the monitor top. An assert guards the boss/rail clash.

## Web installer (webflasher/)
- Single board (ESP32-S3-Zero, 4 MB, default.csv). The page fetches `./firmware/manifest.json` (own layout plus an ESP Web Tools
  `builds[]` entry) and writes either the merged image at 0x0 (clears NVS: Wi-Fi/presets/settings) or the app image at 0x10000 (keeps them).
- **esptool-js 0.5.7 does not patch image headers** (its `_updateImageFlashParams` parses the magic byte with `parseInt` on a binary
  string, so it always bails out). The flash mode/size/freq must already be right in the image: PlatformIO writes `dio`/`4MB`/`80m`
  (header bytes 0x02 0x2F) even though platformio.ini says `qio`. `build_bundle.py` checks this and fails the build if it changes; the page
  passes `"keep"`. The pure-Python merge was verified byte-identical to `esptool.py merge_bin --flash_mode dio --flash_freq 80m --flash_size 4MB`.
- The vendored bundle is byte-identical to the one on the FlightScnr installer; update by re-downloading from npm and re-verifying.
- Tested headless against a fake serial port and stub esptool (offsets, SHA-256, modes, erase), and confirmed working on real hardware by
  the author (2026-11-20) through the deployed GitHub Pages site. User-facing steps are in the README ("Install the firmware from your browser").

## Conventions
- Units mm; enclosure axes: X along the capsule, −Y = front (display side), Z up.
- `mutex_capsule.scad` is the single source of truth for the enclosure. After changing a default, re-export the STLs/3MF.
