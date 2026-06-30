# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Repo Is

Project Nova is a hardware + firmware design project for a custom Bluetooth game controller. There is no compiled code or test suite yet — the current repo is entirely documentation and build planning. Prototype-1 (perfboard build) is the active phase.

---

## Repository Structure

| File / Dir | Purpose |
|---|---|
| `SPEC.md` | Master engineering specification — all locked hardware decisions, firmware logic, ZMK layer architecture, LED sequences, PCB topology, physical constraints |
| `OVERVIEW.md` | Plain-language user-facing reference — all inputs, combos, modes, LED states |
| `BOM.md` | Final build BOM (custom JLCPCB PCBs) — tabular with prices and direct purchase links |
| `prototype-1/APPROACH.md` | What's different for the perfboard prototype and why |
| `prototype-1/BOM.md` | Prototype-1 BOM — organized by order (Amazon / Adafruit / Mouser / McMaster), prices, links, purchased items struck through |
| `prototype-1/SOURCING.md` | California supplier guide with recommended order sequence |

---

## Locked Hardware Decisions

These are finalized — do not suggest alternatives unless the reason is a verified electrical incompatibility.

- **MCU:** Seeed XIAO nRF52840 — castellated pads, UF2 bootloader (no SWD programmer needed for normal flashing), native USB HID, integrated antenna
- **LDO:** XIAO's built-in XC6210 (~160–200mV dropout). AMS1117-3.3 was removed — it requires 4.5V minimum input and browns out on LiPo. Power path: TP4056 OUT+ → XIAO 5V pin → XIAO 3V3 → all sensors
- **Joysticks:** TMR modules (GuliKit, ALPS pinout) — drift-free, quantum tunneling, zero mechanical wear
- **IMU:** BMI270 via I2C — active in DS4 mode only
- **Triggers:** AH49E Hall effect sensors + neodymium magnets embedded in 3D-printed PETG trigger arms
- **Firmware:** ZMK (Zephyr-based). GP2040-CE is RP2040-only and cannot run on nRF52840
- **PCB topology:** Modular daughterboards connected via JST-SH 1.0mm harnesses — not one monolithic board
- **USB-C:** Single port, dual-function. VBUS+GND → TP4056 (charging); D+/D− → XIAO (wired HID). Adafruit #4090 breakout exposes all four lines

---

## Critical Wiring Rules

These are easy to get wrong and hard to debug after the fact:

1. **DRV8833 VM pin → TP4056 OUT+ (battery rail, 3.7–4.2V) — never XIAO 3.3V.** Motor current spikes will brown out the MCU under rumble load if wired to 3.3V.
2. **BMI270 I2C pull-ups:** SDA and SCL each need 4.7kΩ to 3.3V. Most breakout boards include these — verify before wiring. Without them the gyro appears completely unresponsive.
3. **Arduino joystick breakout trace check:** After desoldering the stock potentiometer, probe X and Y output pins to the ALPS footprint holes in continuity mode. Some boards have a pull-up resistor across a signal trace that must be removed.
4. **Wired HID trigger:** VBUS high alone does not trigger wired mode — a wall charger also raises VBUS. Firmware must wait for USB enumeration (D+/D− handshake). Wall charger = charge only, computer = wired HID + charge.
5. **Trigger magnet isolation:** Neodymium trigger magnets must be physically deep inside the trigger housing, separated from TMR joystick PCBs to avoid field interference.

---

## Firmware Architecture (ZMK Layers)

| Layer | Name | Activation |
|---|---|---|
| 0 | XInput Base | Default boot (SPDT switch High) — BMI270 suspended |
| 1 | DS4 Base | Boot with SPDT switch Low — BMI270 active |
| 2 | Bonding — Connect | Momentary: `&mo 2` via Sync button |
| 3 | Bonding — Clear & Re-pair | Momentary: `&mo 3` via Sync + Guide ZMK combo |
| 4 | Calibration Mode | Combo hold: L3 + R3 + Guide for 3 seconds |

**Guide button:** Pure `&kp SYSTEM_GUI` — no hold-tap, no timing threshold. Every press sends Home.

**Sync button:** Pure `&mo 2` — no HID keycode, invisible to all hosts. Slot combos (Sync + face button) fire `&bt BT_SEL` locally; the host receives nothing for either button.

**Calibration (Layer 4):** Suppresses all HID output, samples ADC resting values from both TMR axes and both Hall triggers, writes to nRF52840 flash as absolute zero and boundary deadzones. LED strobe confirms write.

**ADC filtering:** TMR sensors are sensitive. Perfboard has no ground plane, so BLE antenna RF couples into wire runs. Fix in firmware only — moving-average filter + deadzone in Zephyr. Do not attempt a hardware fix on prototype-1.

---

## BOM Conventions

- All BOM tables include unit price, total, and a direct purchase link (Amazon search URL or specific product page)
- AliExpress is excluded — all sourcing is US suppliers
- Purchased/owned items are struck through with `~~text~~` in the table rows
- Prototype-1 BOM is organized by order (Amazon / Adafruit / Mouser / McMaster) to allow placing all orders in one session
- Main BOM.md is for the final JLCPCB build; prototype-1/BOM.md is the active build

---

## Builder Context

- Has: soldering station, 3D printer (PETG, PLA, TPU), common consumables
- Does not have: Dremel (cut perfboard by score-and-snap with a utility knife, or design shell to accept standard perfboard sizes)
- Shell is 3D printed PETG — Xbox Series X/S external dimensions used as ergonomic reference skeleton
- No KiCad or PCB fabrication in prototype-1 — everything on FR4 perfboard and breakout modules

---

## GitHub

Remote: `https://github.com/sidhucode/project-Nova.git` (branch: `main`)

Commit style: clean messages only, no AI co-author attribution.
