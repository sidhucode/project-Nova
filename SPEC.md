# Project Nova — Master Controller Specification

## Overview

Project Nova is a custom Bluetooth game controller built around the Nordic nRF52840, drift-free TMR joysticks, precision gyro aiming, and multi-device virtual partitioning. Target form factor is an Xbox-style shell with a 15–20 hour battery life and a 1–2 month build timeline. Supports both wireless BLE and wired USB HID.

---

## Hardware Components

### Processing

| Component | Specification | Engineering Rationale |
|---|---|---|
| Microcontroller | **Seeed XIAO nRF52840** *(locked)* | Complete module with integrated antenna, USB-C, and UF2 USB bootloader — no SWD programmer needed for normal flashing, no RF antenna design required. Castellated pads mount directly to mainboard. Same unit used for breadboard dev phase and soldered into final build. Industry-standard Nordic BLE stack. Native USB HID. |

### Power

| Component | Specification | Engineering Rationale |
|---|---|---|
| Battery System | 1500mAh **protected** LiPo Flat Pack + TP4056 Charge Controller + USB-C Breakout | Must be a **protected** cell — includes internal DW01A overdischarge cutoff, no separate protection circuit needed. Order battery first, measure actual dimensions, design shell cavity around them. XIAO's built-in 50mA charger is too slow (30hr charge time) — power XIAO from TP4056 regulated output; do not connect battery to XIAO BAT pin. Wired HID only activates on USB enumeration, not VBUS alone. |
| Regulation & Safety | XIAO nRF52840 built-in XC6210 regulator (3.3V, 200mA) | External AMS1117-3.3 removed — it requires 4.5V minimum input and browns out on LiPo. XIAO's XC6210 has ~160–200mV dropout, stable across full LiPo range. Wire TP4056 OUT+ → XIAO 5V pin. All sensors from XIAO 3V3 pin. Verified current budget: TMR ×2 (~6mA) + Hall ×2 (~10mA) + BMI270 (<1mA) + LEDs ×4 (~20mA) = ~37mA total. 200mA limit gives 5× safety margin. |

### Motion

| Component | Specification | Engineering Rationale |
|---|---|---|
| Gyroscope / IMU | BMI270 **or** LSM6DSOX (via I2C) | Ultra-low-noise 6-axis IMU. Wired via SDA/SCL. Precision gyro aiming comparable to or exceeding the DualSense. Enabled only in DS4 mode. |

### Inputs

| Component | Specification | Engineering Rationale |
|---|---|---|
| Joysticks | TMR Modules — **Favor Union FJH10K-TMR** or **GuliKit TMR (720° Tension Adjustable)** | True TMR modules with standard ALPS pinouts; available as drop-in replacements on Amazon/AliExpress. Quantum magnetic tunneling resistance eliminates mechanical wear and drift. **Bench test required:** wire both to nRF52840 on breadboard, compare raw Zephyr ADC resting voltage stability without software smoothing — tightest baseline wins. |
| Triggers (L2/R2) | AH49E Hall Effect Sensors + 3mm×2mm N35–N52 Neodymium Disc Magnets + 3D-printed PETG trigger arms + 2mm steel pivot pins + coil return springs | AH49E reads the changing magnetic field as the trigger arm rotates. Magnet is embedded in the trigger arm body. Return force provided by a small coil spring (~4mm OD) seated on or beside the 2mm steel pivot axle. Trigger arms are 3D printed PETG. Trigger magnets must be physically isolated deep in the housing to shield TMR joysticks. |
| D-Pad | 4× Tactile Microswitches + 3D-printed central pivot post | Crisp, mechanical click feel. Pivot post prevents simultaneous opposite directions (e.g., Left + Right), mimicking official D-pad physics. |
| Bumpers (L1/R1) | Horizontal Mouse Microswitches (Kailh / Omron) | Sub-millimeter travel, instant tactility, high cycle life. |
| Face Buttons (A/B/X/Y) | Permanent Xbox layout (friction-fit / glued) | Fixed visual interface for engineering pragmatism and timeline. Nintendo prompt remapping handled via software translation layers or physical stickers. |
| Menu + View Buttons | 2× Tactile Microswitches — center cluster | Menu = Start/Options equivalent (pause, in-game menus). View = Select/Share equivalent (map, scoreboard). Required by virtually all games; placed in the standard center-cluster position between the face buttons and Guide button. In DS4 mode: Menu maps to Options, View maps to Create/Share. |
| Sync Button | 1× Tactile Microswitch — Share button position (below Guide) | Repurposes the Xbox Series Share/Capture button footprint. Dedicated BLE slot selector: Sync held + face button switches the active bonding slot. Removes all hold-tap complexity from the Guide button — Guide becomes a single-function tap button. |

