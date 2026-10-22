# The quench oscillator — design, evidence, and what is still unproven

Written 2026-09-21. **Build this only if `BUILD.md` stage 6.3 confirms the detector does not
self-quench** (`sim/RX_SIM_NOTES.md`). Simulation says it won't, at ~70 % confidence.

Netlists: `scratchpad/q1..q6.net` were the working files; the mechanism proof is
reproduced below.

---

## 1. What it does

`sim/RX_SIM_NOTES.md` established that Q2 oscillates continuously and therefore
ignores the antenna — audio output was identical for 10 µV and 50 mV inputs. A
super-regenerative detector needs to be **switched fully off and on** at 20–50 kHz;
the gain comes from how fast it rebuilds each time, which a few microvolts of
incoming signal dominates.

This adds the missing switch: a 555 astable chopping Q2's supply rail.

---

## 2. The evidence that it works

**This is the important result.** Ideal square-wave quench, 40 kHz, 50 % duty,
supply chopped 0 ↔ 9 V, measured at the R9/C11 audio node:

| Antenna input | V(aud) p-p |
|---|---|
| none | 0.1934 V |
| 10 µV | 0.2351 V |
| 100 µV | 0.2551 V |
| 1 mV | 0.2799 V |

**Monotonic across a 100:1 input range, and it responds to 10 µV.** Compare the
un-quenched detector, where 10 µV and 50 mV both gave 7.7 mV and switching the
antenna off entirely changed the output by 82 µV. The mechanism is real.

The response is compressive rather than linear — each decade of input adds a
roughly constant increment. That is textbook **logarithmic-mode** super-regeneration,
which is what a hard on/off quench gives you, and it behaves like built-in AGC.

**Duty cycle turns out to be critical.** At 32 kHz, comparing recovered 2 kHz audio
with and without a 100 µV signal:

| Duty | Response to signal |
|---|---|
| **50 %** | **1.90×** |
| 67 % | 1.04× — essentially deaf |

A plain 555 astable gives 67 %. **It needs the diode.**

Quench frequency sweep at 100 µV input favoured **30 kHz** (21.8 mV recovered)
over 20 kHz (18.0 mV), 40 kHz (9.6 mV) and 60 kHz (9.2 mV).

---

## 3. The circuit

```
        +9V ──┬──────────┬──────────┐
              │          │          │
             R1 33k      │       ┌──┴──┐ 8
              ├────┬─────┼───────┤4 RST│
              │   ▼ D1   │       │     │3
             R2 33k  1N4148      │ 555 ├───── to Q2 supply
              │   (across R2)    │     │      (L2 pin 1 / R5)
              ├──────────┼───────┤6 TH │
              │          │       │2 TR │
              C1 680p    │    ┌──┤5 CV │
              │          │  10n  │     │1
             GND        GND GND └──┬───┘
                                  GND
```

Standard 555 astable with **D1 across R2** so it charges through R1 only and
discharges through R2 — that is what gets duty to 50 %.

- **f = 1.44 / ((R1 + R2) × C1)** = 1.44 / (66 k × 680 p) = **32 kHz**
- **duty = R1 / (R1 + R2)** = **50 %** with R1 = R2

Pin 3 replaces the wire from SW2 pole B to L2 pin 1. The detector draws ~5 mA; a
555 sources 200 mA, so it drives it directly. Output high is about Vcc − 1.7 ≈
7.3 V, which is ample — the detector oscillates happily from 6 V up.

**Power the 555 from the receiver supply — SW2 pole B, pin 6 — not the always-on
+9 V rail** (the "+9V" in the diagram above means that point). Then pressing PTT
switches off the 555 and the detector together. Powered from the always-on rail,
the detector would keep oscillating while you transmit, a few centimetres from the
transmitter.

**Diode direction:** the band (cathode) goes towards pins 6/2; the other end towards
pin 7. Backwards, the duty cycle is wrong and the receiver is nearly deaf.

Optional: a 100 nF capacitor from pin 8 to pin 1, right at the chip, keeps the 555's
switching spikes off the supply. Step-by-step wiring: `BUILD.md` §6.4.

### Parts, per unit

| Part | Value | Source | Cost |
|---|---|---|---|
| IC | NE555 | [RobotechShop ৳15](https://robotechshop.com/shop/components/general-ics/timer-ic-555/) | ৳15 |
| R1, R2 | 33 kΩ ×2 | already in your 33k pack of 20 | — |
| D1 | 1N4148 | [TechShopBD ৳1.50](https://techshopbd.com/product/diode-1n4148) | ৳1.50 |
| C1 | 680 pF = **560 p ∥ 120 p** | [560 p](https://techshopbd.com/product/ceramic-capacitor-561-560pf) + [120 p](https://techshopbd.com/product/ceramic-capacitor-121-120pf), TechShopBD | ৳2.09 |
| C2 | 10 nF (pin 5) | already in your order | — |
| socket | DIP-8 (14-pin cut down) | already in order B | — |

A single 680 pF part is out of stock online as of 2026-09-21; 560 pF and 120 pF in
parallel give the same value. Solder them side by side between pins 6/2 and ground.
At Nawabpur you can simply buy a 680 pF.

**Also buy 470 pF and 1 nF** (৳1.09 and ৳2.04) — see §4. About **৳40 for both
units**, and the 1N4148 is the one I earlier told you to drop from the BOM. It is
back.

---

## 4. Tune the quench frequency on the bench — do not trust my number

**I could not pin the optimum in simulation, and you should know why.** The audio
measurement at the R9/C11 node is contaminated by quench residue leaking into the
same frequency region as the recovered audio. Small changes in quench frequency
shift how that residue lands, so the results stopped ordering cleanly: at 30 kHz
with realistic 555 output levels, every signal level gave more output than no
signal (1.62×, 1.35×, 1.17× for 10 µV / 100 µV / 1 mV) but not in a clean order.

Some of that is genuine log-mode compression and some is measurement artefact, and
I cannot separate them with this setup.

**This matters less than it sounds**, because quench frequency on a super-regen is
universally set by ear on the bench anyway. The practical method:

1. Keep R1 = R2 = 33 k so duty stays at 50 % — **that part is not negotiable**, it
   is the difference between 1.90× and 1.04× response.
2. Change only C1 to move the frequency:
   - **470 pF → 46 kHz**
   - **680 pF → 32 kHz** (start here)
   - **1 nF → 22 kHz**
3. Fit whichever gives the loudest, cleanest hiss with no signal and the best
   audio with the far unit transmitting.

Socket C1, or just tack-solder and swap.

---

## 5. What is verified and what is not

**Verified in simulation:**
- Chopping the supply makes a dead detector respond to a 10 µV signal, monotonically
  across 100:1. This is the load-bearing claim and it is solid.
- 50 % duty is required; 67 % is nearly deaf.
- ~30 kHz is better than 20, 40 or 60 kHz.
- The detector still works from the 555's real output levels (0.3–7.3 V) through a
  20 Ω source impedance.

**Not verified:**
- **The 555 itself is not simulated.** I modelled its output as a pulse source with
  realistic levels and impedance, not an actual 555 model. The astable equations
  are textbook and the loading is trivial, so the risk is low — but it is not zero.
- The exact optimum quench frequency (see §4).
- Interaction with the audio chain and the LM386, which has not been simulated at all.
- Whether the real detector self-quenches after all, which would make this whole
  document unnecessary.

**If it does not work on the bench**, the first two things to try are swapping C1
(§4) and checking that the quench is actually reaching L2 pin 1 as a clean
0→7 V square — scope pin 3 directly.
