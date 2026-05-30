
# VENDETTA FPV Stack

> A fully open-source, custom-designed 4-in-1 ESC and F7 Flight Controller stack for FPV multirotors, designed from scratch in KiCad.

![Board Status](https://img.shields.io/badge/status-in%20development-orange)
![KiCad](https://img.shields.io/badge/EDA-KiCad%208-blue)
![License](https://img.shields.io/badge/license-CERN--OHL--S-green)

---

## Overview

The VENDETTA stack consists of two boards designed to work together as a matched pair, built around modern components and open firmware with no proprietary dependencies.

| Board | Folder | Status |
|---|---|---|
| VENDETTA ESC (VESC) | `/VESC` | Schematic complete, layout WIP |
| VENDETTA FC (VFC) | `/VFC` | Schematic complete, layout WIP |

---

## VENDETTA ESC (VESC)

A 30×30mm 4-in-1 ESC (25A x 4) designed for 3–6S FPV multirotors.

<img width="2526" height="3451" alt="imagev2" src="https://github.com/user-attachments/assets/bde25ee8-3351-4142-8fb1-18407447f4bf" />


> Note: 6 Layer Board, GND (plane 2 and 5) plane photos not included.

### Specifications

| Parameter | Value |
|---|---|
| MCU | Artery AT32F421GBU7 |
| Gate Driver | NSG2065Q |
| MOSFETs | SP40N03GNJ (×6 per channel, 24 total) |
| Continuous current | 25A per channel |
| Burst current | 30A per channel |
| Input voltage | 3–6S LiPo (12.6V–25.2V) |
| Motor outputs | 4× 3-phase BLDC |
| Firmware | AM32 |
| Protocols | DShot150 / DShot300 / DShot600 |
| Telemetry | Bidirectional DShot (RPM) |
| Current sensing | INA185 + 0.5mΩ Kelvin shunt |
| TVS protection | SMBJ30A |
| Bulk capacitance | 320µF (32× 10µF MLCC) |
| BEC (8V) | MP9943GQ-Z |
| BEC (3.3V) | TLV76733 |
| Mounting | 30×30mm M3 |
| Connector | JST SH 1.0mm 8-pin (SM08B-SRSS-TB) |

### ESC Block Diagram

```
LiPo ──→ TVS + Bulk Caps ──→ 4× Half-Bridge (NSG2065Q + SP40N03GNJ)
                           ──→ 8V BEC ──→ Gate Driver VCC
                           ──→ Shunt (0.5mΩ Kelvin) ──→ INA185 ──→ AT32F421
AT32F421 ──→ DShot (from FC) ──→ PWM generation ──→ NSG2065Q ──→ Motors
```

---

## VENDETTA FC (VFC)

A 30×30mm F7 Flight Controller targeting feature parity with commercial boards like the SpeedyBee F7 V3, with improvements in IMU performance and power isolation.

<img width="2763" height="3492" alt="imagev2" src="https://github.com/user-attachments/assets/044c504c-7cdd-4e0b-b641-fed1b171914d" />


> Note: 6 Layer Board, GND (plane 2 and 5) plane photos not included.

### Specifications

| Parameter | Value |
|---|---|
| MCU | STM32F722RETx (216MHz Cortex-M4) |
| IMU | ICM-42688-P (32kHz gyro, SPI) |
| Barometer | BMP388 (I2C) |
| Blackbox | W25N04KVZEIR (512MB NAND Flash, SPI) |
| Wireless | ESP32-C3 (WiFi + BT via UART1) |
| OSD | None (digital OSD via MSP DisplayPort) |
| Firmware | Betaflight |
| Motor outputs | 4× DShot (TIM2 CH1–CH4, PA0–PA3) |
| UARTs | 6× (UART3 OSD, UART4 DJI RX, UART5 ELRS, UART6 GPS, UART7 spare) |
| I2C | 1× (GPS compass) |
| USB | USB-C (USB 2.0 FS) |
| BEC (9V/2A) | MP9943GQ-Z |
| BEC (5V/3A) | MP9943GQ-Z |
| LDO Digital (3.3V/1A) | AP7361C |
| LDO Analog (3.3V/700mA) | NCP167AMX330 |
| Blackbox | 512MB onboard flash |
| DJI support | Air Unit / O3 via UART3+UART4 |
| GPS | UART7 + I2C (compass) |
| Receiver | UART5 (ELRS/CRSF/SBUS) |
| LED outputs | 4× |
| Buzzer | MOSFET driven |
| Mounting | 30×30mm M3 |

### FC Block Diagram

```
                    ┌─────────────────────────────┐
LiPo (via ESC) ────→│ 9V BEC → 5V BEC → 3.3V x2  │
                    │                             │
                    │  STM32F722 ←─SPI─→ ICM-42688-P (IMU)
                    │             ←─SPI─→ W25N04KV (Flash)
                    │             ←─UART1─→ ESP32-C3 (WiFi/BT)
                    │             ←─I2C─→ BMP388 (Baro)
                    │             ←─USB─→ USB-C
                    │             ──UART3/4──→ DJI Air Unit
                    │             ──UART5────→ Receiver (ELRS)
                    │             ──UART6────→ GPS
                    │             ──TIM2─────→ ESC DShot M1-M4
                    └─────────────────────────────┘
```

### Power Architecture

```
VBAT (from ESC 8-pin) ──→ 9V/2A (MP9943) ──→ DJI Air Unit
                                           ──→ 5V/3A (MP9943) ──→ JST 5V outputs
                                                               ──→ 3.3V Digital (AP7361C) ──→ STM32, ESP32, Flash
                                                               ──→ [Ferrite] ──→ 3.3V Analog (NCP167) ──→ IMU, Baro, VDDA
```

---

## Connector Pinout

### ESC → FC (JST SH 1.0mm 8-pin)

| Pin | Signal | Description |
|---|---|---|
| 1 | VBAT | Battery voltage |
| 2 | GND | Ground |
| 3 | M1 | Motor 1 DShot |
| 4 | M2 | Motor 2 DShot |
| 5 | M3 | Motor 3 DShot |
| 6 | M4 | Motor 4 DShot |
| 7 | CURR | Current sense (INA185 output) |
| 8 | TLM | ESC telemetry UART |

### FC Bottom Connectors (JST SH 1.0mm)

| Connector | Pins | Function |
|---|---|---|
| ESC | 8 | M1–M4, CURR, TLM, VBAT, GND |
| RECEIVER | 4 | +5V, GND, UART5_RX, UART5_TX |
| GPS | 6 | +5V, GND, UART6_TX, UART6_RX, SDA, SCL |
| DJI AIR UNIT | 6 | +9V, GND, UART3_TX, UART3_RX, UART4_RX, NC |
| UART5 | 4 | +5V, GND, TX, RX |
| LED 1–4 | 3 each | +5V, GND, Signal |
| BUZZER | 2 | +5V, Signal |

---

## Firmware (Not yet Completed)

### ESC — AM32

AM32 is the only supported firmware for the AT32F421 MCU.

Flash via DShot passthrough from Betaflight:
```
Betaflight Configurator → Motors → ESC Configuration → Flash All
```

Or via SWD using the exposed test pads (SWDIO/SWDCLK).

Recommended target: `AT32F421_VESC` — confirm correct pin mapping before flashing.

### FC — Betaflight

Target: `STM32F7X2` unified target or a custom target for this board.

Flash via USB DFU:
1. Hold BOOT button, plug USB-C
2. Open Betaflight Configurator → Firmware Flasher
3. Select STM32F7X2, flash

Or wirelessly via ESP32-C3 over WiFi through the SpeedyBee App.

---

## Repository Structure

```
VENDETTAFC/
├── VESC/
│   ├── VESC.kicad_pro
│   ├── VESC.kicad_sch
│   ├── ESC1.kicad_sch        # Per-channel ESC subsheet
│   └── ...
├── VFC/
│   ├── VENDETTAFC.kicad_pro
│   ├── VENDETTAFC.kicad_sch  # Top-level schematic
│   ├── interface.kicad_sch   # Connectors and I/O
│   ├── pwr.kicad_sch         # Power management
│   ├── sensor.kicad_sch      # IMU, baro, flash, ESP32
│   └── ...
└── README.md
```

---

## Design Files

Both boards are designed in **KiCad 10**. All schematic symbols and footprints use standard KiCad libraries plus LCSC-sourced footprints where needed.

To open:
```
KiCad 10 → Open Project → select .kicad_pro file
```

---

## Status

- [x] ESC schematic complete
- [x] FC schematic complete  
- [x] ESC PCB layout
- [x] FC PCB layout
- [x] Gerber generation
- [ ] First prototype order
- [ ] Firmware bringup
- [ ] Testing and validation

---

## License

Hardware design files are released under the **CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S v2)**.

You are free to study, modify, distribute, and manufacture this hardware, provided derivatives are released under the same license with attribution.

---

## Author

Designed by **Caped-Crussader**

> Contributions, bug reports, and suggestions welcome via Issues.