### Haptics

| Component | Specification | Engineering Rationale |
|---|---|---|
| Rumble System | Dual DC ERM (Eccentric Rotating Mass) Motors + DRV8833 Motor Driver | Haptic motors generate heavy transient current spikes. DRV8833 H-bridge isolates MCU pins entirely. **Critical wiring: DRV8833 VM pin must connect directly to TP4056 OUT+ (raw 3.7–4.2V battery rail), NOT to XIAO 3.3V.** Motors need full battery voltage; routing through the 3.3V rail would brown out the MCU under load. |

### Telemetry

| Component | Specification | Engineering Rationale |
|---|---|---|
| Status Array | 4× 0603 SMD LEDs | Displays pairing animations and connection state. LEDs light sequentially to indicate active bonding slot (1–4). Light pipes removed — shell is 3D printed so LED positions are designed flush to shell surface. Clear PETG inserts can be printed if a standoff gap requires bridging. |

---

## Firmware & Logic

### 1. Dual-Boot Hardware Signal Toggle (XInput / DS4)

**Hardware:** Physical SPDT slide switch mounted through the rear shell, wired to a digital GPIO pin (pulled high or grounded).

**Logic:** At boot, the MCU reads the pin state:

- **High → Xbox Mode:** Boots the XInput BLE library. Presents as a native Xbox controller to Windows. No gyro stream. Native Game Pass / Epic Games compatibility.
- **Low → PlayStation Mode:** Boots the DS4 BLE library. Enables the BMI270 gyro data stream. Native support on macOS, Steam Deck, and Steam on Windows.

---

### 2. Virtual Hardware Partitioning (Multi-Device Bonding)

**Problem:** Standard Bluetooth bonding causes "tug-of-war" between multiple paired devices.

**Solution:** The firmware maintains four independent BLE profiles, each with a distinct MAC address. Each profile bonds to one target device and is completely invisible to the others.

There are three distinct Bluetooth operations, each with its own combo:

#### A. Reconnecting / Switching (daily use)

**Combo:** Hold **Sync**, press a face button.

ZMK behavior: `&bt BT_SEL n`

| Combo | MAC Suffix | Slot | Target Device |
|---|---|---|---|
| Sync + Y | `...01` | Slot 1 | Steam Deck |
| Sync + B | `...02` | Slot 2 | macOS |
| Sync + A | `...03` | Slot 3 | Windows PC |
| Sync + X | `...04` | Slot 4 | Spare / Mobile |

- **Slot is already paired:** Controller connects directly to the saved device. No advertising, no pairing dialog on the host.
- **Slot is empty (never used):** Controller immediately starts advertising. Host discovers and pairs it.

`BT_SEL` never clears bonding data. Switching back to a paired device can never accidentally trigger re-pairing.

#### B. First-Time Pairing (empty slot)

No special action required. Selecting an empty slot via Sync + face automatically starts advertising. First-time pairing and daily reconnecting use the same combo — the firmware handles the distinction internally based on whether bond data exists for that slot.

#### C. Re-Pairing (overwriting a slot with a new device)

**Combo:** Hold **Sync + Guide simultaneously**, then press a face button.

ZMK behavior: `&bt BT_CLR` on the selected slot → immediately begins advertising.

- Clears all bonding data for that slot on the controller
- Controller starts advertising for a fresh pair
- LED: Cylon scan (same as first-time pairing)

This combo requires two buttons held before any slot is chosen, making it nearly impossible to trigger mid-game or accidentally during a normal slot switch.

