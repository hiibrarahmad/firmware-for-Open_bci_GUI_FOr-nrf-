# OpenBCI-Compatible Firmware & Tools for XIAO nRF52840 + ADS1299

This repository offers **firmware** and **Python utilities** to bridge a **Seeed Studio XIAO nRF52840 Sense** with a **Texas Instruments ADS1299** (8-ch EEG front-end). It enables real-time EEG streaming in **OpenBCI Cyton** format, making it compatible with the [OpenBCI GUI](https://github.com/OpenBCI/OpenBCI_GUI) and custom analysis pipelines.

---

## 🚀 Features

- Stream up to 8-channel, 24-bit EEG data from ADS1299
- OpenBCI Cyton-style packet (33 bytes per sample)
- Out-of-the-box compatibility with OpenBCI GUI
- Python tools for:
  - Serial monitoring & hex inspection
  - Packet integrity validation
  - Data logging (CSV)
  - Live plotting (Matplotlib)
  - BrainFlow-based filtering & band power

---

## 🧠 Hardware Requirements

| Component                      | Notes                                      |
| ------------------------------ | ------------------------------------------ |
| **Seeed Studio XIAO nRF52840** | MCU (nRF52840 SoC), Arduino-compatible     |
| **ADS1299 (TI)**               | 24-bit EEG analog front end, SPI interface |
| **Electrodes + Cap**           | For EEG signal acquisition                 |
| **Wiring, breadboard, or PCB** | For interconnections                       |

---

## 📌 Pin Mapping

| XIAO nRF52840 Pin | ADS1299 Signal | Description                       |
| ----------------- | -------------- | --------------------------------- |
| D1                | DRDY           | Data ready interrupt (active low) |
| D2                | RESET          | Reset ADS1299                     |
| D3                | START          | Start/stop conversions            |
| D4                | CS (SS)        | SPI chip select                   |
| D5                | SCLK           | SPI clock                         |
| D6                | MOSI           | SPI master out, slave in          |
| D7                | MISO           | SPI master in, slave out          |
| 3.3V / GND        | VDD / GND      | Power supply                      |

> **Note:** `DRDY` is a dedicated interrupt (not SPI) and is essential for correct timing.

---

## ⚡ Packet Format

Each data packet = **33 bytes**:

| Byte(s) | Purpose                              |
| ------- | ------------------------------------ |
| 0       | Start byte (`0xA0`)                  |
| 1       | Sample counter (0–255)               |
| 2–25    | EEG data (8 channels × 3 bytes)      |
| 26–31   | Aux data (accelerometer/reserved)    |
| 32      | Stop byte (`0xC0`)                   |

- EEG: 24-bit signed, MSB first
- Mirrors OpenBCI Cyton serial stream

---

## 🛠 Directory Structure

```
.
├── latest xaio code/
│   └── nrf52480.ino                 # Arduino sketch for XIAO + ADS1299
│
├── OpenBCI_32bit_Library/           # Custom OpenBCI driver (header + cpp)
│   ├── OpenBCI_32bit_Library.h
│   └── OpenBCI_32bit_Library.cpp
│
├── *.py                             # Python tools
│   ├── dataformat.py                # Hex monitor for serial stream
│   ├── dataformatchec.py            # Packet checker
│   ├── readdata.py                  # Serial to CSV logger
│   ├── graph.py                     # Live plotting from CSV
│   └── vrainflow.py                 # BrainFlow filtering & band power
```

---

## 🐍 Python Toolchain Overview

- **`dataformat.py`**  
  Hex dump of incoming serial packets; quick debugging.

- **`dataformatchec.py`**  
  Checks packet start/stop bytes; warns on errors.

- **`readdata.py`**  
  Logs serial stream to CSV (expand for real-time analysis).

- **`graph.py`**  
  Live plot up to 4 EEG channels from CSV (Tkinter + Matplotlib).

- **`vrainflow.py`**  
  Uses [BrainFlow](https://brainflow.org/) for DSP (bandpass, notch, band power visualization).

**Dependencies:**  
`pyserial`, `matplotlib`, `brainflow` (install via pip).

---

## ⚙️ Setup Instructions

### 1️⃣ Upload Firmware

1. Open `latest xaio code/nrf52480.ino` in Arduino IDE.
2. Board: **Seeed XIAO nRF52840 Sense**.
3. Install dependencies (e.g., `SPI` library).
4. Upload.

### 2️⃣ Hardware Wiring

- Wire XIAO ↔ ADS1299 per **Pin Mapping** above.
- Common GND essential. Use clean 3.3V for ADS1299.

### 3️⃣ Run Python Tools

1. Connect board via USB (115200 baud default).
2. Edit scripts for your serial port (`COM6`, `/dev/ttyACM0`, etc).
3. Log data:
   ```bash
   python readdata.py
   ```
4. Live plot:
   ```bash
   python graph.py
   ```

---

## ⚠️ Notes

- **Data rate:** 250–500 Hz typical (ADS1299 setting + firmware).
- **DRDY:** Use interrupt, not polling.
- **Tested:** Python 3.8+, Windows & Linux.
- **Dependencies:** `pyserial`, `matplotlib`, `brainflow`.

---

## 🤝 Contributing

PRs welcome!  
For new features (e.g., BLE streaming, SD logging):

- Follow code style
- Document thoroughly
- Test with OpenBCI GUI

---

## 📄 License

MIT License — free for all uses.

---

## 🔗 Credits

- Inspired by OpenBCI Cyton hardware/stream
- Uses TI ADS1299 & Seeed XIAO nRF52840 Sense
- Python DSP via BrainFlow ([https://brainflow.org](https://brainflow.org/))

---

## 📌 Example Wiring Diagram (ASCII)

```text
┌───────────────────────────────────┐
│    XIAO nRF52840 Sense           │
│                                   │
│  D1 ── DRDY  ──┐                  │
│  D2 ── RESET  ─┤                  │
│  D3 ── START  ─┤                  │
│  D4 ── CS     ─┼─── ADS1299       │
│  D5 ── SCLK   ─┤                  │
│  D6 ── MOSI   ─┤                  │
│  D7 ── MISO   ─┘                  │
│  3.3V ── VDD                      │
│  GND  ── GND                      │
└───────────────────────────────────┘
```
