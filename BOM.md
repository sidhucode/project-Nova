# Project Nova — Bill of Materials (Final Build)

Final build uses custom JLCPCB PCBs. All quantities include working spares.

---

## MCU

| Component | Spec | Qty Needed | Qty Order | Unit Price | Total | Link |
|---|---|---|---|---|---|---|
| Seeed XIAO nRF52840 | USB bootloader, integrated antenna, castellated pads for mainboard mount | 1 | 3 | ~$10.99 | ~$32.97 | [amazon.com/s?k=seeed+xiao+nrf52840](https://www.amazon.com/s?k=seeed+xiao+nrf52840) |

> **Note:** Do not use XIAO's built-in BAT charger pin (50mA, too slow for 1500mAh). Wire TP4056 OUT+ → XIAO 5V pin. Use XIAO 3V3 for all sensors.

---

## Power

| Component | Spec | Qty Needed | Qty Order | Unit Price | Total | Link |
|---|---|---|---|---|---|---|
| LiPo 1500mAh flat pack (protected) | Must be a protected cell — internal DW01A + FS8205A for overdischarge cutoff. Order first; measure actual dimensions, design shell cavity around them. | 1 | 2 | ~$13.99 | ~$27.98 | [amazon.com/s?k=1500mah+lipo+flat+protected+3.7v](https://www.amazon.com/s?k=1500mah+lipo+flat+protected+3.7v) |
| TP4056 USB-C charging module | 1A charge rate, USB-C input variant | 1 | 3 | ~$2.33 (3-pack) | ~$6.99 | [amazon.com/s?k=tp4056+usb-c+charging+module](https://www.amazon.com/s?k=tp4056+usb-c+charging+module) |
| ~~AMS1117-3.3~~| **REMOVED.** AMS1117 requires 4.5V minimum input; LiPo peaks at 4.2V — browns out immediately. Use XIAO's built-in XC6210 (160–200mV dropout). Wire TP4056 OUT+ → XIAO 5V pin. | — | — | $0 | $0 | — |
| USB-C breakout (data + power) | **Adafruit #4090.** Exposes D+, D−, VBUS, GND. VBUS+GND → TP4056; D+/D− → XIAO. Single port handles both charging and wired HID. | 1 | 2 | $2.95 | $5.90 | [adafruit.com/product/4090](https://www.adafruit.com/product/4090) |
| JST-PH 2.0mm 2-pin connector pair | TP4056 → battery pigtail | 1 pair | 5 pairs | ~$0.70/pair | ~$6.99 | [amazon.com/s?k=jst+ph+2.0mm+2+pin+connector+pair](https://www.amazon.com/s?k=jst+ph+2.0mm+2+pin+connector+pair) |

---

## Motion

| Component | Spec | Qty Needed | Qty Order | Unit Price | Total | Link |
|---|---|---|---|---|---|---|
| BMI270 IMU breakout | SparkFun SEN-22397, I2C/Qwiic. **Verify pull-ups:** most boards include 4.7kΩ on SDA/SCL. If not, add externally — without them the gyro appears completely unresponsive. | 1 | 2 | ~$14.95 | ~$29.90 | [sparkfun.com/products/22397](https://www.sparkfun.com/products/22397) (order via mouser.com — search "SEN-22397") |

---

## Inputs

| Component | Spec | Qty Needed | Qty Order | Unit Price | Total | Link |
|---|---|---|---|---|---|---|
| TMR joystick modules | **GuliKit TMR, ALPS pinout.** Drift-free. **Verify listing is the bare TMR sensor module, not a complete thumbstick replacement assembly.** If bare module unavailable on Amazon, order from gulikit.com directly. | 2 | 4 (2 spares) | ~$15/unit | ~$59.98 | [amazon.com/s?k=gulikit+electromagnetic+joystick+module](https://www.amazon.com/s?k=gulikit+electromagnetic+joystick+module) |
| Hall effect sensor | **AH49E**, TO-92S, linear output. One per trigger. | 2 | 10 | ~$0.80 | ~$7.99 | [amazon.com/s?k=AH49E+hall+effect+sensor](https://www.amazon.com/s?k=AH49E+hall+effect+sensor) |
| Neodymium trigger magnets | 3mm × 2mm disc, N52 grade. Embedded in trigger arm. | 2 | 6 | ~$0.14 | ~$6.99 (50-pack) | [amazon.com/s?k=neodymium+disc+magnet+3mm+2mm+N52](https://www.amazon.com/s?k=neodymium+disc+magnet+3mm+2mm+N52) |
| D-pad / face / center microswitches | SPST tactile, 6mm square, through-hole, low actuation force | 12 | 25 | ~$0.32 | ~$7.99 (25-pack) | [amazon.com/s?k=6mm+tactile+push+button+microswitch+assortment](https://www.amazon.com/s?k=6mm+tactile+push+button+microswitch+assortment) |
| Bumper microswitches (LB / RB) | **Kailh horizontal mouse switch**, 2-pin, sub-mm travel | 2 | 10 | ~$1.00 | ~$9.99 (10-pack) | [amazon.com/s?k=kailh+mouse+microswitch+horizontal](https://www.amazon.com/s?k=kailh+mouse+microswitch+horizontal) |
| SPDT slide switch | Through-hole, 3-pin, rear-panel mount | 1 | 5 | ~$1.20 | ~$5.99 (5-pack) | [amazon.com/s?k=spdt+slide+switch+3+pin+through+hole](https://www.amazon.com/s?k=spdt+slide+switch+3+pin+through+hole) |

---

## Trigger Mechanism (3D Printed + Hardware)

| Component | Spec | Qty Needed | Qty Order | Unit Price | Total | Source |
|---|---|---|---|---|---|---|
| Trigger arm | PETG, designed as part of shell CAD. Left/right mirrored. Embeds neodymium magnet. | 2 | Print extras | filament only | — | 3D printed |
| Trigger pivot axle | 2mm diameter steel dowel pin, 20–25mm length | 2 | 10 | ~$0.85 | ~$8.50 | mcmaster.com — search "2mm diameter steel dowel pin 25mm" |
| Trigger return spring | Compression spring ~4mm OD, ~10–15mm free length, light return force | 2 | 10 | ~$0.80 | ~$8.00 | mcmaster.com — search "compression spring 4mm OD 15mm" |

---

## Haptics

| Component | Spec | Qty Needed | Qty Order | Unit Price | Total | Link |
|---|---|---|---|---|---|---|
| ERM coin vibration motor | 10mm diameter, 3V nominal. One per grip. | 2 | 4 | ~$1.50 | ~$8.99 (6-pack) | [amazon.com/s?k=erm+coin+vibration+motor+10mm+3v](https://www.amazon.com/s?k=erm+coin+vibration+motor+10mm+3v) |
| DRV8833 motor driver breakout | Dual H-bridge. **VM pin → TP4056 OUT+ (battery rail, 3.7–4.2V). NOT XIAO 3.3V.** Motor current spikes will brown out MCU if VM is on the 3.3V rail. | 1 | 2 | ~$3.50 | ~$6.99 (2-pack) | [amazon.com/s?k=drv8833+motor+driver+breakout](https://www.amazon.com/s?k=drv8833+motor+driver+breakout) |

---

## Telemetry

| Component | Spec | Qty Needed | Qty Order | Unit Price | Total | Link |
|---|---|---|---|---|---|---|
| 0603 SMD LED | Choose one color; 3.3V forward voltage preferred | 4 | 20+ | ~$0.05 | ~$7.99 kit | [amazon.com/s?k=0603+smd+led+assortment](https://www.amazon.com/s?k=0603+smd+led+assortment) |

> Light pipes removed — shell is 3D printed, LEDs position flush to surface.

---

## Passive Components

| Component | Values | Qty Order | Unit Price | Total | Link |
|---|---|---|---|---|---|
| Ceramic capacitors | 100nF (0402/0603) — decoupling per IC. 10µF (0603) — bulk caps on 3.3V and VBUS rails. | 50× 100nF + 20× 10µF | kit | ~$8.99 | [amazon.com/s?k=smd+ceramic+capacitor+assortment+0402+0603](https://www.amazon.com/s?k=smd+ceramic+capacitor+assortment+0402+0603) |
| Resistors | 4.7kΩ — I2C pull-ups (×2). 68Ω — LED current limiting (×4). 1kΩ — ADC input filter for Hall sensors (×2). | 10× each value | kit | ~$9.99 | [amazon.com/s?k=smd+resistor+assortment+0402+0603](https://www.amazon.com/s?k=smd+resistor+assortment+0402+0603) |

---

## Interconnects

| Component | Spec | Qty Needed | Qty Order | Unit Price | Total | Link |
|---|---|---|---|---|---|---|
| JST-SH 1.0mm pre-crimped harness kit | 2× 5-pin (sticks), 2× 3-pin (triggers), 2× 7-pin (button boards) | 8 harnesses | 1 kit | ~$15.99 | ~$15.99 | [amazon.com/s?k=jst+sh+1mm+pre-crimped+wire+harness+assortment](https://www.amazon.com/s?k=jst+sh+1mm+pre-crimped+wire+harness+assortment) |
| 28AWG silicone wire | Stranded, for point-to-point runs | ~1m | 1 spool | ~$8.99 | ~$8.99 | [amazon.com/s?k=28awg+silicone+stranded+wire](https://www.amazon.com/s?k=28awg+silicone+stranded+wire) |

---

## Shell & Mechanical Hardware

| Component | Spec | Qty Needed | Qty Order | Unit Price | Total | Source |
|---|---|---|---|---|---|---|
| M2 brass heat-set inserts | Press in with soldering iron tip | 8 | 50 | ~$0.18 | ~$9.00 | mcmaster.com — search "M2 heat set insert brass" |
| M2 × 8mm pan head machine screws | Stainless, shell halves into inserts | 8 | 25 | ~$0.12 | ~$3.00 | mcmaster.com — search "M2 x 8mm pan head machine screw" |
| M2 × 4mm self-tapping screws | PCBs into printed standoffs | 10 | 25 | ~$0.12 | ~$3.00 | mcmaster.com — search "M2 x 4mm self tapping screw" |

---

## 3D Printed Parts (Design Checklist)

| Part | Material | Qty | Notes |
|---|---|---|---|
| Shell left half | PETG | 1 | Main grip, left stick cutout, D-pad, bumper, trigger |
| Shell right half | PETG | 1 | Main grip, right stick cutout, ABXY, bumper, trigger |
| Face button caps (A/B/X/Y) | PETG | 4 | Friction-fit over microswitch plunger |
| D-pad cross cap | PETG | 1 | Rocker over 4 microswitches + pivot post |
| Center cluster caps | PETG | 4 | Menu, View, Guide, Sync |
| Bumper caps (LB/RB) | PETG | 2 | Snap over horizontal switches |
| Trigger arms (L/R) | PETG | 2 | Left + right mirror, embed neodymium magnet |
| TPU grip inlays | TPU | 2 | Left and right grip panels |
| TPU motor cradles | TPU | 2 | Vibration isolation for ERM motors |

---

## PCB Fabrication (JLCPCB)

Send all 6 designs in one batch to share shipping. Minimum 5 units per design.

| Board | Contents | Qty to Fab | Notes |
|---|---|---|---|
| Mainboard | XIAO footprint, TP4056, DRV8833, BMI270, LED pads, SWD header, all connectors | 5 | Largest board |
| Left stick daughterboard | TMR module footprint, L3 switch, 5-pin JST-SH | 5 | |
| Right stick daughterboard | TMR module footprint, R3 switch, 5-pin JST-SH | 5 | Mirror of left |
| Trigger board (shared L/R design) | AH49E footprint, 3-pin JST-SH | 5 | One design; use 2 boards |
| Button board — left | D-pad, LB, View, 7-pin JST-SH | 5 | |
| Button board — right | A/B/X/Y, RB, Menu, Guide, Sync, 7-pin JST-SH | 5 | |

**JLCPCB estimated cost:** ~$2–4/design × 6 designs = ~$12–24 + ~$20 shipping = **~$35–50 total**

---

## Tools & Consumables

| Item | Unit Price | Link |
|---|---|---|
| Multimeter (Aneng AN8002 or equivalent) | ~$17.99 | [amazon.com/s?k=aneng+AN8002+multimeter](https://www.amazon.com/s?k=aneng+AN8002+multimeter) |
| ~~Leaded solder (63/37 rosin core 0.5mm)~~ | ~~$7.99~~ | ~~purchased~~ |
| ~~Solder flux paste~~ | ~~$5.99~~ | ~~purchased~~ |
| ~~Flux remover spray~~ | ~~$7.99~~ | ~~purchased~~ |
| ~~Precision tweezers set~~ | ~~$7.99~~ | ~~purchased~~ |
| ~~Desoldering pump (solder sucker)~~ | ~~$9.99~~ | ~~purchased~~ |
| ~~Solder wick (desoldering braid)~~ | ~~$7.99~~ | ~~purchased~~ |
| ~~ESD foam cleaning swabs~~ | ~~$6.99~~ | ~~purchased~~ |
| ~~Joystick soldering iron tip~~ | ~~$9.99~~ | ~~purchased~~ |
| ~~PCB vise~~ | ~~$19.99~~ | ~~purchased~~ |
| ~~ESD (anti-static) work gloves~~ | ~~$9.99~~ | ~~purchased~~ |

> Already have: soldering station, 3D printer, PETG, PLA, TPU.
> No SWD programmer needed — XIAO flashes via USB (UF2 bootloader). SWD header is recovery-only.

---

## Cost Summary (Final Build)

| Category | Total |
|---|---|
| MCU (XIAO ×3) | ~$32.97 |
| Power system | ~$47.86 |
| Motion (BMI270 ×2) | ~$29.90 |
| TMR joystick modules (×4) | ~$59.98 |
| Hall sensors + trigger hardware | ~$31.48 |
| Switches (all types) | ~$23.97 |
| Haptics | ~$15.98 |
| Telemetry (LEDs) | ~$7.99 |
| Passives | ~$18.98 |
| Interconnects | ~$24.99 |
| Shell hardware (screws + inserts) | ~$15.00 |
| PCB fabrication (JLCPCB) | ~$35–50 |
| Tools & consumables | ~~$47.95~~ ~$17.99 (multimeter only — rest purchased) |
| **Total** | **~$362–377 shipped** |