**BLE bond asymmetry — important:** BLE has no standardized mechanism for a peripheral to instruct a host to forget it. `BT_CLR` only clears the controller's side of the bond. The old host device (e.g., the macOS machine being replaced in a slot) retains its copy of the shared keys.

Consequences:
- The new device pairs successfully regardless — the controller advertises fresh and exchanges new keys with the new host
- If the old host is nearby and tries to auto-reconnect, it sends its old keys, the controller rejects them (key mismatch), and the old host shows a connection failure or "Not Connected" error
- The old host will not interfere with or block the new pairing
- To fully clean up, the old host should manually remove the controller from its Bluetooth settings — but this is not required for the new device to work

Re-pairing is a rare, deliberate action. The stale ghost entry on the old device is a minor one-time cleanup, not a recurring problem.

---

### 3. On-Board Flash Calibration (Zero-Point Calibration)

**Problem:** TMR joysticks are exceptionally sensitive. Minor structural tolerances or housing flex can offset the "absolute center" resting voltage.

**Solution:** A boot-time calibration routine samples and stores resting values directly in the MCU's non-volatile EEPROM/Flash. Eliminates reliance on external software calibration tools.

**Activation:** Hold **L3 + R3 + Guide** for 3 seconds to enter the calibration routine.

**What it captures:**
- Resting voltages of both TMR joysticks → stored as true center
- Resting voltages of both linear Hall triggers → stored as boundary deadzones

All values are written to flash and persist across power cycles.

---

### 4. Automated Deep-Sleep Architecture

**Problem:** Battery drain when the controller is left idle on a desk.

**Trigger:** No analog or digital state changes detected for 5 consecutive minutes.

**Sleep sequence:**
1. BLE radio power amplifier shut down
2. MCU enters low-frequency sleep cycle
3. BMI270 sensor powered down

**Wake:** Pressing the Guide button triggers a hardware interrupt pin, instantly resuming full operation.

---

### 5. Wired USB HID Passthrough

**Problem:** Wireless-only limits utility for competitive play (latency) and dead-battery scenarios.

**Hardware:** No extra silicon required. The nRF52840 has native USB support; the existing USB-C breakout (already present for charging) carries the data lines.

**Logic:** VBUS high alone is not sufficient to trigger wired HID mode — a wall charger or power bank also raises VBUS with no data host behind it. The firmware waits for successful USB enumeration (D+/D- handshake with a host) before switching modes.

| USB-C state | VBUS | Enumerated | Behavior |
|---|---|---|---|
| Unplugged | Low | No | Wireless BLE, battery powered |
| Wall charger / power bank | High | No | Charge only. BLE stays active if battery has charge. Controller stays off / charging if battery is dead. |
| Computer (with data) | High | Yes | Suspend BLE radio. Switch to wired USB HID. TP4056 charges battery in parallel. |
| Cable unplugged | Falls low | — | Resume BLE radio. Return to last active bonding slot. |

**Dead battery + wall outlet:** Controller charges until it accumulates enough voltage to boot, then powers on normally in BLE mode. It will not present as a USB HID device — there is no data host on the other end.

**Dead battery + computer:** Same charging phase first. Once the nRF52840 has enough voltage to boot, it enumerates as a USB HID device and becomes fully playable over the wire while charging.

The mode switch is interrupt-driven — no polling, no latency penalty.

---

## PCB Topology

**Decision: Modular Daughterboards** *(locked)*

A single monolithic PCB aligned to the shell requires sub-millimeter precision. Any 3D print shrinkage (~2% during cooling) or shell revision would require a full board redesign.

**Architecture:**

| Board | Contents | Notes |
|---|---|---|
| Central Mainboard | nRF52840, power management (TP4056, LDO), DRV8833, BMI270/LSM6DSOX IMU, LED array | IMU mounted here to isolate it from stick/trigger vibrations and prevent gyro micro-jitters |
| Left Stick Daughterboard | TMR module (left), L3 switch | Short flex cable or ribbon to mainboard |
| Right Stick Daughterboard | TMR module (right), R3 switch | Short flex cable or ribbon to mainboard |
| Trigger Boards (×2) | Hall effect sensor (AH49E), trigger magnet housing | Isolated deep in trigger housing; physically separated from stick boards |
| Button Board | Face buttons (A/B/X/Y), D-pad microswitches, bumpers | Can be one board or split left/right |

