# MiBeesFly — working notes for AI assistants

## Conventions

**Commit messages must not contain AI attribution.** No `Co-Authored-By` lines, no session links,
no "generated with" footers. Commits are authored by the repository owner.

**Do not commit or push unless explicitly asked.** Make the change, report it, and wait. "Edit this
file" is not permission to commit it.

**Paths in this repo contain `(`, `)` and `&`.** Quote them in shell commands, and percent-encode
them in Markdown links (`(1)HIghLevelDesign&(NOTES)` → `%281%29HIghLevelDesign%26%28NOTES%29`).
An unencoded `)` silently truncates a Markdown link.

## What this project is

A flight controller built from scratch: a **KiCad-designed carrier module** around an **STM32
NUCLEO-F767ZI**, running **C firmware written in STM32CubeIDE**, flying a 450-class quadcopter.
GPS, compass, optical flow, lidar and barometer are fused for drift-free position hold indoors
and out.

Active, early-stage personal project. Nothing is flight-proven. The architecture is defined; the
schematic, PCB and firmware are all still ahead.

## Settled decisions — do not re-litigate

| Decision | Choice | Why |
|---|---|---|
| MCU | STM32F767ZI on NUCLEO-F767ZI | Cortex-M7 double-precision FPU for the fusion filter |
| Form factor | Carrier module, not a standalone FC board | Keeps MCU supply, clocking and debug circuitry off the V0.1 PCB |
| IMU | Bosch BMI323 on Shuttle Board 3.0 | Evaluate before committing a footprint |
| IMU bus | SPI, not I²C | I²C can't sustain the rate loop's sample rate |
| 5 V and 3V3 | Dedicated DFR1015 buck converters | Avoids paralleling four ESC UBECs |
| Props | APC 1147 | Deliberately paired with the low-Kv 880 KV motor — larger, slower prop is more efficient in hover |
| Logging | SD card on SPI 2 | PID tuning without logs is guesswork |

The 1147 propeller choice in particular has already been questioned and settled. It is a
considered decision, not an oversight.

## Where things live

- **`(1)HIghLevelDesign&(NOTES)/Architecture.md`** — the full system design. Read this first.
  Contains the original drawing and a Mermaid version of the same diagram.
- `(1)HIghLevelDesign&(NOTES)/STMN32UCLEOBOARD/` — ST reference docs. **UM1974** is the Nucleo-144
  user manual and the authority on pinouts, solder bridges and power-supply modes.
- `(2)Hardware/MiBeesFly_FC_V0.1(PCB)/` — KiCad carrier project (not started)
- `(3)Firmware(Software)/` — STM32CubeIDE project (not started)
- `(4)Drone_Build_ComponentsList.xlsx` — parts, costs, what's still needed
- Work is tracked in GitHub **Issues** (33) and **Milestones** (M1–M9)

The ST material under `STMN32UCLEOBOARD/` is STMicroelectronics' work under ST's own terms, not
Apache 2.0. Don't relicense it.

## Build rules that must survive into hardware

- **The red wire on all four ESC servo leads stays disconnected.** 5 V comes from the buck
  converter, so connecting the ESC UBECs would parallel four regulators.
- **Props off the motors for every bench test.** No exceptions.
- Power rails metered on a current-limited supply before the Nucleo is ever plugged into the
  carrier.

## Open questions

Don't assume these are resolved — check the issue tracker:

- Nothing is specified on the CAN bus yet (transceiver is a TCAN3403DRQ1)
- ST Morpho vs Zio mating not decided
- Sustained motor current on 1147 / 4S not yet measured
- IST8310 (on the GNSS module) vs MMC5983MA — which compass, or both
- PWM vs DShot for the ESCs
