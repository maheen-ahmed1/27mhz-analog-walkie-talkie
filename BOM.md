# 27 MHz Walkie-Talkie — Bill of Materials

Source: `wt27_v5.kicad_sch` → `out/wt27_v5.net` (46 designators, 34 nets).
**Quantities are for TWO units** unless a column says otherwise.
Stock and prices **checked live 2026-09-19** at your four preferred stores.
Machine-readable copy: `BOM.csv`.

Stores:
[TechShopBD](https://techshopbd.com) ·
[RobotechShop](https://robotechshop.com) ·
[RoboticsBD](https://store.roboticsbd.com) ·
[Electronics.com.bd](https://electronics.com.bd)

---

## 0. Buy-order summary

> **Updated 2026-09-21 — `BOM.csv` is now the authoritative order list.** Stock was
> re-checked; the tables further down this file predate some of the changes below.

| Order | Items | Cost |
|---|---|---|
| **A — TechShopBD** | all passives, BC547 ×10, LM386, electret, veroboard, battery, quench-mod passives + 1N4148, test-jig parts | **৳587** |
| **B — RobotechShop** | CV1 trimmers ×4 (40 pF variant), NE555 ×2, 14-pin sockets ×4 | **৳142** |
| **D — Electronics.com.bd** | 27 MHz crystals ×2 | **৳36** |
| **C — Nawabpur / Stadium Market** (in person) | PTT switches, speakers, whips, enamelled wire | **~৳615 est.** |

**Total ≈ ৳1,380 for both units**, of which ৳43 is the quench modification.

**Changes since the first version of this BOM:**
- **Crystal moved online** (order D, ৳18 each) — see §4b.
- **CV1: the 4–27 pF trimmer went out of stock.** Use the RF trimmer, **40 pF
  variant** (৳20) — it covers the 12–30 pF netting range. The 20 pF variant only
  reaches mid-range.
- **8-pin DIP sockets went out of stock.** Buy the 14-pin variant and cut down.
  Now 4 needed: LM386 ×2 and NE555 ×2.
- **680 pF (quench C1) is out of stock.** Fit **560 pF ∥ 120 pF = 680 pF**, both in
  stock at TechShopBD. Keeps the verified 32 kHz.
- **1N4148 from TechShopBD** at ৳1.50, same order, rather than RobotechShop.
- **BC547 ×4 → ×10** — test every one, plus one for the crystal test jig.
- Quench-mod and crystal-test-jig parts added (`QUENCH_MOD.md`, `BUILD.md` §6.2).

---

## 1. Order A — TechShopBD

Every line below was confirmed **in stock** on 2026-09-19.

### Semiconductors

| Ref | Qty | Part | Unit | Total | Link |
|---|---|---|---|---|---|
| Q1, Q2 | 4 | BC547 NPN TO-92 | ৳5.04 | ৳20.16 | [BC547 NPN Transistor](https://techshopbd.com/product/bc547-npn-transistor) |
| U1 | 2 | LM386 audio power amp, DIP-8 | ৳20.05 | ৳40.10 | [LM386 OPAMP](https://techshopbd.com/product/lm386-opamp) |
| MK1 | 2 | Electret microphone capsule | ৳20.04 | ৳40.08 | [Electret Microphone](https://techshopbd.com/product/electret-microphone) |



### Resistors — 1/4 W, sold in packs of 20

One pack of each covers both units with spares (11 resistors per unit).

| Ref | Value | Qty needed | Pack | Link |
|---|---|---|---|---|
| R1, R5 | 33 kΩ | 4 | ৳10.04 | [33K Ohm 1/4W ×20](https://techshopbd.com/product/33k-ohm-14w-resistor-pack-of-20-1) |
| R2, R6 | 15 kΩ | 4 | ৳10.04 | [15K Ohm 1/4W ×20](https://techshopbd.com/product/15k-ohm-14w-resistor-pack-of-20-1) |
| R3, R7 | 56 Ω | 4 | ৳9.99 | [56 Ohm 1/4W ×20](https://techshopbd.com/product/56-ohm-14w-resistor-pack-of-20) |
| R4, R8 | 390 Ω | 4 | ৳10.39 | [390 Ohm 1/4W ×20](https://techshopbd.com/product/390-ohm-14w-resistor-pack-of-20) |
| R9 | 10 kΩ | 2 | ৳10.04 | [10K Ohm 1/4W ×20](https://techshopbd.com/product/10k-ohm-14w-resistor-pack-of-20) |
| R10 | 4.7 kΩ | 2 | ৳15.07 | [4.7K Ohm 1/4W ×20](https://techshopbd.com/product/47k-ohm-14w-resistor-pack-of-20) |
| R11 | 10 Ω | 2 | ৳10.30 | [10 Ohm 1/4W ×20](https://techshopbd.com/product/10-ohm-14w-resistor-pack-of-20) |

**Subtotal ৳75.87.**



### Capacitors

| Ref | Value | Qty | Buy | Unit | Total | Link |
|---|---|---|---|---|---|---|
| C5, C10 | 15 pF | 4 | 15 pF ceramic | ৳1.09 | ৳4.36 | [Ceramic 15pF (150)](https://techshopbd.com/product/ceramic-capacitor-150-15pf) |
| C2, C7 | 33 pF | 4 | **30 pF** ⚠️ | ৳1.99 | ৳7.96 | [Ceramic 30pF (300)](https://techshopbd.com/product/ceramic-capacitor-300-30pf) |
| C3, C8 | 82 pF | 4 | **100 pF** ⚠️ | ৳1.50 | ৳6.00 | [Ceramic 100pF (101)](https://techshopbd.com/product/ceramic-capacitor-101-100pf) |
| C4, C9 | 10 nF | 4 | 10 nF ceramic | ৳1.50 | ৳6.00 | [Ceramic 10nF (103)](https://techshopbd.com/product/ceramic-capacitor-103-10nf) |
| C1, C6 | 39 nF | 4 | **33 nF** ⚠️ | ৳1.20 | ৳4.80 | [Ceramic 33nF (333)](https://techshopbd.com/product/ceramic-capacitor-333-33nf) |
| C18 | 47 nF | 2 | **33 nF** ⚠️ | ৳1.20 | ৳2.40 | same as above (buy 6 total) |
| C11 | 4.7 nF | 2 | 0.0047 µF 100 V | ৳2.50 | ৳5.00 | [0.0047uF 100V](https://techshopbd.com/product/00047uf-100v-capacitor) |
| C19 | 100 nF | 2 | 100 nF ceramic | ৳1.50 | ৳3.00 | [Ceramic 100nF (104)](https://techshopbd.com/product/ceramic-capacitor-104-100nf) |
| C12 | 0.47 µF | 2 | **1 µF 50 V** ⚠️ | ৳1.54 | ৳3.08 | [1uF 50V](https://techshopbd.com/product/1uf-50v-capacitor) |
| C13 | 1 µF | 2 | 1 µF 50 V | ৳1.54 | ৳3.08 | same as above (buy 4 total) |
| C14, C15 | 10 µF | 4 | 10 µF 25 V elec. | ৳2.50 | ৳10.00 | [10uF 25V](https://techshopbd.com/product/10uf-25v-capacitor) |
| C16, C20 | 100 µF | 4 | 100 µF 16 V elec. | ৳3.02 | ৳12.08 | [100uF 16V](https://techshopbd.com/product/100uf-16v-capacitor) |
| C17 | 220 µF | 2 | 220 µF 16 V elec. | ৳5.02 | ৳10.04 | [220uF 16V](https://techshopbd.com/product/220uf-16v-capacitor) |

**Subtotal ৳77.80.** The ⚠️ lines are value substitutions — justified in §3.

### Electromechanical and board

| Ref | Qty | Part | Unit | Total | Link |
|---|---|---|---|---|---|
| RV1 | 2 | 10 kΩ potentiometer (volume) | ৳20.05 | ৳40.10 | [10k Potentiometer](https://techshopbd.com/product/10k-potentiometer) |
| SW1 | 2 | SPDT slide switch (use as SPST power) | ৳5.03 | ৳10.06 | [SPDT Slide Switch 0.34in](https://techshopbd.com/product/spdt-slide-switch-034in) |
| BT1 | 2 | 9 V battery | ৳65.04 | ৳130.08 | [9V Battery](https://techshopbd.com/product/9v-battery) |
| BT1 | 2 | 9 V battery snap/clip | ৳10.04 | ৳20.08 | [9V Battery Connector](https://techshopbd.com/product/9v-battery-connector) |
| — | 2 | Veroboard, single sided, **line type**, 14.5 × 6.5 cm | ৳35.04 | ৳70.08 | [Veroboard Single Sided Line Type](https://techshopbd.com/product/veroboard-single-sided-line-type) |

Line type = strips, which is what `BUILD.md` §5 assumes. **Do not buy the dot type.**
145 × 65 mm is slightly larger than the ~100 × 60 mm floorplan — that is the right
direction; do not try to make it smaller.

**Order A total: ৳524.**

---

## 2. Order B — RobotechShop

| Ref | Qty | Part | Unit | Total | Link |
|---|---|---|---|---|---|
| CV1 | 4 | Trimmer capacitor 4–27 pF 100 V (2 + 2 spare) | ৳10 | ৳40 | [Trimmer Capacitor 4-27PF 100V](https://robotechshop.com/shop/components/capacitors/trimmer-capacitor-4-27pf-100v/) |
| U1 socket | 2 | DIP-8 IC base socket (pick "8 Pin") | ৳6 | ৳12 | [IC Base Sockets / Dip Sockets](https://robotechshop.com/shop/components/ic-base-sockets-dip-sockets/) |

**Order B total: ৳52.**

CV1 is specified 5–30 pF; 4–27 pF covers it. If you would rather not pay delivery
on a ৳52 order, the alternative is
[Adjustable RF Trimmer 20/40/60/120 pF, ৳18–24](https://robotechshop.com/shop/components/capacitors/adjustable-trimmer-capacitor-variable-ceramic-capacitor-pcb-mount-for-rf-tuning/)
(pick 20 pF) — or just buy both at the market with the crystals.

The DIP-8 socket is not in the schematic but you want it: `BUILD.md` §5 has you
cutting four strips under U1, and a socket lets you pull the LM386 if the audio
stage misbehaves.

---

## 3. Value substitutions — read before you order

None of the four stores carries **33 pF, 82 pF, 39 nF or 47 nF** as through-hole
ceramics. TechShopBD's through-hole range is 10/12/15/20/22/27/30/47/56/100/120/150/
470/560/680/1000 pF and 10/15/20/33/100 nF; RobotechShop's kit stops at
22p/47p/100p/330p/1n/10n/100n; RoboticsBD has 22p/47p/104 only. Those four values
exist only as SMD 1206 (৳6 each) — unusable on veroboard.

So: either buy the exact values at Nawabpur/Stadium alongside the crystals, or
take these substitutions, all of which are defensible.

| Ref | Design | Sub | Why it is fine |
|---|---|---|---|
| C2, C7 | 33 pF | 30 pF | Colpitts feedback pair. Paired with the C3/C8 sub below, series capacitance goes 23.5 pF → 23.1 pF — a 2 % shift, absorbed by CV1 on the TX and by L2 on the RX. |
| C3, C8 | 82 pF | 100 pF | Same node. Feedback ratio moves 2.5 → 3.3, i.e. slightly *more* feedback into the emitter — a super-regen wants loop gain, so this errs the safe way. If Q1 refuses to start, drop C3 to 56 pF. |
| C1 (TX) | 39 nF | 33 nF | Base decoupling. Swept 27–51 nF with **zero** effect on the transmitter. |
| **C6 (RX)** | 39 nF | **33 nF — better than 39 nF** | **Not irrelevant — corrected 2026-09-21.** With the quench fitted, C6 sets how fast Q2's base follows the on/off switching. Simulated response to 100 µV: 4.7n 1.11×, 10n 1.14×, **22n 1.51×, 33n 1.44×**, 39n 1.32×, **47n 1.06×, 100n 0.93× (deaf)**. **Keep C6 between 22 and 39 nF — never 47 nF or larger.** |

**39 nF is also unavailable at Nawabpur** (reported 2026-09-21), so buy **33 nF
(marked "333")** for C1, C6 *and* C18 — one value, seven pieces.
| C18 | 47 nF | 33 nF | LM386 Zobel with R11. Corner moves 339 kHz → 482 kHz, both far above audio. The datasheet's own typical is 10 Ω + 50 nF and the network is there to stop RF-range instability, not to shape sound. |
| C12 | 0.47 µF | 1 µF | RX audio coupling into RV1 (10 k). High-pass corner improves 34 Hz → 16 Hz. Strictly better, and it is the same part as C13. (The 0.47 µF 50 V is out of stock at TechShopBD anyway.) |

**Do not substitute C5/C10 (15 pF) or C11 (4.7 nF).** C5/C10 are the RF output
coupling and C11 sets the 3.4 kHz quench filter with R9 — both are exact-value parts
and both are in stock.

### These substitutions are verified, not just argued — 2026-09-21

The transmitter was re-simulated with the substitute values actually fitted
(C2 = 30 p, C3 = 100 p, C1 = 33 n) rather than the schematic's values:

| L1 | Peak swing | AM index | Carrier |
|---|---|---|---|
| 1.00 µH | 12.77 V | 0.500 | 27.1663 MHz |
| **1.05 µH** | **13.09 V** | 0.508 | 27.1593 MHz |
| 1.10 µH | 12.97 V | 0.509 | 27.1550 MHz |
| 1.15 µH | 12.61 V | 0.506 | 27.1518 MHz |

Baseline with the schematic's own values was 11.49 V and index 0.497. **The
substituted build is slightly stronger, and the carrier stays inside
27.148–27.168 MHz across the whole range** — comfortably in band.

The optimum coil shifts from 1.10 µH to about **1.05 µH** with these parts. The
difference between the two is under 1 %, so wind for 1.05–1.10 µH and peak it on
the scope; the adjustable coil covers the rest.

---

## 4. Order C — in person, Nawabpur Road / Stadium Market

Not obtainable from any of the four stores. **This is the critical path.**

| Ref | Qty | Part | Est. | Notes |
|---|---|---|---|---|
| ~~**Y1**~~ | — | — | — | **MOVED ONLINE 2026-09-21 — see below.** |
| SW2+SW3 | 2 | **4PDT momentary PTT switch** | ৳50–100 ea | Needs 4 poles: antenna, RX supply, LM386 input, LM386 output. Nothing online is 4PDT momentary. A 4PDT slide/toggle works if you accept latching instead of push-to-talk. |
| LS1 | 2 | 8 Ω 0.5–1 W speaker, 50–57 mm | ৳30–60 ea | **Both TechShopBD speakers are out of stock** (8 Ω 0.25 W ৳20.31 and 8 Ω 2 W ৳70.81), and RobotechShop's in-stock ones are 32 Ω or 4 Ω. Buy at the market. |
| ANT1 | 2 | Telescopic whip ~1 m | ৳100–200 ea | Not stocked anywhere online. `BUILD.md` §6.9 has you range-testing on 2.7 m wire first anyway — the whips are for the demo. |
| L1, L2 | — | 0.5 mm enamelled copper wire | ৳50–100 | For the 9T/2T coils and the 3.4 µH antenna loading coil (28 T, 10 mm former). Not stocked online. |
| C2/C3/C7/C8 | — | exact 33p, 82p ceramics | ~৳20 | Optional — buy here if you would rather not take §3's substitutions. **Not 39 nF:** it is not available at the market either, and 33 nF is the better value for C6 anyway (§3). |

Also worth picking up: a spare 9 V clip, tinned wire for the ground-bus links, and
a small enclosure per unit.

---

## 4b. The crystal — found online after all, at Electronics.com.bd

Earlier notes said no 27 MHz crystal was obtainable online in Bangladesh. **That
was wrong**, and the reason is worth recording: Electronics.com.bd blocks direct
search URLs, so it was never properly searched. It stocks them.

| Part | Price | Stock | Link |
|---|---|---|---|
| **27 MHz crystal** (Model# 3067) | **৳18** | **IN STOCK** | [27 MHZ CRYSTAL OSCILLATORS](https://electronics.com.bd/27-mhz-crystal-oscillators) |
| 27.145 MHz crystal (Model# 3068) | ৳18 | **Stock out** | [27.145 Mhz Crystal Oscillator](https://electronics.com.bd/27-145-mhz-crystal-oscillator) |

Both are listed under *Components → Crystal Oscillators And Resonators →
**Quartz Crystal***, spec'd **±20 ppm, load capacitance 20 pF**. Buy **two** of
whichever is in stock, in one order so they come from the same batch.

**"Crystal Oscillator" here is loose retail naming — you want the 2-pin passive
quartz crystal, NOT a 4-pin oscillator module.** The listing's own spec confirms
it is the passive part: a load-capacitance figure only exists for a passive
crystal (a powered XO module would quote a supply voltage and output type
instead). **Check the pin count when it arrives** — a 4-pin module cannot work in
this circuit at all.

### Does 27.000 MHz instead of 27.145 MHz matter?

**No.** 27.000 MHz sits inside the 26.957–27.283 MHz band the brief requires, so
it is compliant. Consequences are all minor:

- L1 and L2 want about **1 % more inductance** (27.000 is 0.53 % below 27.145).
  Both coils are adjustable, so this is absorbed with no change of plan.
- Update `fx` in `sim/tx_am_27mhz.net` and `sim/rx_am_27mhz.net` if you re-run them.
- **Netting still works comfortably.** ±20 ppm at 27 MHz is ±543 Hz, so two
  crystals differ by at most ~1.1 kHz. CV1 gives about 7 kHz of range
  (`PARTS_TOLERANCE.md`), so there is 6× margin.

### The one real risk: fundamental vs 3rd overtone

The listing does not say which. **If it is a 3rd-overtone crystal (fundamental
~9 MHz), this Colpitts will not oscillate at 27 MHz** — it will run at 9 MHz or
refuse to start, and the whole simulation assumes a fundamental part.

Most 27 MHz crystals sold for microcontroller clocking are fundamental AT-cut
(fundamental mode is routine to 30–40 MHz), and ±20 ppm with CL = 20 pF is a
typical fundamental spec, so this is probably fine. **Confirm on the frequency
counter at `BUILD.md` stage 6.2 (the crystal test jig)** — if you read ~9 MHz, that is the answer, and the
cure is a tuned collector tank that rejects the fundamental.

## 5. Alternates and second sources

| Part | Alternate | Where | Note |
|---|---|---|---|
| BC547 (Q1, Q2) | C1815 NPN, ৳2.58 | [TechShopBD](https://techshopbd.com/product/c1815-npn-transistor) | Cheaper, same class, verified in stock. TO-92 pinout differs — check on the component tester (`BUILD.md` §4). |
| BC547 (Q1, Q2) | BF494 RF NPN, ৳20 | [RobotechShop](https://robotechshop.com/shop/components/transistors/bf494-transistor-npn-rf-amplifier-high-frequency-bipolar-junction-transistor-bf494-%e0%a6%9f%e0%a7%8d%e0%a6%b0%e0%a6%be%e0%a6%a8%e0%a6%9c%e0%a6%bf%e0%a6%b8%e0%a7%8d%e0%a6%9f%e0%a6%b0/) | f_T ≈ 260 MHz vs BC547's ~150–300 MHz. The proper RF part if either oscillator is reluctant. Pinout is *not* BC547-compatible. |
| LM386 | LM386, ৳20 | [RobotechShop](https://robotechshop.com/shop/components/general-ics/lm386-audio-power-amplifier-ic-low-voltage-op-amp-for-battery-powered-audio-projects/) | Second source at the same price. |
| Electret (MK1) | CEM-C9745 capsule, ৳25 | [RobotechShop](https://robotechshop.com/shop/sensors/electret-mic/) | Second source. |
| Electrolytics | 100 µF ৳4, 220 µF ৳5, 10 µF ×5 ৳10 | [RoboticsBD](https://store.roboticsbd.com/capacitor/) | Cheapest of the three for bulk electrolytics if you are ordering there anyway. |
| Mixed caps | Capacitor Collections, ৳5–10 per value | [RobotechShop](https://robotechshop.com/shop/components/capacitors/robotechshop-capacitor-kit/) | 22p/47p/100p/330p/1n/10n/100n + electrolytics. Does **not** cover 33p/82p/39n/47n. |

**Store search URLs**, for anything not listed here:
TechShopBD `https://techshopbd.com/search?q=TERM` ·
RobotechShop `https://robotechshop.com/?s=TERM&post_type=product` ·
RoboticsBD `https://store.roboticsbd.com/search?controller=search&s=TERM` ·
Electronics.com.bd — direct search URLs are blocked, use the on-page search box.

---

## 6. Coil targets — RESOLVED 2026-09-21

This used to be an open item. Simulation has now fixed the values:

| Coil | Primary target | Why |
|---|---|---|
| **L1** (TX) | **1.10 µH** | Crystal-controlled, so this only sets output amplitude. Tune for maximum output. 1.00–1.15 µH all work; above 1.3 µH the oscillator stops. |
| **L2** (RX) | **1.14 µH** | The detector free-runs, so this **is** the receiver's tuning. 1.10 µH puts it at 27.63 MHz; 1.20 µH at 26.51 MHz. Tune it to hear the transmitter. |

Both stay 9 turns primary / 2 turns secondary. Former diameter and gauge are still
yours to choose — wind, measure, adjust. **Wind both one turn long and leave the
turns slightly spread**: squeezing or spreading them is the only adjustment that
recovers RF capacitor tolerance, and it recovers nearly all of it
(`PARTS_TOLERANCE.md` §A). Fix with candle wax once peaked.

**Former sizes (calculated):** 9 turns of **0.3 mm on a 10 mm former = 1.10 µH**;
with 0.5 mm wire use a **12 mm** former (1.18 µH). The antenna loading coil for a
1 m whip is **~3.1 µH ≈ 22 turns of 0.5 mm or 18 turns of 0.3 mm on 10 mm** — an
earlier "28 turns" figure here was wrong (that gives ~4.2 µH). See `BUILD.md` §7–§8.

## 7. Quench oscillator parts — ADD THESE TO THE ORDER

The receiver almost certainly needs an external quench (`sim/RX_SIM_NOTES.md`).
The circuit is designed and the mechanism is simulation-verified —
**`QUENCH_MOD.md`**. Total **~৳40 for both units**, so buy them now rather than
wait for the stage 6.3 bench test and lose a day to a second delivery.

| Part | Qty (2 units) | Price | Link |
|---|---|---|---|
| NE555 timer | 2 | ৳15 ea | [RobotechShop Timer IC 555](https://robotechshop.com/shop/components/general-ics/timer-ic-555/) |
| 1N4148 diode | 1 pack of 5 | ৳12 | RobotechShop — **needed after all**, it sets the 555 to 50 % duty |
| 680 pF ceramic | 2 | ৳1.09 | [TechShopBD 681](https://techshopbd.com/product/ceramic-capacitor-681-680pf) |
| 470 pF ceramic | 2 | ৳1.09 | [TechShopBD 471](https://techshopbd.com/product/ceramic-capacitor-471-470pf) |
| 1 nF ceramic | 2 | ৳2.04 | [TechShopBD 102](https://techshopbd.com/product/ceramic-capacitor-102-1000pf) |

R1/R2 (33 k ×2 per unit), the pin-5 10 nF and the DIP-8 sockets all come from
parts already in orders A and B.

**Three capacitor values on purpose** — C1 sets the quench frequency (470 p =
46 kHz, 680 p = 32 kHz, 1 n = 22 kHz) and the optimum has to be found by ear on
the bench. `QUENCH_MOD.md` §4 explains why simulation could not settle it.

> **Correction:** an earlier version of this BOM said "no 1N4148 is needed —
> there is no diode in the netlist". True of the schematic as drawn, but the
> quench modification needs one.

---

*Generated 2026-09-19 from the verified netlist. Prices are live-checked but will drift —
re-check before ordering if more than a week passes.*
