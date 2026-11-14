# MiBeesFly 🐝 — Project Wiki

![Status](https://img.shields.io/badge/STATUS-WORK%20IN%20PROGRESS-E60026?style=plastic&labelColor=0D0D0D&logo=github&logoColor=white)
![Phase](https://img.shields.io/badge/PHASE-HARDWARE%20DESIGN-1E90FF?style=plastic&labelColor=0D0D0D)
![MCU](https://img.shields.io/badge/MCU-STM32F767ZI-03234B?style=plastic&labelColor=0D0D0D&logo=stmicroelectronics&logoColor=white)
![EDA](https://img.shields.io/badge/EDA-KiCad-314CB0?style=plastic&labelColor=0D0D0D&logo=kicad&logoColor=white)
![Firmware](https://img.shields.io/badge/Firmware-C%20%7C%20STM32CubeIDE-00599C?style=plastic&labelColor=0D0D0D&logo=c&logoColor=white)

**A flight controller built from scratch** — a KiCad-designed carrier module around an STM32 Nucleo-F767ZI, running C firmware written in STM32CubeIDE, flying a 450-class quadcopter. GPS, compass, optical flow, lidar and barometer are fused so the aircraft holds position without drift, indoors and out.

*Last updated: 14 November 2026*

---

## 📊 Where the project is right now

**Short version: parts are bought, the compute platform is chosen, and the custom hardware design is about to begin.** No schematic has been drawn and no firmware has been written yet. The project is roughly two weeks old.

```
Overall progress   ███░░░░░░░░░░░░░░░░░   ~15%
```

| # | Phase | Progress | State |
|---|---|---|---|
| 0 | Research & parts procurement | `█████████░` 90% | 10 orders placed, NZ$838 committed |
| 1 | Platform & architecture definition | `████░░░░░░` 40% | Nucleo-F767ZI chosen; IMU still open |
| 2 | Carrier schematic (KiCad) | `░░░░░░░░░░` 0% | **Next up** |
| 3 | PCB layout & fabrication | `░░░░░░░░░░` 0% | Not started |
| 4 | Board bring-up | `░░░░░░░░░░` 0% | Not started |
| 5 | Sensor drivers (C / CubeIDE) | `░░░░░░░░░░` 0% | Not started |
| 6 | Attitude estimation & stabilised flight | `░░░░░░░░░░` 0% | Not started |
| 7 | Sensor fusion & position hold | `░░░░░░░░░░` 0% | Not started |
| 8 | Failsafe, assembly & first flight | `░░░░░░░░░░` 0% | Not started |

---

## ✅ What has actually been done

**Procurement — essentially complete for the airframe.**
Ten orders placed across 8–9 November, NZ$838.46 paid including shipping. Frame, motors, ESCs, battery, radio, GNSS, optical flow and the discrete sensors are all bought and in transit or awaiting delivery.

**Compute platform selected — NUCLEO-F767ZI.**
This is the single biggest decision made so far, and it's a sound one. The STM32F767ZIT6 is a Cortex-M7 part with a double-precision FPU — fusion filters and control loops are floating-point heavy, and having hardware double precision removes a whole class of numerical headaches that would bite on an M4. The Nucleo-144 form factor also exposes nearly every pin on the ST Morpho headers, which is what makes a carrier-board approach viable at all.

**Reference documentation gathered.**
The ST reference material for the board is now committed under `(1)HIghLevelDesign&(NOTES)/STMN32UCLEOBOARD/`:

| File | What it's for |
|---|---|
| `DB_nucleo-f767zi.pdf` | Nucleo-144 family data brief (DB3171) — confirms NUCLEO-F767ZI → STM32F767ZIT6, board ID MB1137, on-board ST-LINK/V2-1 |
| `um1974-stm32-nucleo144-boards-mb1137-*.pdf` | **The important one.** Full user manual: connector pinouts, solder-bridge configuration, power-supply modes |
| `NUCLEO-144pins_Altium_Schematics-Layout/` | ST's official MB1137 source schematics and PCB (Altium) |
| `tn1235-overview-of-stlink-derivatives-*.pdf` | ST-LINK variant reference |
| `stm32_eval-tools_portfolio.pdf` | ST evaluation tooling overview |

**Repository restructured.**
`(3)Software` renamed to `(3)Firmware(Software)`, `.gitignore` added covering Office lock files, KiCad backups and CubeIDE build output, and `Architecture.md` created as a placeholder.

---

## 🚧 What is in progress

| Item | Status |
|---|---|
| `Architecture.md` | Created but **empty** — the design itself hasn't been written down yet |
| Flight controller | Marked *"On the process of designing"* in the parts list |
| Power distribution board | Marked *"On the process of designing"* — may be folded into the carrier |
| Carrier schematic | Reference material gathered, nothing drawn |

---

## ⚠️ Open decisions — these are blocking

These need resolving before the schematic can be finished. They're listed roughly in order of how much they block.

### 1. IMU — unselected 🔴 **critical**
There is **no gyroscope or accelerometer in the parts list**, and the Nucleo board doesn't have one. Nothing flies without this. It also drives the schematic directly: the bus choice (SPI strongly preferred over I²C for IMU sample rates), the interrupt pin routing, and the mechanical isolation on the PCB all depend on which part is chosen. Common choices in this class are the ICM-42688-P, BMI270 or MPU-6000-family parts.

### 2. Carrier mating strategy 🟠
Does V0.1 mate to the **ST Morpho** headers (full pin access, physically bulkier) or the **Zio/Arduino** headers (fewer pins, more standard)? UM1974 has the pinout tables needed to decide. This determines the board outline and the entire connector layout.

### 3. Propellers 🟠
The 1147 (11-inch) carbon props on order are too large for a 450 frame and risk pushing the SunnySky X2216 past its ~20 A limit on 4S. **1047 (10×4.7) is the correct size.** Prop adapters are also needed — the X2216 has a flat shaft. This is a known issue carried over from the parts list and is still unresolved.

### 4. Power architecture 🟡
How the Nucleo gets powered in flight (ESC UBEC → E5V/VIN), what rails the carrier generates (3V3 for sensors, 5V for peripherals), and whether battery voltage/current sensing lives on the carrier or a separate PDB.

### 5. EDA format mismatch 🟡
ST's reference design is **Altium**; this project uses **KiCad**. Those files can't be opened directly in KiCad — treat them as a visual/PDF reference for pinouts and connector geometry, not as an importable starting point. The PDF exports (`MB1137.pdf`) are the practical way to use them.

---

## 🗺️ Roadmap

### Phase 1 — Architecture definition *(current)*
- [ ] Select the IMU and justify the choice
- [ ] Decide Morpho vs Zio mating
- [ ] Draw the power tree (battery → ESC UBEC → Nucleo → carrier rails)
- [ ] Allocate buses and pins: IMU on SPI, baro/mag on I²C, GPS on UART, receiver on UART (ELRS/CRSF), 4× motor PWM/DShot on timer outputs
- [ ] Write all of the above into `Architecture.md`

### Phase 2 — Carrier schematic (KiCad)
- [ ] Set up the KiCad project under `(2)Hardware/MiBeesFly_FC_V0.1(PCB)/`
- [ ] Nucleo header footprints and net assignment
- [ ] Power regulation and protection
- [ ] Sensor sub-circuits — IMU, BMP390, MMC5983MA
- [ ] Connectors: GPS, flow/lidar, ESC signal, receiver, telemetry
- [ ] Export schematic PDF to `(2)Hardware/SchematicPDF/`

### Phase 3 — PCB layout & fabrication
- [ ] Board outline and mechanical fit to the 450 frame
- [ ] Placement — IMU near the centre of mass, magnetometer away from current paths
- [ ] Routing, ground plane strategy, power pour
- [ ] DRC, fab output, order V0.1

### Phase 4 — Bring-up
- [ ] Power rails verified before the Nucleo is ever plugged in
- [ ] ST-LINK connection and blinky
- [ ] One sensor driver at a time, each verified against known-good readings

### Phase 5 — Flight software
- [ ] Sensor drivers: IMU, baro, mag, GNSS, optical flow + lidar
- [ ] Attitude estimation (complementary filter first, then EKF)
- [ ] Rate loop → angle loop → stabilised flight
- [ ] Motor mixing and ESC protocol
- [ ] Receiver input and arming logic

### Phase 6 — Fusion & autonomy
- [ ] Altitude hold (baro + lidar + accel)
- [ ] Outdoor position hold (GNSS)
- [ ] Indoor position hold (optical flow)
- [ ] **Clean handover between GNSS and flow** — the hardest and most interesting part of the project
- [ ] Failsafe and telemetry

### Phase 7 — Integration
- [ ] Full assembly, vibration isolation, wiring
- [ ] Bench tests with props off
- [ ] Tethered hover
- [ ] First free flight ✈️

---

## 🔧 Hardware summary

### Compute
| | |
|---|---|
| Board | NUCLEO-F767ZI (Nucleo-144, ST board ID MB1137) |
| MCU | STM32F767ZIT6, Arm Cortex-M7 with double-precision FPU |
| Debugger | On-board ST-LINK/V2-1 — no external probe needed |
| User manual | UM1974 |

### Sensors
| Sensor | Part | Role | Status |
|---|---|---|---|
| IMU | **TBD** | Gyro + accel — the core of attitude estimation | 🔴 **not selected** |
| GNSS + compass | Beitian BE-882i (M10 + IST8310) | Outdoor position and heading | Ordered |
| Optical flow + lidar | Matek 3901-L0X | Indoor velocity and low-altitude range | Ordered |
| Magnetometer | MMC5983MA ×2 | On-board heading reference | Awaiting delivery |
| Barometer | BMP390 ×2 | Altitude hold | Awaiting delivery |

> **Note:** the MMC5983MA and BMP390 were originally flagged as *"possibly redundant"* in the parts list — true for a commercial flight controller, which has them built in. Building a custom carrier makes them **required**.

### Airframe & power
| | |
|---|---|
| Frame | Happy 450 (4-axle), ~10 in max prop |
| Motors | SunnySky X2216 880 KV ×4 |
| ESCs | 40 A, 2–4S, 5 V 3 A UBEC ×4 (+1 spare) |
| Battery | 4S 5200 mAh 120C LiPo, XT60 (~376 g) |
| Radio | RadioMaster Pocket (ELRS) + XR1 receiver |
| Props | ⚠️ 1147 ordered — should be 1047 |

---

## 💰 Build cost

| | NZ$ |
|---|---|
| Parts ordered (10 orders, incl. shipping & fees) | **838.46** |
| Items subtotal | 644.46 |
| Shipping and fees | 194.00 |
| Still to buy | TBD |

**Already owned:** 18650 cells for the transmitter.

**Still needed:** correctly-sized 1047 props and adapters, FPV camera, VTX and antenna, goggles or phone receiver, LiPo balance charger, and assembly accessories (battery strap, XT60 connectors, GPS mast, vibration dampers, LiPo bag, wire, heat shrink).

Full detail lives in `(4)Drone_Build_ComponentsList.xlsx`.

---

## 📁 Repository layout

```
(1)HIghLevelDesign&(NOTES)/
    Architecture.md              High-level design  [empty — to write]
    MiBeesFly_NOTES.docx         Working notes
    STMN32UCLEOBOARD/            ST reference docs + MB1137 Altium source
(2)Hardware/
    MiBeesFly_FC_V0.1(PCB)/      KiCad carrier project  [not started]
    SchematicPDF/                Exported schematics for review
(3)Firmware(Software)/           STM32CubeIDE C project  [not started]
(4)Drone_Build_ComponentsList.xlsx
(5)FullyAssemble/                Assembly notes, wiring, test logs
```

---

## 🎯 Why build the controller instead of buying one

Off-the-shelf controllers (Matek, Pixhawk, SpeedyBee) are cheap and work well, so this isn't about saving money — a commercial FC would be faster and cheaper. The point is owning the whole stack end to end: schematic capture, PCB layout, sensor drivers, the fusion filter and the control loops. The goal is a controller where every sensor reading and every motor output is traceable to code and copper the author designed.

---

## ⚡ Safety

A 450 quad with 10-inch props on 4S is not a toy.

- **Props off the motors for every bench test.** No exceptions.
- LiPos charged and stored in a fire-safe bag.
- No armed testing without a verified failsafe path.
- First flights tethered or in open space, well clear of people.
- Fly within local regulations — in New Zealand that means **CAA Part 101**.

---

## 📜 Changelog

| Date | Milestone |
|---|---|
| 2026-11-14 | ST reference documentation and MB1137 Altium source added; `(3)Software` → `(3)Firmware(Software)` |
| 2026-11-12 | `Architecture.md` created |
| 2026-11-11 | `.gitignore` extended |
| 2026-11-10 | README rewritten with full project description and status badges |
| 2026-11-08/09 | 10 parts orders placed — frame, motors, ESCs, battery, radio, GNSS, flow/lidar, sensors |

---

*Licensed under Apache 2.0. This is an active personal project — nothing here is flight-proven.*
