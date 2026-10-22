# Building this with market parts 
Written 2026-09-21, after measuring the TX in LTspice rather than guessing.
Companion to `BOM.md` and `sim/TX_SIM_NOTES.md`.

---


## About tolerance

Every component in the transmitter was swept individually at **−30 %, nominal and
+30 %** in LTspice 26.0.2. Nominal is 7.62 V pp at 27.1350 MHz. These are measured
numbers, not estimates.

| Part | −30 % | nominal | +30 % | Frequency spread | Verdict |
|---|---|---|---|---|---|
| R1 33 k | 7.76 V | 7.62 V | 5.77 V | 27.122–27.139 | safe |
| R2 15 k | 4.97 V | 7.62 V | 7.75 V | 27.119–27.137 | safe |
| R3 56 Ω | 6.88 V | 7.62 V | 7.96 V | 27.127–27.138 | safe |
| R4 390 Ω | 8.32 V | 7.62 V | 6.33 V | 27.130–27.138 | safe |
| C1 39 n | 7.62 V | 7.62 V | 7.62 V | 27.134–27.135 | **no effect at all** |
| C2 33 p | 8.85 V | 7.62 V | **1.74 V** | 27.117–27.146 | **critical** |
| C3 82 p | 7.04 V | 7.62 V | 8.06 V | 27.133–27.136 | safe |
| C4 10 n | 7.64 V | 7.62 V | 7.61 V | 27.134–27.136 | **no effect at all** |
| C5 15 p | 7.67 V | 7.62 V | 7.56 V | 27.135–27.136 | negligible |
| L1 1.10 µH | 3.37 V | 7.62 V | **0.03 V** | 27.097–27.136 | **critical** |

### Result 1: no passive tolerance can take this transmitter off channel

Across **every** component at ±30 %, the carrier stays inside
**27.117 – 27.146 MHz**. That is a 29 kHz spread, in a band
(26.957–27.283 MHz) that is **326 kHz wide**. The crystal is holding the
frequency and it does not care what your capacitors measure.

C1 and C4 are the clearest case: 30 % either way changes the output by **10 mV out
of 7.62 V**. Their only job is to be a short circuit at 27 MHz (0.15 Ω and 0.59 Ω),
and they still are at half their marked value. Do not waste a single taka
selecting those.

### Result 2: only two parts can actually kill the transmitter

**C2 (33 pF) and L1.** Both of them 30 % *high* takes the output from 7.62 V to
1.74 V and 0.03 V respectively — from working to dead. Both −30 % is survivable.

The pattern is the same in both cases: **the tank must not end up tuned below the
crystal.** C2 high and L1 high both drag tank resonance down, and the oscillator
stops. Everything else in the transmitter has margin to spare.

### What tolerance *does* affect is amplitude and whether it starts at all

| R3 (nominal 56 Ω) | Collector swing |
|---|---|
| 56 Ω | 6.8 V pp |
| 39 Ω (−30 %) | 5.4 V pp |
| 27 Ω (−52 %) | 0.6 V pp, barely alive |
| 15 Ω | **dead** |

| CV1 | Collector swing | Carrier |
|---|---|---|
| 5 pF | **dead** | — |
| 12 pF | 5.8 V pp | 27.1107 MHz |
| 20 pF | 6.8 V pp | 27.1138 MHz |
| 30 pF | 7.4 V pp | 27.1176 MHz |

So R3 tolerates about −30 % before it is in trouble, and dies at −50 %. A ±5 % or
even ±20 % resistor is nowhere near that cliff. **R3 does not need trimming; it
needs to not be wildly wrong**, which a multimeter check takes ten seconds to
confirm.

Note also that CV1's whole useful range only moves the carrier **7 kHz**. That is
your netting adjustment — and it is the *only* frequency adjustment in the
transmitter.

---

## The real risks, in priority order

Ranked by how likely they are to cost you a working radio:

### 1. The transistors — by a wide margin

Remarked and counterfeit small-signal transistors are common, and a fake BC547
with a low transition frequency **will not oscillate at 27 MHz at all**. There is
no symptom short of "it doesn't work", and no amount of trimming fixes it.

