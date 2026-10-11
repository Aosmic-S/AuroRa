

# 🌌 Aurora: Minimalist Smart Halo Lamp

> *"Clean lines. Soft light. Local control."*

**Aurora** is an open-source, DIY smart table lamp designed with a premium "Halo Tower" aesthetic. Built around the **ESP32-C3**, it features 14 addressable LEDs hidden behind a frosted diffuser, creating smooth aurora-like gradients. Unlike cloud-dependent bulbs, Aurora runs entirely locally with a custom web dashboard, REST API, and intuitive single-button gesture control.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![PlatformIO Build Status](https://img.shields.io/badge/Build-PlatformIO-blue)](.github/workflows/build.yml)
[![Hardware](https://img.shields.io/badge/Hardware-DIY-orange)](#hardware--bill-of-materials)

---

## ✨ Key Features

*   **🎨 Dynamic Lighting Engine:** Powered by FastLED, supporting solid colors, brightness ramps, and procedural "Aurora" gradient modes.
*   **📱 Local Web Dashboard:** No cloud accounts required. Connect to the lamp's WiFi AP or your home network to access a sleek, responsive HTML5 control interface hosted directly on the device.
*   **⚙️ Single-Button Gestures:** Control power, cycle presets, adjust brightness, and toggle WiFi mode using one flush-mounted metal button.
*   **🔋 Safe Power Architecture:** A dedicated 3-pin hardware switch physically separates **CHARGE** and **USE** modes, preventing battery degradation from simultaneous charging/discharging.
*   **☁️ OTA Updates:** Integrated ElegantOTA support allows wireless firmware updates via the web browser.
*   **🛠️ CI/CD Ready:** Automated GitHub Actions workflow builds full binary packs (`bootloader`, `partitions`, `firmware`) for easy flashing without local toolchain setup.

---

## 🏗️ Hardware Specifications

| Component | Specification | Role |
| :--- | :--- | :--- |
| **MCU** | ESP32-C3 DevKit | WiFi/BLE Connectivity & Logic |
| **Lighting** | 14x WS2812B LEDs | Ring Configuration (Upward Facing) |
| **Power Source** | 1800mAh Li-Ion (3.7V) | Portable Operation |
| **Charging** | TP4056 Module | Linear Charging Protection |
| **Boost Converter**| MT3608 | Steps 3.7V → 5V for Stable LED/MCU Power |
| **Diffuser** | Frosted Acrylic Disc | Light Homogenization |
| **Enclosure** | 3D Printed (PLA/PETG) | Matte White Finish |

### Physical Dimensions
*   **Total Height:** ~18 cm
*   **Base Diameter:** 9 cm
*   **Head Diameter:** 10 cm

---

## 🧾 Bill of Materials (BOM)

### Essential Electronics
*   [ ] 1x ESP32-C3 Development Board
*   [ ] 1x WS2812B LED Strip (Cut to 14 LEDs / ~23cm)
*   [ ] 1x TP4056 Charging Module (with protection circuit recommended)
*   [ ] 1x MT3608 Boost Converter Module
*   [ ] 1x 3-Pin SPDT Switch (Toggle)
*   [ ] 1x Momentary Push Button (Metal, Panel Mount)
*   [ ] 1x 1800mAh Li-Ion Battery (Flat top preferred)
*   [ ] 1x USB-C Breakout or Cable

### Passive Components & Consumables
*   [ ] 1x Electrolytic Capacitor (470µF - 1000µF, 10V+)
*   [ ] 2x Ceramic Capacitors (0.1µF)
*   [ ] 1x Series Resistor (330Ω or 470Ω) for Data Line
*   [ ] 2x Resistors (100kΩ) for Voltage Divider
*   [ ] Hookup Wires (22-24 AWG Power, 26 AWG Signal)
*   [ ] Heat Shrink Tubing
*   [ ] M2/M3 Screws & Standoffs

### Mechanical Parts
*   [ ] Frosted Acrylic Sheet/Disc (100mm diameter, 2-3mm thick)
*   [ ] Hollow Tube/Stem (20-25mm diameter)
*   [ ] Filament for 3D Printing (White PLA/PETG recommended)

---

## 💻 Software Stack

The firmware is built using **PlatformIO** and the **Arduino Framework**.

*   **Core Libraries:**
    *   `FastLED`: High-performance LED driving.
    *   `ESP Async WebServer`: Non-blocking HTTP server for the dashboard.
    *   `ElegantOTA`: Over-the-air update management.
    *   `Preferences`: NVS storage for WiFi credentials and settings.
*   **Architecture:** Modular C++ classes handling Buttons, LEDs, Power Monitoring, and Networking independently.

### REST API Endpoints
Interact with Aurora programmatically:

```http
GET  /api/state       # Returns JSON: {power, brightness, color, battery}
POST /api/power       # Body: {"on": true/false}
POST /api/color       # Body: {"r": 255, "g": 0, "b": 100}
POST /api/brightness  # Body: {"value": 128}
GET  /api/battery     # Returns voltage and percentage
```

---

## 🔌 Wiring Diagram Overview

*(Refer to `docs/wiring.png` if available, otherwise follow logic below)*

1.  **Battery (+)** → **Switch Common Pin**
2.  **Switch Pin A** → **TP4056 B+** (Charge Mode)
3.  **Switch Pin B** → **MT3608 VIN+** (Use Mode)
4.  **Battery (-)** → **Common Ground** (TP4056 B-, MT3608 GND, ESP32 GND)
5.  **MT3608 VOUT+** → **5V Rail** (Feeds ESP32 5V/VIN and First LED)
6.  **ESP32 GPIO** → **Resistor (330Ω)** → **First LED DIN**
7.  **Capacitor (470µF)** → Across 5V/GND near first LED.

⚠️ **Safety Rule:** The 3-pin switch ensures you cannot charge while discharging. Never bypass this switch.

---

## 🚀 Getting Started

### Option 1: Flash Pre-built Binaries (Recommended for Beginners)
1.  Go to the **[Releases Page](../../releases)**.
2.  Download the latest zip file containing:
    *   `bootloader.bin`
    *   `partitions.bin`
    *   `boot_app0.bin`
    *   `firmware.bin` (Includes embedded Web Dashboard)
3.  Use [ESP Web Flash Tool](https://espressif.github.io/esptool-js/) or `esptool.py`.
4.  Flash at these addresses:
    ```bash
    esptool.py --chip esp32c3 write_flash \
      0x0 bootloader.bin \
      0x8000 partitions.bin \
      0xe000 boot_app0.bin \
      0x10000 firmware.bin
    ```

### Option 2: Compile from Source (For Developers)
1.  Install [VS Code](https://code.visualstudio.com/) and the [PlatformIO Extension](https://platformio.org/install/ide?install=vscode).
2.  Clone this repository:
    ```bash
    git clone https://github.com/Aosmic-S/AuroRa.git
    cd AuroRa
    ```
3.  Open the folder in VS Code. PlatformIO will automatically detect dependencies.
4.  Click **"Upload"** (the arrow icon) in the bottom bar.
5.  Monitor serial output (115200 baud) to get the initial IP address.

---

## 🕹️ User Guide

### Initial Setup
1.  Power on the lamp (Set switch to **USE**).
2.  Look for WiFi Network: `Aurora-Setup`.
3.  Password: `password123` (Default).
4.  Connect your phone/laptop to this AP.
5.  Navigate to `http://192.168.4.1` to configure your Home WiFi SSID/Password.
6.  Restart the lamp. It will now appear on your local network as `http://aurora.local`.

### Button Gestures
| Gesture | Action |
| :--- | :--- |
| **Short Press** | Toggle On/Off |
| **Double Press** | Cycle Color Presets |
| **Long Press (>1s)** | Ramp Brightness Up/Down |
| **Very Long Press (5s)** | Toggle WiFi Mode (AP ↔ Station) |
| **Hold 10s** | Factory Reset (Clears WiFi creds) |

---

## 🏭 Assembly Instructions

1.  **Electronics Prep:** Solder headers to ESP32, mount components on perfboard or PCB. Test power rails with multimeter before connecting LEDs.
2.  **LED Ring:** Cut WS2812 strip to 14 segments. Solder data/power/gnd. Glue into circular mold or print a ring holder.
3.  **Stem & Base:** Print base and stem. Insert electronics into base. Route wires through hollow stem.
4.  **Head Assembly:** Mount LED ring inside head housing. Place frosted acrylic disc on top (leave 10-15mm air gap for diffusion).
5.  **Final Touches:** Spray paint matte white if desired. Ensure no screws are visible externally.

---

## 🤝 Contributing

Contributions are welcome! Please fork the repository and submit a Pull Request.
1.  Fork the Project
2.  Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3.  Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4.  Push to the Branch (`git push origin feature/AmazingFeature`)
5.  Open a Pull Request

---

## 📄 License

This project is licensed under the **Apache License 2.0**.

See the [LICENSE](LICENSE) file for details. This means you are free to use, modify, and distribute this code (even commercially), provided you retain the original copyright notice and license text.

---

## 🙏 Acknowledgments

*   **FastLED Team** for the incredible library performance.
*   **Me-no-dev** for ESPAsyncWebServer.
*   **Ayush Sharma** for ElegantOTA.
*   The Maker Community for inspiration on minimalist industrial design.

---

<div align="center">
  <sub>Built with ❤️ and ☕ by Aosmic</sub>
</div>
