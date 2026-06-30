# Project Nova — Controller Overview

> Plain-language reference for all inputs, sensors, and features. For full engineering specs and firmware architecture, see [SPEC.md](SPEC.md).

---

## All Physical Inputs

### Face Buttons
Four buttons on the right side of the controller face, permanent Xbox layout:

| Button | Xbox Label | DS4 Equivalent |
|---|---|---|
| Bottom | A | Cross (×) |
| Right | B | Circle (○) |
| Left | X | Square (□) |
| Top | Y | Triangle (△) |

### D-Pad
Four directional inputs (Up, Down, Left, Right), each a separate microswitch. A 3D-printed pivot post in the center physically blocks opposite directions from registering simultaneously — you can't press Left and Right at the same time, matching how official D-pads behave.

### Bumpers
- **LB / L1** — left bumper
- **RB / R1** — right bumper

Both use horizontal mouse-style microswitches (Kailh or Omron). Extremely short travel, instant tactile click, very high cycle life.

### Triggers
- **LT / L2** — left trigger
- **RT / R2** — right trigger

These are fully analog — the firmware reads exactly how far down you've pressed them, not just whether they're pressed. Squeezing LT halfway reports 50% depth; all the way reports 100%. No mechanical wear because they use a magnet and a Hall effect sensor instead of a physical sliding contact.

### Thumbsticks
- **Left stick** — also clickable (L3)
- **Right stick** — also clickable (R3)

The sticks use TMR (Tunnel Magnetoresistance) modules. Unlike the potentiometer tracks inside standard controller sticks, TMR has no physical contact between parts. It cannot wear out, and it cannot drift. The click function (pressing the stick straight down) is a standard built-in feature of the modules.

### Center Cluster
Four buttons in the middle of the controller face:

| Button | Function |
|---|---|
| **View** | Select / map / Create/Share equivalent |
| **Menu** | Start / pause / Options equivalent |
| **Guide** | Home button — always sends the Home command, every press |
| **Sync** | Dedicated BLE slot selector — hold + face button to switch connected device |

The Sync button occupies the Share/Capture button footprint from the Xbox Series X/S shell (the small button just below the Guide button). It has no other function.

### Rear Slide Switch
A physical toggle on the back of the shell — not a button you press during play. Set it before powering on. It determines which personality the controller boots into (see Modes below).

---

## All Sensors

### TMR Joystick Modules (×2)
One per thumbstick. Each reads the stick's horizontal and vertical position as a continuous analog value on two axes. The output is a voltage that the microcontroller reads via ADC (analog-to-digital converter). Zero mechanical wear.

### Hall Effect Trigger Sensors (×2)
One per trigger. Each reads the exact depth of the trigger press as a continuous analog value. A small neodymium magnet is embedded in the trigger mechanism; the sensor reads the changing magnetic field as the trigger moves. Again, zero physical contact, zero wear.

### BMI270 IMU
A 6-axis inertial measurement unit — 3-axis gyroscope (rotation) and 3-axis accelerometer (movement). This is the gyro aiming sensor. Tilting the controller rotates the camera in games that support motion input. **Only active in DS4/PlayStation mode** — completely powered down in Xbox mode to conserve battery.

### USB-C Enumeration Detection
The firmware monitors the USB-C port for a data handshake with a host computer. This is distinct from just detecting power — a wall charger raises power on the port but has no data host behind it. Only a successful host handshake triggers wired mode.

---

## Modes & Features

### Mode 1 — XInput (Xbox)
Set by flipping the rear slide switch to the High position before powering on. The controller presents itself to Windows as a native Xbox pad — no drivers required, works immediately with Game Pass, Epic Games, and any XInput-compatible game. Gyro is suspended in this mode.

### Mode 2 — DS4 (PlayStation / Gyro)
Set by flipping the rear slide switch to the Low position before powering on. The controller presents itself as a DualShock 4. Gyro aiming is active. Works natively on macOS, Steam Deck, and Steam on Windows.

---

### Multi-Device Bonding (4 Slots)
The controller maintains four completely separate Bluetooth identities — each target device sees it as a unique, dedicated controller and will never compete with the others.

There are three distinct things you might want to do, and each has its own combo:

---

#### Switching between paired devices (daily use)
Hold **Sync**, press a face button:

| Sync + | Connects to |
|---|---|
| Y | Steam Deck |
| B | macOS |
| A | Windows PC |
| X | Spare / Mobile |

If the device is already paired, the controller connects directly — no pairing dialog, no advertising. Fast and silent. This combo **never clears bonding data**, so it is impossible to accidentally trigger a re-pair during a normal device switch.

---

#### First-time pairing (empty slot)
Same combo — **Sync + face button**. If you've never paired anything to that slot, the controller automatically starts advertising instead of trying to connect. The target device discovers it and pairs normally. No separate "enter pairing mode" step needed.

---

#### Re-pairing a slot to a new device
Hold **Sync + Guide simultaneously**, then press a face button while both are held:

| Sync + Guide + | Clears and re-pairs |
|---|---|
| Y | Steam Deck slot |
| B | macOS slot |
| A | Windows PC slot |
| X | Spare / Mobile slot |

This clears the old bonding data for that slot and immediately starts advertising for a fresh pair. Use this when you want to reassign a slot to a completely different device.

The two-button requirement (Sync + Guide held before choosing a slot) makes this hard to trigger accidentally. A normal slot switch (Sync alone) can never cause a re-pair.

**What about the old device?** BLE has no way for the controller to reach out and tell a host to forget it — that's a limitation of the protocol. So the controller clears its side of the bond, but the old host still has its copy saved. The new device pairs fine regardless. The old host will show a "Not Connected" error if it tries to auto-reconnect, and will keep that ghost entry in its Bluetooth list until you manually remove it from that device's settings. It won't block or interfere with anything — it's just a cosmetic cleanup on the old device's end.

---

The Sync button has no HID keycode — completely invisible to all host platforms. The Guide button is a pure Home button with no secondary behavior.

---

### Wired USB HID Passthrough
Plug in a USB-C cable to a **computer** and the controller automatically switches to wired mode — lower latency, no Bluetooth in use, and the battery charges at the same time.

Plugging into a **wall charger or power bank** only charges the battery — Bluetooth stays active and wired mode does not engage, because there is no computer on the other end to receive input data.

**If the battery is completely dead:**
- Plugged into wall → controller charges silently until it has enough power to boot, then starts up in BLE mode
- Plugged into a computer → same charging phase, then once there's enough power, it enumerates as a USB HID device and becomes fully playable over the wire while charging

---

### On-Board Calibration
Hold **L3 + R3 + Guide** for 3 seconds. The controller reads the exact resting position of all four analog inputs (both sticks, both triggers) at that moment and saves those values permanently as the true zero point. This corrects for any tiny physical misalignment from assembly, without needing any external app or software.

The all-four-LEDs strobe confirms the values have been written to memory.

---

### Auto Deep Sleep
Leave the controller untouched for 5 minutes and it powers down the Bluetooth radio and IMU automatically to protect the battery. Press the Guide button to wake it instantly.

---

### Rumble Haptics
Two ERM (eccentric rotating mass) motors, one in each grip, driven by a dedicated DRV8833 motor driver chip. The driver chip handles the power draw directly from the battery — the motors never stress the microcontroller's pins, even during heavy vibration bursts.

---

### LED Status Array
Four small LEDs on the face of the controller. They only activate on state changes — they are off during all normal gameplay.

| Event | What you see |
|---|---|
| XInput boot | Two quick blinks → off |
| DS4 boot | Three quick blinks → off |
| Bonding — searching | Sweeping Cylon scan (1 → 2 → 3 → 4 → 3 → 2 → 1, looping) |
| Bonding — connected | Slot number LED holds solid for 3 seconds → off |
| Calibration saved | All 4 LEDs strobe rapidly for 3 seconds → off |

LED 1 = Steam Deck slot, LED 2 = macOS slot, LED 3 = Windows slot, LED 4 = Spare/Mobile slot.

---

## Button Combo Reference

| Input | Result | Host sees |
|---|---|---|
| Guide (any press) | Home command — Steam overlay, Xbox menu, etc. | `SYSTEM_GUI` keycode |
| Sync + Y | Connect to Steam Deck (or advertise if slot empty) | Nothing |
| Sync + B | Connect to macOS (or advertise if slot empty) | Nothing |
| Sync + A | Connect to Windows PC (or advertise if slot empty) | Nothing |
| Sync + X | Connect to Spare/Mobile (or advertise if slot empty) | Nothing |
| Sync + Guide + Y | Clear Steam Deck slot → advertise for new pair | Nothing |
| Sync + Guide + B | Clear macOS slot → advertise for new pair | Nothing |
| Sync + Guide + A | Clear Windows slot → advertise for new pair | Nothing |
| Sync + Guide + X | Clear Spare/Mobile slot → advertise for new pair | Nothing |
| L3 + R3 + Guide (3 seconds) | Enter calibration mode | Nothing (HID suppressed) |
| Guide (from sleep) | Wake from deep sleep | — |

**Order of operations:**
- Sync combos: hold Sync first, then press face button. No timing threshold.
- Re-pair combos: hold Sync and Guide together first, then press face button while both are still held.
- Calibration: all three buttons must be held simultaneously for a full 3 seconds.

---

## Platform Compatibility

| Platform | Recommended Mode | Gyro Available | Connection |
|---|---|---|---|
| Windows PC (Steam / Epic / Game Pass) | XInput | No | BLE or USB-C wired |
| macOS | DS4 | Yes | BLE or USB-C wired |
| Steam Deck | DS4 | Yes | BLE or USB-C wired |
| Steam on Windows | DS4 or XInput | Yes (DS4 only) | BLE or USB-C wired |
| Mobile / Spare device | Either | DS4 only | BLE |