**Benefits:**
- Physical wiggle room between boards absorbs shell tolerance variance
- Failed or damaged input module can be replaced without reflowing the whole board
- Easier to iterate on individual input zones during prototyping

---

## Firmware Base

**Decision: BlueMicro → ZMK** *(GP2040-CE ruled out)*

GP2040-CE is hardcoded to the RP2040 architecture (Raspberry Pi Pico silicon). It cannot run on the nRF52840 and its wireless support via the Pico W (CYW43439) has historically poor BLE latency compared to Nordic chips.

| Option | Assessment |
|---|---|
| **ZMK** | Mature Zephyr-native framework with extensive nRF52840 support, HID profiles, macro layers, and an active community. Target for final firmware. |
| **BlueMicro** | Lighter nRF52 firmware, faster to get running. Use for early hardware validation (sensors, BLE bonding, calibration) before migrating to ZMK. |
| GP2040-CE | RP2040 only. Not viable. |

**Build path:** Start on BlueMicro to validate all hardware (TMR ADC reads, Hall trigger reads, BLE bonding, I2C IMU). Migrate to ZMK for final firmware with full layer structure, profile switching, and calibration macro.

---

## ZMK Layer Architecture

ZMK was designed for mechanical keyboards, so the controller is treated as a macro pad with complex modifier keys. The key principle: **highest active layer always wins**, so base modes sit at the bottom and transient command modes stack on top.

### Layer Map

| Layer | Name | Activation | Behavior |
|---|---|---|---|
| 0 | XInput Base | Default boot state (SPDT switch High) | Standard Xbox HID output. BMI270 I2C polling suspended — gyro data ignored entirely to save power. |
| 1 | DS4 Base | Toggled at boot (SPDT switch Low) | Broadcasts as DualShock 4 HID. Activates BMI270 I2C polling; raw gyro + accelerometer streamed to host. |
| 2 | Bonding — Connect | Momentary hold: **Sync** (`&mo 2`) | Face buttons fire `&bt BT_SEL 0/1/2/3`. Connects to saved device (or advertises if slot is empty). Never clears bonding data. |
| 3 | Bonding — Clear & Re-pair | Momentary hold: **Sync + Guide** simultaneously (`&mo 3` via ZMK combo) | Face buttons fire `&bt BT_CLR` on the selected slot, then immediately begin advertising. Use to overwrite a slot with a new device. |
| 4 | Calibration Mode | Combo hold: L3 + R3 + Guide (`&mo 4`) | Locks out all HID output to host. Triggers custom Zephyr macro: reads ADC resting values from both TMR sticks and Hall triggers, writes to flash as absolute zero and boundary deadzones. |

### The Guide Button

With the dedicated Sync button handling Layer 2, the Guide button is a simple single-function key:

```
&kp SYSTEM_GUI
```

Every press — tap or hold — sends the OS Home command (Steam overlay, Xbox menu, etc.). No hold-tap behavior, no timing threshold, no ambiguity. The `tapping-term-ms` setting is no longer relevant to this button.

### The Sync Button: Layer 2 Modifier

The Sync button (Share button footprint, below Guide) is the dedicated BLE slot selector. It activates Layer 2 as a standard momentary layer shift:

```
&mo 2
```

Hold Sync, then press a face button to switch slots. Release Sync to return to the base layer. No timing ambiguity — the face button press is always unambiguous because Sync has no tap behavior of its own.

### Sync Combo Conflict Analysis

The Sync button replaces Guide as the BLE slot selector, which resolves the platform conflict question more cleanly than hold-tap timing ever could.

**Why the combos are safe — the architectural reason:**

The Sync button has no HID keycode assigned to it at all. It is purely a ZMK layer modifier (`&mo 2`). When held, it activates Layer 2. When a face button is pressed in Layer 2, ZMK fires `&bt BT_SEL` — a local Bluetooth profile command that never generates a HID keycode. The host receives nothing for either button.