**Do:** buy 8–10 rather than 4. Check every one on the component tester — you are
looking at hFE (should be 200–450 for BC547B) *and* the pinout, which varies by
maker (`BUILD.md` §4 already says this). If an oscillator refuses to start, swap
the transistor before you touch anything else.

Fallback if BC547s turn out to be junk: **BF494** (RobotechShop, ৳20) is a genuine
RF part with f_T ≈ 260 MHz and is the safer choice for Q1/Q2. Different pinout.

### 2. Ceramic capacitors — mismarked, or the wrong dielectric

A "82p" that is actually 100p is common. Worse, cheap high-K ceramics (Y5V/Z5U)
lose 50–80 % of their capacitance under DC bias and over temperature — so a part
that measures right on the bench is wrong in circuit.

This matters for **C2, C3, C5** (33 p, 82 p, 15 p), which set the tank and the
feedback divider. It does **not** matter for C1 and C4 (39 n, 10 n), whose only job
is to be a short circuit at 27 MHz — at 0.15 Ω and 0.59 Ω respectively, they can be
50 % off and still be shorts.

**Do:** measure C2/C3/C5 on the component tester and select. Ask for **NP0 / C0G**
for these three if the shop knows the difference; those are stable by construction.
Remember `BOM.md` §3 already substitutes 30 p for 33 p and 100 p for 82 p, so you
are working to a target that is itself approximate — which is fine, because L1
absorbs it (see below).

### 3. Electrolytics — routinely under capacity

Market electrolytics are often a fraction of their marked value with high ESR.
Affects C16/C17/C20 in the audio section. Symptom is motorboating or weak, distorted
audio. **Do:** measure them; if a 220 µF reads 90 µF, use two in parallel or buy better.

### 4. The crystal — now that you have one, verify it

You have the crystal, so this is cheap insurance: **put it in the oscillator and
measure the actual frequency with the counter before you build the second unit.**
Two things to confirm:

- It is inside the 26.957–27.283 MHz band the brief requires.
- **You have two on the same channel.** The design needs two *identical* crystals,
  not a 455 kHz-spaced TX/RX pair — a super-regenerative receiver uses no crystal.
  If you only bought one so far, buy the second from the same batch.

If it turns out to be, say, 27.095 rather than 27.145, that is completely fine —
just update `fx` in the netlist and note the real channel in your report.

### 5. Resistors — lowest risk of anything here

Carbon film resistors are the one thing markets rarely get wrong, and this circuit
tolerates ±20 % on every one of them. Measure them if you like; do not trim them.

---

## Two changes actually worth making

Everything above says "don't redesign". These two are the exceptions.

### A. Make L1 adjustable — this is the trimmer the circuit actually needs

**This is proved, not asserted.** I took the worst case from the table above —
C2 30 % high, a 33 pF part that is really 43 pF, output collapsed to 1.74 V — and
then retuned L1 alone:

| L1 | Output | Carrier |
|---|---|---|
| 1.10 µH (nominal) | 1.74 V pp | 27.1167 MHz |
| 0.94 µH | 4.57 V pp | 27.1218 MHz |
| 0.88 µH | 5.82 V pp | 27.1290 MHz |
| **0.83 µH** | **6.65 V pp** | 27.1417 MHz |
| 0.77 µH | 6.27 V pp | 27.1439 MHz |
| 0.72 µH | 4.37 V pp | 27.1363 MHz |

**One adjustable coil recovers 87 % of full output from a 30 % capacitor error**
(6.65 V against the 7.62 V baseline), and the carrier stays on channel the whole
way. That is the entire case for adjusting L1 instead of trimming resistors: it
fixes the one failure mode that actually exists, and no resistor trimpot can fix
it at all.

Note the peak is broad — anywhere from 0.77 to 0.88 µH gives you most of it — so
you do not need a precision adjustment, just an adjustable one.

This costs nothing. L1 is hand-wound on a former, so:

- Wind it **1 turn long** (10 turns rather than 9) on a former you can slide the
  coil along, or leave the turns slightly spread.
