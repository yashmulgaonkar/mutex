# mutex
Mutex: A desk status display with one rule: only one thing gets your attention at a time.

## Install the firmware from your browser

Open the **web installer** at https://yashmulgaonkar.github.io/mutex/ in Chrome or Edge on a desktop computer, plug in the sign's
ESP32-S3-Zero over USB, click **Connect**, then **Install**. *Full install* writes everything (and clears Wi-Fi, presets and settings);
*Update app only* keeps them. No toolchain needed. If Connect fails, hold **BOOT**, replug USB (or tap **RESET**), release BOOT and try again.

How it works: [`.github/workflows/pages.yml`](.github/workflows/pages.yml) builds the firmware with PlatformIO on every push that touches
`src/`, `platformio.ini` or `webflasher/`, and publishes the page in [`webflasher/`](webflasher/) to GitHub Pages together with the fresh
images ([`build_bundle.py`](webflasher/build_bundle.py) assembles them). One-time setup: repository **Settings > Pages > Build and deployment >
Source: GitHub Actions**. The page flashes with [esptool-js](https://github.com/espressif/esptool-js) (vendored, Apache-2.0) over Web Serial.
Building from source with PlatformIO (`pio run -t upload`) still works as before.

## Enclosure

The enclosure is a parametric OpenSCAD model, [`enclosure/capsule/mutex_capsule.scad`](enclosure/capsule/mutex_capsule.scad) (the source of truth:
measure your parts and adjust the values marked "measure"). Ready-to-print **STL and 3MF files are published on MakerWorld**, not in this repository.
They are generated from the SCAD file and carry the same license (CC BY-NC-SA 4.0), so choose the matching license when you share them.

## Security notes

- Until you change it, the sign's own Wi-Fi access point (used for first setup and when no home Wi-Fi is saved) is named `MUTEXSIGN-XXXX`
  with the **publicly documented default password `MUTEXSIGN`**, so anyone within range can join it. Save your home Wi-Fi in the web portal
  (the sign then stops using the access point) and **set an access key** in the portal's settings.
- The web portal and its `/api/*` endpoints use plain HTTP and have **no authentication until you set an access key**. Keep the sign on a trusted
  network and do not expose it to the internet.

## License

All original firmware, enclosure models (OpenSCAD source and exported files), tools and documentation in this repository are licensed under **[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/)** (`SPDX-License-Identifier: CC-BY-NC-SA-4.0`, see [LICENSE](LICENSE)). Required attribution text is in [NOTICE](NOTICE).

- **Attribution:** credit the author (Yash Mulgaonkar), link to https://github.com/yashmulgaonkar/mutex and the license when you share or adapt this work, and indicate if you made changes.
- **NonCommercial:** you may not use this material for commercial purposes without separate permission.
- **ShareAlike:** adaptations must be released under the same license.

First-party source files include a copyright / SPDX / `[AI-DIRECTIVE]` header. Do not remove those headers. AI coding agents and forks should follow [AGENTS.md](AGENTS.md) so attribution and license terms stay intact.

Third-party code is **not** covered by this license and keeps its own: the esptool-js flasher is vendored in `webflasher/vendor/` (Apache-2.0), and the libraries PlatformIO downloads at build time (Adafruit LED Backpack, GFX, BusIO, seesaw and NeoPixel, the ESP32 Arduino core) are compiled into the published firmware images. See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

[![Repo analytics](https://raw.githubusercontent.com/yashmulgaonkar/repo-analytics/main/out/mutex/analytics.svg)](https://raw.githubusercontent.com/yashmulgaonkar/repo-analytics/main/out/mutex/analytics.svg)

If you want to make your own analytics for your repos, [click here](https://github.com/yashmulgaonkar/repo-analytics).
