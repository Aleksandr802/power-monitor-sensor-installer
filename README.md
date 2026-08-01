# Power Monitor Sensor — ESP32-C3 Web Installer

A GitHub Pages site for flashing and updating the **Power Monitor Sensor** firmware
directly from the browser using [ESP Web Tools](https://esphome.github.io/esp-web-tools/).

Supports two board variants, selected via a picker at the top of the page:

| Board | Status | Notes |
|---|---|---|
| **ESP32-C3 Zero** | ✅ Live | Native USB (no UART bridge chip) — hold BOOT while plugging in to enter the bootloader. |
| **ESP32-C3 SuperMini** | ✅ Live | Own firmware build in `boards/supermini/`. The only board-specific code in the firmware is the status LED (`STATUS_LED_PIN` in `PowerMonitorSensor/src/main.cpp`) — SuperMini clones vary by seller, so make sure any future rebuilds target the LED wiring on the actual board you're flashing. |

---

## How to use

### First install

1. Open **https://YOUR_USERNAME.github.io/esp32-web-installer/**
2. Pick your board at the top of the page
3. Plug the board into a USB data cable
4. Click **"Install Smart Sensor"**
5. Select your device in the browser serial-port popup
6. Wait ~20–40 seconds — do not disconnect
7. The device reboots automatically when done

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
| Driver | Zero uses native USB, no driver needed. Some SuperMini clones use a CH340/CP210x USB-UART bridge chip and may need a driver. |
| OS | Windows / macOS / Linux desktop |

---

## Project structure

```
esp32-web-installer/
├── index.html                  # Installer UI (board picker + first install + OTA)
├── boards/
│   ├── zero/
│   │   ├── manifest.json       # Full flash manifest  (bootloader + partitions + firmware)
│   │   ├── manifest-ota.json   # OTA-only manifest    (firmware partition only)
│   │   ├── bootloader.bin      # ESP32-C3 bootloader  (offset 0x0000)
│   │   ├── partitions.bin      # Partition table       (offset 0x8000 / 32768)
│   │   └── firmware.bin        # Application firmware  (offset 0x10000 / 65536)
│   └── supermini/
│       ├── manifest.json       # Full flash manifest  (bootloader + partitions + firmware)
│       ├── manifest-ota.json   # OTA-only manifest    (firmware partition only)
│       ├── bootloader.bin      # ESP32-C3 bootloader  (offset 0x0000)
│       ├── partitions.bin      # Partition table       (offset 0x8000 / 32768)
│       └── firmware.bin        # Application firmware  (offset 0x10000 / 65536)
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

After a successful build, copy from `.pio/build/<env>/` into the matching board folder:

```
bootloader.bin  →  esp32-web-installer/boards/<board>/bootloader.bin
partitions.bin  →  esp32-web-installer/boards/<board>/partitions.bin
firmware.bin    →  esp32-web-installer/boards/<board>/firmware.bin
```

### Updating an existing board's firmware

Both boards are wired up the same way — just overwrite the three `.bin` files in
`boards/zero/` or `boards/supermini/` with a fresh build and commit. No `index.html`
or manifest changes needed unless you're adding a brand-new board.

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
| Firmware version | `boards/<board>/manifest.json` and `manifest-ota.json` — `"version"` field |
| Logo | `index.html` — replace the SVG in `.logo` with an `<img>` tag |
| Button text | `index.html` — "Install Smart Sensor" / "Update Firmware" |
| Chip type | `boards/<board>/manifest.json` — `"chipFamily": "ESP32-C3"` |
| Board names shown in the picker | `index.html` — `BOARD_NAMES` in the `<script>` block |

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
