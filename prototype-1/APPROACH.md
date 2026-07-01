# Prototype-1 Approach

Prototype-1 replaces all custom PCB fabrication with perfboard and pre-made breakout modules. Zero KiCad required. Zero JLCPCB lead time. Same electrical performance, same firmware, same 3D printed shell.

This is the right first step. Validate everything on perfboard before committing to PCB layout — discover problems cheaply, iterate in minutes not weeks.

## What's the same as the final spec

- Seeed XIAO nRF52840 (same chip, same ZMK firmware)
- All sensors (TMR modules, AH49E Hall triggers, BMI270)
- 3D printed shell (PETG + TPU)
- All firmware features (bonding slots, calibration, sleep, wired HID)
- JST-SH 1.0mm harnesses for modularity
- TP4056 + single USB-C port

## What's different

| Final Spec | Prototype-1 Shortcut | Tradeoff |
|---|---|---|
| Custom mainboard PCB (KiCad) | FR4 perfboard, Dremel-cut to shape | No ground plane — may need software ADC smoothing on TMR reads |
| Custom TMR daughterboards | AKNES AK202 TMR modules (owned) — Xbox Series footprint, through-hole pins, mount directly to perfboard | 14 solder joints per module: 3×X axis, 3×Y axis, 4×button, 4×frame anchors. Analog X/Y output wires straight to XIAO ADC pins. |
| Custom trigger sensor boards | AH49E wired direct, legs heat-shrunk, sensor glued into shell slot | No PCB. Advantage: physically shim sensor depth in the slot by reprinting at adjusted depth until voltage curve across full trigger travel is linear. Do this before finalising shell. |
| Custom button boards | Small perfboard squares, switches soldered into grid | Slightly less mechanically rigid but fully functional |

## Noise caveat

The TMR sensors are sensitive. On a final PCB, short controlled traces and a ground plane suppress noise. On perfboard, the nRF52840's BLE antenna will couple RF noise into the unshielded wire runs from the TMR sticks — expect some ADC jitter at rest. Do not try to fix this in hardware on prototype-1.

**Fix in firmware:** Implement a moving-average filter and a small deadzone in ZMK/Zephyr. Zephyr has built-in support for both. The result is invisible to the player — resting jitter is smoothed before it reaches the HID report.

## Critical wiring rules before you solder

These are easy mistakes to make and hard to debug after the fact.

**1. DRV8833 VM pin → TP4056 OUT+ (battery rail), NOT XIAO 3.3V**
The ERM motors draw heavy transient current. The VM pin needs raw battery voltage (3.7–4.2V). If you wire it to the XIAO's 3.3V rail instead, motor current spikes will brown out the MCU mid-rumble. Wire VM directly to the same rail that powers the XIAO — the TP4056 output.

**2. BMI270 I2C pull-ups — verify before soldering**
I2C requires pull-up resistors on SDA and SCL to function at all. Most BMI270 breakout boards include 4.7kΩ pull-ups onboard. Check your specific breakout before wiring it up. If it doesn't have them, solder two 4.7kΩ resistors between SDA→3.3V and SCL→3.3V on the perfboard. Without them the gyro will appear completely unresponsive.

**3. AK202 pin identification before soldering**
The AK202 has 14 solder joints: 4 frame anchors (mechanical, solder for rigidity), 6 potentiometer-equivalent pins (3 per axis: VCC · signal · GND), and 4 button pins. Before committing wires, use a multimeter to confirm which pin is X signal and which is Y signal — power the module from 3.3V, probe each pin to GND with stick centered (~1.65V = signal pin), then deflect the stick to confirm it moves. Takes 5 minutes and prevents a rewire.

## What prototype-1 proves

- Every sensor works and reads correctly
- ZMK firmware compiles and runs — all 4 bonding slots, all layer logic, calibration routine
- Shell ergonomics feel right in hand
- Trigger mechanism returns cleanly, Hall reads are linear
- TMR bench test winner confirmed
- Wired HID passthrough works

## What moves to prototype-2 / final

- Custom PCBs in KiCad (ground planes, short traces, clean TMR routing)
- Possibly JST-SH connectors replaced with designed footprints
- Any shell revisions based on prototype-1 feel