- **Sync alone (released without face button):** No HID event sent. The host sees nothing.
- **Sync held + face button:** ZMK fires local BT_SEL command. Host sees nothing.

This means OS-level and Steam-level chord intercepts are irrelevant — they can only intercept events that reach them, and in the hold path, nothing does.

**Per-platform audit:**

The Sync button has no identity on any platform — it sends no HID keycode and is invisible to the host in all cases. There are no OS-level or Steam-level shortcuts bound to an unidentified modifier + face button. No conflicts exist on any target platform.

For reference, the Guide/PS button platform behaviors (now relevant only to the simple Home tap):

| Platform | Guide/PS button behavior |
|---|---|
| Windows + Steam (XInput) | Intercepted by Windows (Game Bar) and Steam (overlay). Games cannot read it via public XInput API. |
| macOS + Steam (DS4) | No OS-level intercept. Steam intercepts for overlay. |
| Steam Deck external DS4 | Compositor maps PS button to STEAM button behavior. PS+A/B/X are reserved for system shortcuts; PS+Y is unbound. |

These are no longer relevant to slot selection since Guide is now a pure tap button.

---

### Bluetooth Slot Mapping (Layer 2)

| Button (held on Layer 2) | ZMK Behavior | BLE Profile | Target Device |
|---|---|---|---|
| Y | `&bt BT_SEL 0` | Profile 0 / MAC `...01` | Steam Deck |
| B | `&bt BT_SEL 1` | Profile 1 / MAC `...02` | macOS |
| A | `&bt BT_SEL 2` | Profile 2 / MAC `...03` | Windows PC |
| X | `&bt BT_SEL 3` | Profile 3 / MAC `...04` | Spare / Mobile |

### Calibration Combo (Layer 4)

Entering Layer 4 requires a deliberate three-button hold (L3 + R3 + Guide) — impossible to trigger accidentally mid-game. While Layer 4 is active:
- All HID reports to the host are suppressed
- A custom Zephyr macro samples current ADC values from both TMR joystick axes and both Hall effect triggers
- Sampled values written to nRF52840 flash as: true center (sticks) and min/max boundaries (triggers)
- LED array flashes a confirmation pattern on write completion before returning to base layer

---

## LED Feedback Sequences

Scripted via Zephyr's LED subsystem. All sequences are designed to be battery-conscious — LEDs shut off after conveying their state rather than staying lit during gameplay.

| Layer / Event | Sequence | Description |
|---|---|---|
| Layer 0 — XInput Boot | Quick double-blink, then off | Two fast pulses confirm XInput mode; LEDs go dark for battery conservation |
| Layer 1 — DS4 Boot | Quick triple-blink, then off | Three fast pulses confirm DS4 mode; LEDs go dark |
| Layer 2 — Bonding Mode (searching) | Cylon scan: LED 1 → 2 → 3 → 4 → 3 → 2 → 1 (loop) | Sweeping animation while actively pairing to the selected BLE slot |
| Layer 2 — Bonding Mode (connected) | Corresponding slot LED holds solid | LED 1 = Steam Deck, LED 2 = macOS, LED 3 = Windows, LED 4 = Spare/Mobile. Turns off after 3 seconds. |
| Layer 3 — Calibration complete | All four LEDs strobe rapidly for 3 seconds, then off | Rapid strobe confirms absolute zero values have been written to flash |

**Note:** No LEDs are lit during active gameplay in any mode. All feedback sequences are transient — they fire on state change, then extinguish.

---

## Physical Layout Constraints

### Magnetic Interference Protection for TMR Joysticks

TMR modules are highly sensitive to nearby magnetic fields. The following placement rules are non-negotiable:

| Source | Risk | Mitigation |
|---|---|---|
| LiPo Battery | Faint EM fields from high-current transients during heavy rumble | Center the battery pack in the rear shell, isolated from analog stick PCBs |
| ERM Rumble Motors | Permanent internal magnets | Mount motors deep inside the lower hand grips; physical distance through plastic keeps them outside the interference radius of both TMR sticks |
| Face Button Caps | Any magnetic keycaps/caps ruled out | No magnetic button caps anywhere on the shell |
| Trigger Magnets (Hall sensors) | Direct magnet near TMR modules | Isolate neodymium trigger magnets deep inside trigger housing, physically shielded from joystick PCBs |

