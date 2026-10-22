# Design Report — 27 MHz Fully Analog AM Walkie-Talkie

| Field | Detail |
|---|---|
| **Project** | `wt27_v5` — pair of 27 MHz AM handheld radios |
| **Authors** | Sarkar Maheen Ahmed, Azmain Muksit, Amir Hamza Talib, Nahinul Islam Prottoy, Ahmmed Fahim |
| **Institution** | AUST, Dhaka — Communication Engineering Lab (EEE 3207) |
| **GitHub** | [maheen-ahmed1](https://github.com/maheen-ahmed1) |
| **Tools** | KiCad 10, LTspice 26 |
| **Constraint** | Fully analog — no MCU, no RF transceiver module |
| **Hardware status** | Not built — blocked by parts availability |
| **License** | MIT — see [`LICENSE`](LICENSE) |
| **Date** | 2026 |



---

## 1. Objective

Design two identical **27 MHz AM** walkie-talkies that a student can understand end-to-end:

1. Crystal-controlled carrier generation  
2. Amplitude modulation without an audio transformer  
3. Analog detection and audio recovery  
4. Shared LM386 for mic→TX and RX→speaker via a 4-pole PTT  

**Non-goals:** using nRF24 / Si4432 / ESP32 radios, or shipping a fabbed PCB. Target construction was **veroboard**.

---

## 2. System architecture

![Block schematic](./out/wt27_BLOCKS.png)

![Fully wired schematic](./out/wt27_WIRED.png)

| Block | Function |
|---|---|
| **TX (Q1)** | Crystal Colpitts @ ~27 MHz; supply taken from LM386 output → AM |
| **RX (Q2)** | Free-running Colpitts intended as super-regenerative detector |
| **Audio (U1)** | LM386, gain 200 — speaker driver on RX, modulator on TX |
| **Mic** | Electret + bias |
| **PTT** | 4PDT: antenna, RX rail, AF source, LM386 destination |
| **Power** | 9 V battery + switch + decoupling |

---

## 3. Key design decisions

| Decision | Reasoning |
|---|---|
| **No MCU / RF module** | Learn the radio chain, not a datasheet wrapper |
| **Super-regen RX** (not superhet) | Lower complexity (~2 tuned circuits); IF transformers hard to source locally |
| **Separate Q1 / Q2** | Avoids classic shared-stage “pull the crystal” failure mode |
| **LM386 supply AM** | No modulation transformer; electret instead of speaker-as-mic |
| **Veroboard, not PCB** | Match build method and parts budget |
| **Quench mod ready** | Super-regen is simple but unreliable; external quench if self-quench fails |
| **Superhet as upgrade** | If a more reliable RX is required later |

---

## 4. Simulation results

### 4.1 Transmitter — pass

![TX LTspice](./sim/tx_sim.png)

| Metric | Value |
|---|---|
| Carrier `fosc` | **27.1500 MHz** |
| AM index `mod_depth` | **0.497** (target 0.5) |
| Envelope linearity | ≈ 2.98 vs theoretical 3.00 at *m* = 0.5 |
| Antenna (50 Ω) | ~0.276 Vpp |

**Finding:** Collector-supply AM works; the oscillator *is* the modulator. See [`sim/TX_SIM_NOTES.md`](sim/TX_SIM_NOTES.md).

### 4.2 Receiver 

![RX LTspice](./sim/rx_sim.png)

| Observation | Result |
|---|---|
| Quench bursting | **Absent** — continuous ~16 Vpp oscillation |
| `quench_test` ratio | ≈ **1.00** (should be ≫ 1 if bursting) |
| Sensitivity | Audio identical for **10 µV** and **50 mV** input |

**Finding:** Stage behaves as a plain regenerative oscillator, not a super-regen. Bias has nothing that switches it off. See [`sim/RX_SIM_NOTES.md`](sim/RX_SIM_NOTES.md).

### 4.3 Quench modification — designed fix

NE555 astable chops Q2 supply (~30 kHz, **50 %** duty with diode). Sim restores monotonic audio vs antenna level down to 10 µV. Documented in [`QUENCH_MOD.md`](QUENCH_MOD.md); not yet drawn into the main schematic (bench never reached).

### 4.4 Parts tolerance (TX ±30 % sweeps)

With no hardware, accuracy questions were answered in LTspice → [`PARTS_TOLERANCE.md`](PARTS_TOLERANCE.md):

- Frequency stays in-band across all sweeps (crystal dominates).  
- Only **C2** and **L1** critically kill amplitude when high.  
- Trimming RF resistors with pots is usually harmful.

---

## 5. Update

| Delivered | Not delivered |
|---|---|
| KiCad schematic (ERC 0/0) | Physical veroboard units |
| TX / RX LTspice + notes | On-air range test |
| Tolerance study | Quench drawn into main `.kicad_sch` |
| Costed BOM (local stores) | |
| Full staged [`BUILD.md`](BUILD.md) | |
| Quench design + cost | |

**Why no hardware:** crystals, PTT switches, and other market parts could not be obtained reliably in the project window. Simulation and documentation became the primary deliverable.

---

## 6. Skills demonstrated

- Discrete RF oscillator design (crystal Colpitts)  
- Analog AM via supply modulation  
- Super-regenerative vs superheterodyne trade-offs  
- LTspice RF transient analysis and `.meas` checks  
- Design-for-market-parts / tolerance thinking  
- KiCad schematic capture, ERC, exports  
- Engineering write-ups (failure analysis, not only success)

---

## 7. Future work

1. Source parts and run [`BUILD.md`](BUILD.md) stage 6.3 (RX quench check).  
2. Fit quench mod if needed; re-export schematic.  
3. Or redesign RX as **superhet** if reliability/selectivity is the priority.  
4. Optional: whip antenna / range characterization once both units work.

---

## 8. Document index

| Doc | Role |
|---|---|
| [`README.md`](README.md) | Repo overview + figures |
| [`BUILD.md`](BUILD.md) | Veroboard build / debug manual |
| [`BOM.md`](BOM.md) | Costed parts list |
| [`PARTS_TOLERANCE.md`](PARTS_TOLERANCE.md) | What must be accurate |
| [`QUENCH_MOD.md`](QUENCH_MOD.md) | External quench |
| [`HANDOFF.md`](HANDOFF.md) | Decision history |
| [`LICENSE`](LICENSE) | MIT |
