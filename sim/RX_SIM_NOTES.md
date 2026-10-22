# RX simulation — and a problem you need to know about before you build

Files: `rx_am_27mhz.net`, `rx_am_27mhz.plt`.
Companion to `TX_SIM_NOTES.md`. Run the same way:

```bash
"C:\Program Files\ADI\LTspice\LTspice.exe" -b -Run "C:\Users\name\KiCad\wt27_v5\sim\rx_am_27mhz.net"
```

~19 s. Traces: `V(msg)` / `V(ant)` / `V(coll)` / `V(aud)` — same four-pane pattern
as the transmitter, described in §2.

---

## 1. Headline

**The Q2 detector does not super-regenerate in simulation, and it does not
respond to an input signal at all.** Audio output is identical for a 10 µV input
and a 50 mV input — a 5,000:1 range — which means what comes out of the audio
take-off is the detector's own ripple, not the message.

This is the opposite of the transmitter result, where everything checked out. It
needs your attention before you spend build evenings on it.

---

## 2. Output

| Trace | Graph |
|---|---|
| `V(msg)` | the audio the far transmitter is sending (reference, 2 kHz) |
| `V(ant)` | the AM signal arriving at the antenna |
| `V(coll)` | Q2's collector — the detector's own oscillation |
| `V(aud)` | the R9/C11 node, where recovered audio should appear |
| `V(aout)` | into the volume pot |

Zoom to 800–1000 µs. `V(coll)` is a continuous 16 V pp sine. **What you will not
see is the thing `BUILD.md` §6.3 tells you to look for** — 27 MHz bursting on and
off at a 20–50 kHz quench rate. That envelope is the "it's alive" signature of a
super-regenerative detector, and this circuit never produces it.

The `.meas` block prints `quench_test`, the collector swing over a long window
divided by the swing over a short one. A super-regen bursts, so that ratio would
be large. **It measures 1.0001.** The detector never stops oscillating.

---

## 3. Tested before concluding this

Five independent attempts to find super-regenerative action, all negative:

| Hypothesis | What was swept | Result |
|---|---|---|
| Circuit as drawn | L2 from 0.90 to 1.30 µH | continuous, `quench_test` = 1.00 |
| Emitter bypass sets quench | C9 from 10 n to 470 n | continuous at every value |
| Base network sets quench | base-leak bias, C6 220 p → 10 n | continuous at every value |
| Regeneration set too hard | R7 from 8 Ω to 56 Ω | continuous at every value, 15.5–16.2 V pp |
| Detector is past threshold | supply from 1.2 V to 9 V | see below |

The supply sweep was the only one that showed anything:

| RX supply | Collector | quench ratio | Audio node |
|---|---|---|---|
| 1.2–2.0 V | not oscillating | — | — |
| **2.5 V** | 4.49 V pp | **1.42** | 0.302 V |
| **3.0 V** | 5.43 V pp | **1.41** | 0.382 V |
| 4.0 V | 7.21 V pp | 1.00 | 0.002 V |
| 9.0 V (as drawn) | 16.19 V pp | 1.00 | 0.017 V |

At 2.5–3 V the envelope does start to squeg. But re-testing sensitivity there was
also negative — audio out was **0.346 V for 10 µV in and 0.346 V for 10 mV in**.
The envelope varies by only ~40 %; the oscillation never fully extinguishes, so
there is no "restart seeded by the incoming signal", which is the entire mechanism
a super-regen gets its gain from.

---

## 4. Why — in one paragraph

A super-regenerative detector works by being **switched fully off and on** at the
quench rate. During each off period the oscillation dies to nothing; during each
restart, how fast it builds back up depends on what was present in the tank when
it restarted — and a few microvolts of incoming signal dominates that. That is
where the enormous gain comes from. This stage has nothing that switches it off.
It has the same bias network as the transmitter (33 k / 15 k, a stiff divider)
and 390 Ω of un-bypassed DC emitter degeneration, which together hold the
operating point rock steady. It oscillates, continuously and happily, and ignores
the antenna.

---

## 5. Reliability

**Be sceptical in one specific direction, but not dismissive.**

- SPICE has **no noise**. A real super-regen's sensitivity comes from noise-seeded
  restart, and the simulator cannot reproduce that. So the *sensitivity* figures
  above are not a fair prediction of the real circuit.
- But the **absence of quench is not a noise question.** Whether the oscillator
  switches off is large-signal, deterministic behaviour, which SPICE models
  correctly. No quench means no super-regeneration, noise or not.
- Supporting evidence from outside the simulation: `HANDOFF.md` §2 already records
  that this base circuit "is copied everywhere but rarely built successfully", that
  the one detailed build log managed 5 m indoors, and that experienced forum
  posters call it "will work but poor". **The simulation agrees with them**, and
  now says why.

**The bench is the arbiter.** Build the detector and look for the bursts.

---

## 6. Requirements under the deadline

`BUILD.md` §6 already building audio first, then the RX detector, then the
TX. Keep that order, it puts this question at **stage 6.3**, which is the earliest
you can find out, and leaves time to react.

**At stage 6.3, scope Q2's collector with the 10× probe:**

- **Bursting at 20–50 kHz** → the simulation was wrong about the real part, carry
  on exactly as planned. Tell me and I will re-model it.
- **A clean continuous 27 MHz sine** (what the simulation predicts) → the detector
  is regenerative, not super-regenerative. It will still demodulate AM, but with
  far less gain, and your range will be metres rather than tens of metres.

**If it is continuous, the fix is an explicit quench oscillator.** `HANDOFF.md` §2
says "resist adding an RF isolation stage or external quench" — that advice assumed
the stage self-quenches. It doesn't, so the premise is gone. An external quench is
one transistor, two resistors and two capacitors: a 30 kHz astable that chops Q2's
supply or emitter. It is the standard, documented cure and it is a couple of hours
of work, not a redesign.

**Do not** try to fix it by trimming R7, changing C9, or dropping the supply — all
three were swept above and none of them produce quenching.

Two other things to check at stage 6.3, both cheap:

- **L2 needs to be about 1.14 µH**, not the 1.10 µH the transmitter wants. The
  detector free-runs, so its tank *is* its tuning — at 1.10 µH it sits at
  27.63 MHz and at 1.20 µH at 26.51 MHz. Make L2 adjustable exactly as recommended
  for L1 in `PARTS_TOLERANCE.md`, and tune it to hear the transmitter.
- Confirm the RX really does run from the **full 9 V rail** via SW2 pole B, not
  from the LM386 output. That is what the netlist says and it is what I simulated.

---

## 7. Not yet simulated

- The audio chain (RV1 → LM386 → speaker).
- Any quench-oscillator variant — I have not designed or verified one; §6's
  description is the standard approach, not a tested circuit.
- TX-to-RX isolation on the same board (`BUILD.md` §5 puts the islands 50 mm apart
  for this reason).

---

*LTspice 26.0.2, 2026-09-21. Netlist derived from `out/wt27_v5.net`; node names
follow the schematic's net names.*
