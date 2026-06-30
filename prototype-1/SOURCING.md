# Prototype-1 Sourcing Guide — California

Organized by supplier to minimize orders and shipping costs. Fast sources first.

---

## Same Day / Walk-in

### Anchor Electronics — Santa Clara, CA
2040 Walsh Ave, Santa Clara, CA 95050 | Mon–Fri 7:30am–4pm

If you're in the Bay Area, go here for small quantities of passives you need today.

| Item | Notes |
|---|---|
| Tactile microswitches | Grab a handful, verify actuation force in person |
| Resistors (4.7kΩ, 68Ω, 1kΩ) | Buy loose if you don't want an assortment kit |
| Ceramic capacitors (100nF, 10µF) | Same |
| SPDT slide switch | Verify physical dimensions against shell design |

---

### McMaster-Carr — Santa Fe Springs, CA
9630 Norwalk Blvd, Santa Fe Springs, CA 90670 | Will-call Mon–Fri 7am–7pm, Sat 7am–3:30pm, Sun 8am–4:30pm

The definitive CA source for all mechanical hardware. Will-call same day or next-day delivery anywhere in CA.

| Item | Search Term | Notes |
|---|---|---|
| M2 heat-set inserts (brass, ×50 kit) | "M2 brass heat set insert" | Press in with soldering iron tip |
| M2×8mm machine screws (×20) | "M2 x 8mm pan head machine screw" | Shell halves |
| M2×4mm self-tapping screws (×20) | "M2 x 4mm self tapping screw" | PCB/perfboard to shell standoffs |
| 2mm steel dowel pins (×10) | "2mm diameter steel dowel pin 25mm" | Trigger pivot axles |
| Compression coil springs (~4mm OD, light) (×10) | "compression spring 4mm OD 15mm length" | Trigger return springs |

---

## 1–2 Day (Amazon Prime)

Order everything below in one cart. Prime ships from CA fulfillment centers to CA addresses in 1–2 days.

| Item | Search Term | Qty to Order |
|---|---|---|
| Seeed XIAO nRF52840 | "Seeed XIAO nRF52840" | 3 |
| Protected LiPo 1500mAh flat pack | "1500mAh lipo flat protected 3.7v" | 2 |
| TP4056 USB-C charging module | "TP4056 USB-C module 1A" | 3 |
| GuliKit TMR joystick modules | "GuliKit TMR joystick replacement" | 2 |
| Arduino joystick breakout modules | "Arduino joystick module PS2 5 pin" | 4 |
| ERM coin vibration motors 10mm | "ERM vibration motor 10mm 3V coin" | 4 |
| DRV8833 motor driver breakout | "DRV8833 dual motor driver breakout" | 2 |
| Neodymium disc magnets 3mm×2mm N52 | "neodymium disc magnet 3mm x 2mm N52" | 6 |
| Kailh horizontal mouse microswitches | "Kailh mouse microswitch horizontal" | 6 |
| Tactile microswitches 6mm (if not from Anchor) | "6mm tactile push button microswitch assortment" | 25 |
| SPDT slide switch (if not from Anchor) | "SPDT slide switch through hole 3 pin" | 3 |
| FR4 perfboard double-sided 2.54mm | "FR4 perfboard double sided fibreglass" | 2 sheets |
| SMD LED 0603 assortment | "0603 SMD LED assortment" | 1 kit |
| Ceramic capacitor assortment 0402/0603 | "ceramic capacitor assortment SMD 100nF 10uF" | 1 kit |
| Resistor assortment 0402/0603 | "SMD resistor assortment 0402" | 1 kit |
| 28AWG silicone stranded wire | "28AWG silicone wire stranded" | 1 spool |
| JST-PH 2.0mm 2-pin connector pairs | "JST PH 2.0mm 2 pin connector pair" | 5 pairs |
| M2 heat-set inserts + screws (if not McMaster) | "M2 brass heat set insert kit" | 1 kit |
| Multimeter | "Aneng AN8002 multimeter" or similar | 1 |
| Solder 63/37 0.5mm | "63/37 solder 0.5mm rosin core" | 1 spool |
| No-clean flux pen | "no clean flux pen electronics" | 1 |
| Isopropyl alcohol 90%+ | "isopropyl alcohol 90 percent" | 1 bottle |
| Fine-tip ESD tweezers | "ESD tweezers fine tip" | 1 pair |

---

## 1–2 Day (Pololu — Las Vegas, NV)

One UPS ground zone from CA. Order before 2pm Pacific for same-day dispatch.

| Item | Product | Notes |
|---|---|---|
| ERM motors (alternative, higher quality) | Search "vibration motor" on pololu.com | Better quality than Amazon coin motors if you want more reliable haptics |

> Only worth a separate Pololu order if you want better motor quality. Amazon coin motors are fine for a prototype.

---

## 1–2 Day (Jameco — Belmont, CA)

Ships from Belmont. 1-day ground to most of California. Good alternative to Anchor if you're not Bay Area local.

| Item | Notes |
|---|---|
| AH49E Hall effect sensors (×10) | Check jameco.com — they carry through-hole linear Hall sensors |
| Through-hole passives (if not from Anchor or Amazon) | Resistors, caps, switches |

---

## 2-Day Air (Mouser — order with expedited shipping)

Ships from Texas but 2-day air gets it to CA fast. Worth it for the BMI270 which isn't on Amazon.

| Item | Part / Notes |
|---|---|
| **BMI270 breakout** | SparkFun SEN-22397 (standard) or SEN-22398 (micro). Search "BMI270 breakout" on mouser.com. Both are Qwiic/I2C. |
| AH49E Hall sensors (if Jameco is out) | Search "AH49E" on mouser.com |
| Kailh horizontal mouse switches (if Amazon is out) | Search "Kailh mouse switch horizontal" |

---

## 2-Day Air (Adafruit — adafruit.com)

Ships from NYC. 4-5 days ground, 2-day air available.

| Item | Product # | Notes |
|---|---|---|
| USB-C breakout with D+/D- | **#4090** | The specific board that exposes data lines. Critical for wired HID. Verify this exact product number. |
| JST-SH 1.0mm pre-crimped harness kit | Search "JST SH 1mm" on adafruit.com | Best quality pre-crimped option. Comes in various pin counts. |

> Adafruit does **not** carry a BMI270 breakout — go to Mouser for that.

---

## Order Sequence

1. **Right now:** Amazon — everything in the Amazon list above. One cart.
2. **Same session:** Adafruit — USB-C breakout #4090. Select 2-day shipping.
3. **Same session:** Mouser — BMI270 breakout (SparkFun SEN-22397). Select 2-day air.
4. **Tomorrow or weekend:** McMaster-Carr or Anchor Electronics — hardware and passives.
5. **Optional:** Jameco — AH49E Hall sensors if you want CA-ground speed over Mouser 2-day price.

---

## What Arrives When

| Timeline | What's In Hand |
|---|---|
| Day 1–2 | Amazon order: XIAO, LiPo, TP4056, GuliKit TMR, joystick breakouts, motors, DRV8833, perfboard, wire, passives |
| Day 1–2 | McMaster-Carr: all mechanical hardware (or walk-in same day) |
| Day 2–3 | Mouser 2-day air: BMI270 breakout |
| Day 2–3 | Adafruit 2-day air: USB-C breakout #4090 |
| Day 1–2 (if local) | Anchor Electronics walk-in: passives, switches |

**You can begin bench testing the GuliKit TMR modules on the XIAO as soon as the Amazon order arrives — Day 1–2.**
