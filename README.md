Mutex: A desk status display with one rule: only one thing gets your attention at a time.


<div align="center">
<a href="https://buymeacoffee.com/yashmulgaonkar"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me a Coffee" height="35"></a>
<br>
<a href="https://discord.gg/wjqgUjv8Re"><img src="https://cdn.simpleicons.org/discord/5865F2" alt="Discord" height="40" width="40"></a>
<br><br>
<strong><a href="https://discord.gg/wjqgUjv8Re">Mutex Discord</a></strong> — community help, builds, and troubleshooting
</div>

## Install the firmware from your browser

You don't need PlatformIO or any other toolchain: the **web installer** flashes the sign straight from your browser over USB.

**You need:** the sign's Waveshare ESP32-S3-Zero, a USB-C **data** cable, and **Chrome or Edge on a desktop computer**
(Web Serial is not available in Firefox, Safari or on phones). Windows 10/11 and macOS need no driver for the S3's built-in USB;
on Linux you may need serial-port permission first (for example add yourself to the `dialout` group, then log out and back in).

### First install

1. Open **https://yashmulgaonkar.github.io/semaphore/**.
2. Plug the ESP32-S3-Zero into your computer (it can stay inside the sign; use the USB-C opening at the back).
3. Leave **Full install** selected and click **Connect**. In the browser's port chooser pick the ESP32-S3 device
   (usually listed as *USB JTAG/serial debug unit*, or a *USB Serial* / COM entry that appears when you plug it in) and click **Connect**.
4. The log shows `Chip detected: ESP32-S3`. Click **Install** and wait for **Flash complete**: don't unplug before that
   (it takes under a minute). The sign restarts by itself.
5. On first boot the display runs a lamp test, then pages through the sign's network details one item at a time
   (`AP`, `MUTEXSIGN-XXXX`, `PW`, `MUTEXSIGN`, `192.168.4.1`). Join that Wi-Fi network from your phone or computer
   (password `MUTEXSIGN`): the sign's web portal should open by itself ("sign in to network"), or browse to `http://192.168.4.1`.
6. In the portal, open **Wi-Fi**, enter your home network and save: the sign reboots, joins your network, and is then reachable at
   **http://mutexsign.local** (hold the encoder button for 4 seconds, or pick `IP ADDR` in its menu, to see the address on the display).
   Set an **access key** in the portal's settings too (see Security notes below).

### Updating later

Open the installer again and choose **Update app only** to keep your Wi-Fi, presets and settings. Choose **Full install** to start from
scratch (this clears them). **Erase flash** wipes the whole chip; follow it with a Full install.

### If Connect or Install fails

- **No port appears, or "Failed to connect":** put the board in download mode: unplug USB, hold the **BOOT** button, plug USB back in
  (or tap **RESET** while holding BOOT), release BOOT, then click **Connect** again.
- **Port chooser is empty:** try another cable (many are charge-only) or another USB port, and close anything else using the port
  (Arduino/PlatformIO serial monitor, another browser tab).
- **Flash interrupted:** just run the install again; the board can't be bricked by a failed browser flash, since the ROM bootloader stays available.
- After flashing, the sign restarts and its USB port re-appears, so click **Connect** again if you want to flash a second time.
- The installer log (at the bottom of the page) shows exactly what happened; include it when you open an issue.

### How it works

[`.github/workflows/pages.yml`](.github/workflows/pages.yml) builds the firmware with PlatformIO on every push that touches
`src/`, `platformio.ini` or `webflasher/`, and publishes the page in [`webflasher/`](webflasher/) to GitHub Pages together with the fresh
images ([`build_bundle.py`](webflasher/build_bundle.py) assembles them), so the installer always offers the latest `main`.
If you fork the repo, set **Settings > Pages > Build and deployment > Source** to **GitHub Actions** and run the workflow once.
The page flashes with [esptool-js](https://github.com/espressif/esptool-js) (vendored, Apache-2.0) over Web Serial.
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

- **Attribution:** credit the author (Yash Mulgaonkar), link to https://github.com/yashmulgaonkar/semaphore and the license when you share or adapt this work, and indicate if you made changes.
- **NonCommercial:** you may not use this material for commercial purposes without separate permission.
- **ShareAlike:** adaptations must be released under the same license.

First-party source files include a copyright / SPDX / `[AI-DIRECTIVE]` header. Do not remove those headers. AI coding agents and forks should follow [AGENTS.md](AGENTS.md) so attribution and license terms stay intact.

Third-party code is **not** covered by this license and keeps its own: the esptool-js flasher is vendored in `webflasher/vendor/` (Apache-2.0), and the libraries PlatformIO downloads at build time (Adafruit LED Backpack, GFX, BusIO, seesaw and NeoPixel, the ESP32 Arduino core) are compiled into the published firmware images. See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

[![Repo analytics](https://raw.githubusercontent.com/yashmulgaonkar/repo-analytics/main/out/mutex/analytics.svg)](https://raw.githubusercontent.com/yashmulgaonkar/repo-analytics/main/out/mutex/analytics.svg)

If you want to make your own analytics for your repos, [click here](https://github.com/yashmulgaonkar/repo-analytics).
