# MiBeesFly [![Wiki](https://img.shields.io/badge/📖%20MORE%20INFORMATION-ON%20THE%20WIKI-2EA043?style=plastic&labelColor=0D0D0D&logo=github&logoColor=white)](https://github.com/mohanan02/MiBeesFly/wiki)
 ![Status](https://img.shields.io/badge/STATUS-WORK%20IN%20PROGRESS-E60026?style=plastic&labelColor=0D0D0D&logo=github&logoColor=white)

![MCU](https://img.shields.io/badge/MCU-STM32F767ZI-03234B?style=flat&logo=stmicroelectronics&logoColor=white)
![EDA](https://img.shields.io/badge/EDA-KiCad-314CB0?style=flat&logo=kicad&logoColor=white)
![Firmware](https://img.shields.io/badge/Firmware-C%20%7C%20STM32CubeIDE-00599C?style=flat&logo=c&logoColor=white)
![IMU](https://img.shields.io/badge/IMU-BMI323-5A2D82?style=flat)
![Airframe](https://img.shields.io/badge/Airframe-450%20class%20quad-444444?style=flat)
![Power](https://img.shields.io/badge/Power-4S%20LiPo%20%7C%205200%20mAh-E8730C?style=flat)

A custom flight controller built from scratch — a **KiCad-designed carrier module** around an **STM32 NUCLEO-F767ZI**, plus **C firmware written in STM32CubeIDE** — flying a 450-class quadcopter. GPS, compass, optical flow, lidar and barometer are fused so the aircraft holds position without drift, indoors and out.

> **⚠️ Work in progress.** This is an active, early-stage personal project. Nothing here is flight-proven yet. The architecture is defined; the schematic, the carrier PCB and the firmware are all still ahead. Treat everything below as the current intent, not a finished product.

📐 **Full system architecture:** [`(1)HIghLevelDesign&(NOTES)/Architecture.md`](%281%29HIghLevelDesign%26%28NOTES%29/Architecture.md)
📋 **Work tracking:** [Issues](https://github.com/mohanan02/MiBeesFly/issues) · [Milestones](https://github.com/mohanan02/MiBeesFly/milestones)

---

## Why build the controller instead of buying one

Off-the-shelf flight controllers (Matek, Pixhawk, SpeedyBee) are cheap and work well, so this project is not about saving money. It's about owning the whole stack end to end: schematic capture, PCB layout, sensor driver code, the fusion filter and the control loops. The goal is a controller I fully understand, where every sensor reading and every motor output is traceable to code I wrote and copper I laid out.

A key consequence: the discrete **BMI323 IMU**, **MMC5983MA magnetometer** and **BMP390 barometer** are not redundant here. A commercial FC would contain all three; a bare Nucleo contains none, so they become part of the carrier module.

---

## Architecture

[![MiBeesFly high-level design](%281%29HIghLevelDesign%26%28NOTES%29/MiBeesFly_HighLevelDesign_v0.1_web.png)](%281%29HIghLevelDesign%26%28NOTES%29/MiBeesFly_HighLevelDesign_v0.1.png)

*High-level design v0.1 — click for full resolution. A Mermaid version of the same diagram, plus the full write-up, is in [Architecture.md](%281%29HIghLevelDesign%26%28NOTES%29/Architecture.md).*

The design splits into three layers:

| Layer | What it is | Status |
|---|---|---|
| **Compute** | **NUCLEO-F767ZI** — STM32F767ZIT6 Cortex-M7 with a double-precision FPU, on-board ST-LINK/V2-1. Used as a module rather than designing an MCU board at V0.1 | ✅ Selected |
| **Carrier module** | Custom PCB in **KiCad**. Carries the sensors, power regulation, ESC signal headers and the connectors for GPS, radio, CAN and logging. The Nucleo plugs into it | 🚧 Schematic not started |
| **Firmware** | **C** in **STM32CubeIDE**. Sensor drivers, fusion filter, attitude and position control loops, motor mixing, receiver input, failsafe | 🚧 Not started |

Designing a carrier around a Nucleo rather than a standalone FC board is deliberate for V0.1 — it removes the MCU supply, crystal, boot and USB/debug circuitry from the first board spin, so the first PCB only has to get sensors, power and connectors right. A fully integrated board is a later revision.

The Cortex-M7's double-precision FPU is the reason for this MCU: attitude estimation and the fusion filter are floating-point heavy, and hardware double precision removes a class of numerical problems that would need careful handling on an M4.

---

## Interface allocation

| Interface | Peripheral | Protocol |
|---|---|---|
| **SPI 1** | BMI323 IMU (Shuttle Board 3.0) | Highest-rate sensor — SPI over I²C for sample rate |
| **SPI 2** | SD card module | Blackbox logging |
| **I²C** | MMC5983MA magnetometer, BMP390 barometer | Shared bus |
| **USART 1** | RP1 V2 ExpressLRS 2.4 GHz receiver | CRSF |
| **USART 2** | Beitian BE-882i GNSS | UBX / NMEA |
| **USART** | Matek 3901-L0X flow + lidar | MSP |
| **CAN FD** | TCAN3403DRQ1 transceiver | Expansion bus |
| **PWM ×4** | ESC signal outputs | Timer outputs |

---

## Power architecture

Both logic rails come from **dedicated DC-DC converters**, not from the ESC BECs.

| Rail | Source | Feeds |
|---|---|---|
| 14.8 V nominal | 4S LiPo, XT60 | 4× ESC, both buck converters |
| 5 V | DFR1015 buck #1 | All 5 V peripheral modules |
| 3.3 V | DFR1015 buck #2 | NUCLEO-F767ZI |

This avoids paralleling the four ESC UBECs, which would set up circulating currents between regulators.

> **Build rule:** because 5 V comes from the buck converter, the **red wire on all four ESC servo leads must be left disconnected**. Only signal and ground go to the carrier.

---

## Sensor suite

The point of the sensor selection is **drift-free position hold in both environments**: GNSS outdoors, optical flow + lidar indoors, with the IMU and barometer tying the two together.

| Sensor | Part | Bus | Role |
|---|---|---|---|
| IMU | **Bosch BMI323** on Shuttle Board 3.0 | SPI 1 | Gyro + accelerometer — the core of attitude estimation |
| GNSS + compass | Beitian BE-882i (M10, IST8310) | USART 2 | Absolute position and heading outdoors |
| Optical flow + lidar | Matek 3901-L0X | USART | Ground-relative velocity and low-altitude range — indoor position hold |
| Magnetometer | MMC5983MA | I²C | On-board heading reference |
| Barometer | BMP390 | I²C | Altitude hold and vertical velocity |
| Logging | SD card module | SPI 2 | Blackbox data for tuning |

---

## Airframe and propulsion

| | |
|---|---|
| Frame | F450-class carbon fibre, 4-axle |
| Motors | SunnySky X2216 **880 KV** ×4 — flat shaft, needs prop adapters |
| ESCs | 40 A, 2–4S, with 5 V 3 A UBEC ×4 (+1 spare) — **UBEC unused** |
| Battery | 4S 5200 mAh 120C LiPo, XT60 (~376 g) |
| Radio | RadioMaster Pocket (ELRS) + RP1 V2 nano receiver |
| Props | APC **1147** carbon fibre |

**On prop sizing:** 1147 was chosen deliberately over 1045/1047 and paired with the low-Kv 880 KV motor — a larger, slower-turning prop is more efficient in hover. Sustained current per motor still needs confirming against SunnySky's thrust table and measuring on the bench.

---

## Repository structure

```
(1)HIghLevelDesign&(NOTES)/
    Architecture.md              ← full system design
    MiBeesFly_HighLevelDesign_v0.1.png       Original drawing, full resolution
    MiBeesFly_HighLevelDesign_v0.1_web.png   Downscaled copy used in pages
    MiBeesFly_NOTES.docx         Working notes
    STMN32UCLEOBOARD/            ST reference docs: UM1974, DB3171, MB1137 source
(2)Hardware/
    MiBeesFly_FC_V0.1(PCB)/      KiCad project — carrier schematic + layout
    SchematicPDF/                Exported schematics for review
(3)Firmware(Software)/           STM32CubeIDE project — C firmware
(4)Drone_Build_ComponentsList.xlsx   Parts ordered, costs, parts still needed
(5)FullyAssemble/                Assembly notes, wiring, photos, test logs
```

---

## Roadmap

- [x] Select the compute platform — NUCLEO-F767ZI
- [x] Select the IMU — BMI323
- [x] Define the power tree and interface allocation
- [ ] Carrier module schematic in KiCad
- [ ] PCB layout and V0.1 fabrication
- [ ] Assembly and soldering
- [ ] Bring-up: power rails metered, then one sensor driver at a time
- [ ] Attitude estimation and stabilised (rate/angle) flight
- [ ] Sensor fusion — GNSS + baro outdoors, flow + lidar indoors, with clean handover
- [ ] Position hold and altitude hold
- [ ] Failsafe, arming logic and telemetry
- [ ] Full assembly and first flight

Tracked in detail under [Milestones](https://github.com/mohanan02/MiBeesFly/milestones).

---

## Build cost so far

Roughly **NZ$960** across 13 orders, parts and shipping included. FPV gear, a LiPo balance charger and assembly accessories are still to come — see the **Still Needed** sheet in `(4)Drone_Build_ComponentsList.xlsx`.

---

## Safety

A 450 quad with 11 in props on 4S is not a toy. Props off the motors for every bench test, LiPos charged and stored in a fire-safe bag, and no armed testing without a reliable failsafe path. Fly within your local regulations — in New Zealand that means CAA Part 101.

---

## License

Apache License 2.0 — see [LICENSE](LICENSE).

ST reference material under `(1)HIghLevelDesign&(NOTES)/STMN32UCLEOBOARD/` is STMicroelectronics' work, distributed under ST's own terms, not Apache 2.0.