---

## Decisions Log

| Decision | Outcome | Rationale |
|---|---|---|
| MCU | **nRF52840** | Superior power management, lowest BLE latency, native USB HID, industry-standard for wireless peripherals |
| USB-C scope | **Charging + wired HID passthrough** | Native to nRF52840; zero extra hardware; VBUS interrupt switches modes automatically |
| PCB topology | **Modular daughterboards** | Tolerates shell variance; easier iteration; isolates IMU from vibration |
| Firmware base | **BlueMicro → ZMK** | GP2040-CE is RP2040-only; ZMK is the mature nRF52840 target; BlueMicro for early hardware validation |
| TMR sourcing | **Bench-test both GuliKit and Favor Union/K-Silver on breadboard first** | Wire both to nRF52840, use Zephyr raw ADC output to compare resting voltage stability without software smoothing; tightest baseline wins |
| Shell | **3D printed — PETG, Xbox Series X/S ergonomic reference** | Printer available; shell designed around exact PCB positions rather than PCBs designed around shell. Use Xbox Series X/S dimensions as ergonomic skeleton (calipers or published dimensions) to get proven grip geometry without designing from scratch. TPU grip inlays printed directly into design. Donor shell no longer required. |
| D-pad pivot post | **PETG, 0.1mm layer height, 99–100% infill, ~0.1–0.15mm undersize** | PETG resists heat creep (PLA fails) and doesn't warp without an enclosure (ABS fails); full infill prevents delamination under impact |
| IMU | **BMI270** | Lower noise floor, industry workhorse for motion-controlled gaming; LSM6DSOX machine-learning core is unnecessary overhead |
| Daughterboard interconnects | **JST-SH 1.0mm wire harness (v1)** | Flex PCBs require minimum fab orders and 2-week lead times; JST-SH allows hot-swap and re-pinning in 60 seconds if a sensor board fails |
| ZMK layer structure | **4-layer hierarchy (XInput base / DS4 base / Bonding overlay / Calibration mode)** | See ZMK Layer Architecture section |
| Guide button behavior | **Pure tap — `&kp SYSTEM_GUI`** | Dedicated Sync button removes the need for hold-tap on Guide; Guide always sends Home command, no timing threshold |
| Sync button | **Dedicated BLE slot selector — `&mo 2`** | Repurposes Xbox Series Share/Capture button footprint; no HID keycode, invisible to host, zero platform conflicts |
| LED feedback sequences | **Per-layer sequences defined** | See LED Feedback section |

## Builder's Setup

**Available:**
- Soldering station
- 3D printer (PETG, PLA, TPU)

**3D printer applications for this build:**
- **Full shell** — PETG, designed around exact PCB positions. Use Xbox Series X/S external dimensions as ergonomic reference skeleton so grip geometry is proven without designing from scratch.
- **TPU grip inlays** — printed directly into shell design as soft grip surfaces, replacing rubber overmold on donor shells
- **ERM motor cradles** — TPU, vibration dampening between motors and shell walls, protecting TMR sensors from magnetic/mechanical interference
- **Dummy boards** — flat prints at exact PCB footprint to verify internal fit before ordering fabrication
- **D-pad pivot post** — PETG (already specced)
- **Custom standoffs and PCB mounting features** — designed into the shell itself rather than retrofitted
- **JST cable strain relief** — TPU exits at each daughterboard connector

**Shell design approach:**
1. Print a rough ergonomic mockup first — low infill, fast, just to check grip feel in hand. Iterate on shape before adding button wells and mounting features.
2. Once grip geometry is confirmed, add all internal features (PCB standoffs, trigger pivot points, button wells, battery cavity).
3. Final print: PETG at 3mm wall thickness minimum, 40%+ infill for structural sections, 0.2mm layer height for surface quality.

**One tool to add:**
- Multimeter (~$15–20) — needed for LDO voltage verification, continuity checks, and sensor debugging

---

## Open Questions / Next Steps

- [ ] Define BOM and begin component sourcing