- With the transmitter running, watch the collector on the scope (10× probe) and
  **squeeze or spread the turns for maximum amplitude**, then fix them with a drop
  of candle wax or nail varnish.
- Target is about **1.10 µH**, but tune for peak output, not for a measured value.

This replaces every resistor trimpot you were considering, and it is the standard
way these radios have always been aligned.

### B. ~~A regeneration trimmer on the RX detector~~ — WITHDRAWN 2026-09-21

An earlier version of this document suggested replacing R7 with 33 Ω in series
with a 100 Ω preset, on the reasoning that a super-regen is touchy about its
regeneration point. **That recommendation is withdrawn — it was written before the
receiver had been simulated, and the simulation disproved it.**

R7 was swept from **8 Ω to 56 Ω** with the detector running. The collector holds a
continuous 15.5–16.2 V pp at *every* value, and the quench ratio stays at 1.00
throughout. There is no regeneration point to find, because the stage never
quenches at all. A preset there would adjust nothing worth adjusting.

Sweeping C9 (10 n → 470 n) and the supply (1.2 V → 9 V) were also negative. The
receiver's problem is structural, not a component value — see
`sim/RX_SIM_NOTES.md`. **Leave R7 at 56 Ω.**

### C. Make L2 adjustable too — and to a different value from L1

L2 needs to be about **1.14 µH**, not L1's 1.10 µH. They are genuinely different
numbers: L1 is crystal-controlled, so its tank only sets *amplitude* and you tune
it for maximum output. L2 free-runs, so its tank **is** the receiver's tuning —
at 1.10 µH the detector sits at 27.63 MHz and at 1.20 µH at 26.51 MHz. Tune L2 to
actually hear the transmitter, not to a meter reading.

---

## Incoming inspection — the 30-minute version

With your multimeter and component tester, before you solder anything:

| Part | Check | Reject if |
|---|---|---|
| BC547 ×8 | hFE and pinout on the tester | hFE < 150, or tester won't identify it |
| C2, C3, C5 | capacitance on the tester | more than ±20 % off |
| C1, C4, C19 | capacitance | more than −50 % off (they are only shorts) |
| C14–C17, C20 | capacitance | more than −40 % off |
| All resistors | resistance | more than ±20 % off |
| Crystal | frequency, once the oscillator runs | outside 26.957–27.283 MHz |
| LM386 | audio stage works standalone (`BUILD.md` §6.1) | no gain, or oscillates |

Sort the good parts into two labelled sets before you start unit 1, so unit 2 is
not built from the leftovers.

---

## Revised priority: where to spend your measuring time

Given the results, the order changed. Ignore resistor tolerance entirely.

| Rank | What | Why | Time |
|---|---|---|---|
| 1 | **Test every transistor** | A fake BC547 won't oscillate at all and nothing fixes it | 10 min |
| 2 | **Measure C2 (33 p) and select the closest** | The only capacitor that can kill the TX. Err **low**, never high | 5 min |
| 3 | **Make L1 adjustable** | Recovers everything else, including a bad C2 | free |
| 4 | Measure electrolytics | Audio stage quality | 5 min |
| 5 | Verify crystal frequency on the counter | Confirms channel and that you have a matched pair | 5 min |
| — | ~~Trim resistors to exact values~~ | **No measurable benefit. Skip it.** | — |
| — | ~~Select C1, C4~~ | **Zero effect measured. Skip it.** | — |

## What has *not* been verified yet

- **Worst-case stacking** — all tank caps high *and* L1 high at once. The
  individual sweeps suggest L1 adjustment still covers it, but I have not run the
  combined corner.
- **Receiver tolerance.** The receiver has now been simulated (see
  `sim/RX_SIM_NOTES.md`) but a component-tolerance sweep of it would be premature:
  the stage does not work as intended yet, so there is nothing stable to perturb.
  Redo this once the quench question is settled on the bench.
- The audio chain around the LM386.

**One caveat that applies to everything above.** These sweeps assume the *right*
part with the *wrong value*. They say nothing about a counterfeit that is a
different device entirely — which is why testing every transistor is still rank 1
in the table above, ahead of measuring anything.
