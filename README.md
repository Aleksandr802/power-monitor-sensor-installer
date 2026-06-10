# Power Monitor Sensor — ESP32-C3 Super Mini Web Installer

A GitHub Pages site for flashing and updating the **Power Monitor Sensor** firmware
directly from the browser using [ESP Web Tools](https://esphome.github.io/esp-web-tools/).

---

## How to use

### First install

1. Open **https://YOUR_USERNAME.github.io/esp32-web-installer/**
2. Plug your **ESP32-C3 Super Mini** into a USB data cable
3. Click **"Install Smart Sensor"**
4. Select your device in the browser serial-port popup
5. Wait ~20–40 seconds — do not disconnect
6. The device reboots automatically when done

### Firmware update (OTA)

**Option A — USB update (same cable, no WiFi needed)**

1. Go to the **Firmware Update (OTA)** tab on the installer page
2. Click **"Update Firmware"**
3. Select your device and wait

**Option B — WiFi OTA (device must be connected to WiFi)**

1. Go to the **Firmware Update (OTA)** tab
2. Enter the device's local IP address
3. Click **"Open Device Page"** — this opens `http://<ip>/update`
4. Upload `firmware.bin` through the browser form

---

## Requirements

| Requirement | Detail |
|---|---|
| Browser | Chrome 89+ or Edge 89+ on desktop |
| Cable | USB **data** cable (not charge-only) |
| Driver | CH340 / CH340G (ESP32-C3 Super Mini) |
| OS | Windows / macOS / Linux desktop |

---

## Project structure

```
esp32-web-installer/
├── index.html          # Installer UI (first install + OTA)
├── manifest.json       # Full flash manifest  (bootloader + partitions + firmware)
├── manifest-ota.json   # OTA-only manifest    (firmware partition only)
├── bootloader.bin      # ESP32-C3 bootloader  (offset 0x0000)
├── partitions.bin      # Partition table       (offset 0x8000 / 32768)
├── firmware.bin        # Application firmware  (offset 0x10000 / 65536)
└── README.md
```

---

## Firmware binary offsets (ESP32-C3)

| File | Hex offset | Decimal offset |
|---|---|---|
| `bootloader.bin`  | `0x0000` | 0      |
| `partitions.bin`  | `0x8000` | 32768  |
| `firmware.bin`    | `0x10000`| 65536  |

> **Note:** ESP32-C3 bootloader lives at `0x0000`, unlike classic ESP32 which uses `0x1000`.

---

## Adding your firmware binaries

### From PlatformIO

After a successful build, copy from `.pio/build/<env>/`:

```
bootloader.bin  →  esp32-web-installer/bootloader.bin
partitions.bin  →  esp32-web-installer/partitions.bin
firmware.bin    →  esp32-web-installer/firmware.bin
```

### From Arduino IDE

Export compiled binary (**Sketch → Export Compiled Binary**), then use
`esptool.py` to extract the three parts at the correct offsets.

### From esptool (manual export)

```bash
esptool.py --chip esp32c3 read_flash 0x00000  0x7000   bootloader.bin
esptool.py --chip esp32c3 read_flash 0x08000  0x1000   partitions.bin
esptool.py --chip esp32c3 read_flash 0x10000  0x100000 firmware.bin
```

---

## Deploy to GitHub Pages

1. Create a new GitHub repository named `esp32-web-installer`
2. Push all files to the `main` branch
3. Go to **Settings → Pages**
4. Set: Source = `main`, Folder = `/ (root)`
5. Your installer will be live at:
   ```
   https://YOUR_USERNAME.github.io/esp32-web-installer/
   ```

---

## Customisation

| What to change | Where |
|---|---|
| Product name | `index.html` — search for "Power Monitor Sensor" |
| Firmware version | `manifest.json` and `manifest-ota.json` — `"version"` field |
| Logo | `index.html` — replace the SVG in `.logo` with an `<img>` tag |
| Button text | `index.html` — "Install Smart Sensor" / "Update Firmware" |
| Chip type | `manifest.json` — `"chipFamily": "ESP32-C3"` |

---

## Troubleshooting

**Serial port not found**
- Use a different USB cable (data cables only)
- Install CH340 driver: https://www.wch-ic.com/downloads/CH341SER_EXE.html
- Linux: `sudo usermod -aG dialout $USER` then re-login

**Flashing fails or freezes**
- Hold the **BOOT button** while clicking Install, release after progress starts
- Close Arduino IDE or any serial monitor before flashing

**Device doesn't respond after flashing**
- Press the **RST button** to reboot
- Open serial monitor at 115200 baud to check boot output

---

## License

MIT — free to use, modify, and redistribute.
