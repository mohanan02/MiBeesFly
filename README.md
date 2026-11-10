# MiBeesFly 🐝

![MCU](https://img.shields.io/badge/MCU-STM32%20Nucleo-03234B?style=flat&logo=stmicroelectronics&logoColor=white)
![EDA](https://img.shields.io/badge/EDA-KiCad-314CB0?style=flat&logo=kicad&logoColor=white)
![Firmware](https://img.shields.io/badge/Firmware-C%20%7C%20STM32CubeIDE-00599C?style=flat&logo=c&logoColor=white)
![Airframe](https://img.shields.io/badge/Airframe-450%20class%20quad-444444?style=flat)
![Power](https://img.shields.io/badge/Power-4S%20LiPo%20%7C%205200%20mAh-E8730C?style=flat)
![Status](https://img.shields.io/badge/Status-work%20in%20progress-1E90FF?style=flat)

A custom flight controller built from scratch — a **KiCad-designed carrier module** around an **STM32 Nucleo** board, plus **C firmware written in STM32CubeIDE** — flying a 450-class quadcopter. GPS, compass, optical flow, lidar and barometer are fused so the aircraft holds position without drift, indoors and out.

> **⚠️ Work in progress.** This is an active, early-stage personal project. Nothing here is flight-proven yet. The schematic, the carrier PCB and the firmware are all still being designed and iterated on, and the repository will change substantially as the build progresses. Treat everything below as the current intent, not a finished product.

---

## Why build the controller instead of buying one

Off-the-shelf flight controllers (Matek, Pixhawk, Speedybee) are cheap and work well, so this project is not about saving money. It's about owning the whole stack end to end: schematic capture, PCB layout, sensor driver code, the fusion filter and the control loops. The goal is a controller I fully understand, where every sensor reading and every motor output is traceable to code I wrote.

A key consequence of that choice: the loose **MMC5983MA magnetometer** and **BMP390 barometer** modules in the parts list are *not* redundant here. A commercial FC would already contain them; a bare Nucleo does not, so they become part of the carrier module.

---

## Architecture

The design splits cleanly into three pieces:

| Layer | What it is | Status |
|---|---|---|
| **Compute** | STM32 Nucleo development board — the MCU, debugger and core clocking, used as a module rather than designing an MCU board from scratch at V0.1 | Selecting the exact Nucleo variant |
| **Carrier module** | Custom PCB designed in **KiCad**. Carries the sensors, power regulation, ESC signal headers, and the connectors for GPS, radio and telemetry. The Nucleo plugs into it. | Schematic in progress |
| **Firmware** | **C**, written in **STM32CubeIDE** using HAL/LL drivers. Sensor drivers, fusion filter, attitude and position control loops, motor mixing, receiver input, failsafe. | Not started / early |

Designing a carrier around a Nucleo rather than a full standalone FC board is deliberate for V0.1 — it removes the MCU power supply, crystal, boot and USB/debug circuitry from the first board spin, so the first PCB only has to get the sensors, power and connectors right. A fully integrated board is a later revision.

---

## Sensor suite

The whole point of the sensor selection is **drift-free position hold in both environments**: GPS outdoors, optical flow + lidar indoors, with the barometer and IMU tying the two together.

| Sensor | Part | Role |
|---|---|---|
| GNSS + compass | Beitian BE-882i (M10 GNSS, IST8310 compass) | Absolute position and heading outdoors |
| Optical flow + lidar | Matek 3901-L0X | Ground-relative velocity and low-altitude range — indoor position hold where GPS is useless |
| Magnetometer | MMC5983MA | Heading for the custom board (the IST8310 is on the GPS mast; this is the on-board reference) |
| Barometer | BMP390 | Altitude hold and vertical velocity |
| IMU | TBD | Gyro + accelerometer — **still to be selected**; the single most important open hardware decision |

---

## Airframe and power

| | |
|---|---|
| Frame | Happy 450 (4-axle), ~10 in max prop |
| Motors | SunnySky X2216 880 KV ×4 |
| ESCs | 40 A, 2–4S, 5 V 3 A UBEC ×4 (+1 spare) |
| Battery | 4S 5200 mAh 120C LiPo, XT60 |
| Radio | RadioMaster Pocket (ELRS) + XR1 receiver |
| Props | **Under review** — see note below |

> **Open issue — propellers.** The 1147 (11 in) carbon props currently on order are likely too large for a 450 frame and risk pushing the X2216 past its ~20 A limit on 4S. **1047 (10×4.7)** is the correct size and is what the build should move to. Prop adapters are also needed — the X2216 has a flat shaft.

---

## Repository structure

```
(1)HIghLevelDesign&(NOTES)/   High-level design, requirements, research notes
(2)Hardware/
    MiBeesFly_FC_V0.1(PCB)/   KiCad project — carrier module schematic + layout
    SchematicPDF/             Exported schematics for review and sharing
(3)Software/                  STM32CubeIDE project — C firmware
(4)Drone_Build_ComponentsList.xlsx   Parts ordered, costs, and parts still needed
(5)FullyAssemble/             Assembly notes, wiring, photos, test logs
```

---

## Roadmap

- [ ] Finalise Nucleo variant and IMU selection
- [ ] Carrier module schematic in KiCad — power tree, sensor buses (I²C/SPI), ESC outputs, receiver and GPS connectors
- [ ] PCB layout and V0.1 fabrication
- [ ] Bring-up: power rails, then one sensor driver at a time in STM32CubeIDE
- [ ] Attitude estimation and stabilised (rate/angle) flight
- [ ] Sensor fusion — GPS + baro outdoors, flow + lidar indoors, with clean handover between them
- [ ] Position hold and altitude hold
- [ ] Failsafe, arming logic and telemetry
- [ ] Full assembly and first flight

---

## Build cost so far

Roughly **NZ$838** paid across 10 AliExpress orders (parts, shipping and fees included). The flight controller stack, power distribution, FPV gear, charger and correctly-sized propellers are still to come — see the **Still Needed** sheet in `(4)Drone_Build_ComponentsList.xlsx`.

---

## Safety

A 450 quad with 10 in props on 4S is not a toy. Props off the motors for every bench test, LiPos charged and stored in a fire-safe bag, and no armed testing without a reliable failsafe path. Fly within your local regulations — in New Zealand that means CAA Part 101.

---

## License

Apache License 2.0 — see [LICENSE](LICENSE).
