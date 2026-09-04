# Display Configuration & HiDPI Guide

This guide documents how the internal display is configured in this EFI, how native Apple **"Color LCD"** identification was activated via EDID injection, and how to achieve MacBook-style **HiDPI Retina scaling** on the 1080p panel using BetterDisplay.

---

## Hardware Overview

* **Laptop Model:** HP Pavilion Gaming Laptop 15-DK0XXX
* **Internal Panel:** LG Display (`LGD062E`, 15.6" IPS)
* **Native Resolution:** 1920 × 1080 @ 60Hz / 40Hz
* **Interface:** eDP (connected to Intel UHD Graphics 630 at `PciRoot(0x0)/Pci(0x2,0x0)`)
* **Pixel Density:** ~141 PPI (Standard / Non-Retina)

---

## 1. The Initial Issue: "Unknown Display"

By default in macOS, the display settings showed:
* Color profile: **"Unknown Display"**
* Display metadata: Generic / unbranded monitor

### Root Cause (EDID Inspection)
Dumping the panel's 128-byte EDID via `ioreg -lw0 | grep "IODisplayEDID"` revealed:
1. **Manufacturer ID:** Bytes 8–9 were `30e4` (LG Display `LGD`). macOS has no built-in color calibration tables or display profiles for third-party OEM PC panels.
2. **Missing Descriptor:** Descriptor Block 3 (offset 90–107) was completely unprogrammed (18 blank bytes of zeroes: `00000000...`). Because the screen firmware did not include an ASCII Monitor Name descriptor (`tag: 0xFC`), macOS had no string to read and defaulted to "Unknown Display".

---

## 2. Activating Native Apple "Color LCD" via EFI

Instead of using deprecated system-level patch scripts, the issue was solved natively in OpenCore through **EDID Injection** via WhateverGreen.

### The Patch Applied
1. **Preserved Physical Timings:** Detailed timing descriptors 1 and 2 (1920×1080 @ 60Hz and 40Hz) were kept 100% untouched to ensure zero signal disruption or black screen risk.
2. **Apple Vendor ID:** Bytes 8–9 changed from `30e4` (`LGD`) to `0610` (`APP` / Apple).
3. **Injected Monitor Name:** Replaced the 18 blank zeroes in Descriptor 3 with an official Apple ASCII monitor name descriptor (`tag: 0xFC`):
   ```
   00 00 00 fc 00 43 6f 6c 6f 72 20 4c 43 44 0a 20 20 20  ("Color LCD\n   ")
   ```
4. **Recalculated Checksum:** Byte 127 updated to `0xD5` (`sum % 256 == 0`, valid).
5. **Injected into OpenCore:** Added `AAPL00,override-no-connect` to `EFI/OC/config.plist` under `PciRoot(0x0)/Pci(0x2,0x0)`:
   ```xml
   <key>AAPL00,override-no-connect</key>
   <data>AP///////wAGEC4GAAAAAAAdAQSVIhN4AzjVl15ZjiccUFQAAAABAQEBAQEBAQEBAQEBAQEBJDaAoHA4H0AwIDUAWMIQAAAZGCSAoHA4H0AwIDUAWMIQAAAZAAAA/ABDb2xvciBMQ0QKICAgAAAAAgAMOv8KPH0TFCZ9AAAAANU=</data>
   ```

### Results in macOS
* macOS identifies the screen as **"Built-in Display"** with an Apple laptop icon.
* Under **Color Profile**, it natively selects and assigns **"Color LCD"**.
* 100% compliant with **System Integrity Protection (SIP Enabled)**.
* Survives all macOS updates without re-patching.

---

## 3. Why macOS Shows a Resolution List Instead of 5 Mac Scaling Tiles

On a genuine MacBook Pro, System Settings shows 5 scaling tiles (*Larger Text*, *Default*, *More Space*). On a 1080p Hackintosh, it shows:
* `1920 x 1080 (Default)`
* `1600 x 900`
* `1344 x 756`

### The Reason
Apple reserves the 5 scaling tiles exclusively for high-density **Retina displays (~220+ PPI)** where macOS renders the entire UI at **2x HiDPI** and downsamples. Because 1080p on a 15.6" panel is ~141 PPI, macOS classifies it as a standard (1x) display and hides the Retina scaling tiles (the same behavior as an external 1080p monitor on a real Mac).

---

## 4. Enabling MacBook Pro HiDPI Scaling (BetterDisplay)

> [!WARNING]
> Do **NOT** use legacy scripts such as `one-key-hidpi`. These 2017/2018 scripts are incompatible with modern macOS Signed System Volume (SSV) security and can cause boot failures.

The modern standard to unlock Retina HiDPI scaling is **[BetterDisplay](https://github.com/waydabber/BetterDisplay)**.

### Method A: Flexible Scaling (Native UI Slider)
1. Install BetterDisplay:
   ```bash
   brew install --cask betterdisplay
   ```
2. Open BetterDisplay **Settings (gear icon) > Displays**.
3. Select your internal display (`Color LCD`).
4. Toggle on **"Edit the default system configuration of this display model"**.
5. Toggle on **"Enable flexible scaling"**.
6. Click **Apply** (red button) and reboot your laptop.
7. After rebooting, you can use the smooth resolution scaling slider in BetterDisplay to scale UI elements crisply.

### Method B: Virtual Screen (Dummy Mirror)
If macOS locks the system configuration of the internal panel:
1. Open BetterDisplay **Settings > Virtual Screens**.
2. Click **Create New Virtual Screen**:
   * **Aspect Ratio:** `16:9`
   * **Resolution:** `1920x1080` (or `2560x1440` / `3840x2160` HiDPI).
3. In macOS **System Settings > Displays**, set your physical display to **Mirror** the Virtual Screen.
4. macOS will render at high-resolution 2x HiDPI and downscale cleanly to the 1080p panel, giving razor-sharp text rendering.
