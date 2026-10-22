# TX simulation — how to run it and what you should see

Files: `tx_am_27mhz.net` (the netlist), `tx_am_27mhz.plt` (optional plot layout).

---

## 1. Run it

Open `tx_am_27mhz.net` in LTspice and press **Run**, or from a shell:

```bash
"C:\Program Files\ADI\LTspice\LTspice.exe" -b -Run "C:\Users\name\KiCad\wt27_v5\sim\tx_am_27mhz.net"
```

Takes about **15 seconds** and writes a ~68 MB `.raw`. The `-b` form is batch —
no window, results go to `tx_am_27mhz.log`.

## 2. Add the traces

If `tx_am_27mhz.plt` is next to the netlist, LTspice loads the four panes
automatically. If it doesn't, build them by hand — **Plot Settings → Add Plot
Pane** three times, then right-click each pane → **Add Trace**:

| Pane | Trace | Graph |
|---|---|---|
| 1 | `V(msg)` | **the message** — the audio coming out of the LM386 |
| 2 | `V(carrier)` | **the carrier** — clean 27.145 MHz reference (see §5) |
| 3 | `V(coll)` and `V(txv)` | **the modulated signal** at Q1's collector, with the supply rail on top of it so you can see the envelope following the audio |
| 4 | `V(ant)` | **the transmitted signal** across the 50 Ω antenna load |

**Then zoom the time axis to 900 µs – 1000 µs.** The run is 1.1 ms long but
`t = 0…400 µs` is the oscillator starting up, and full scale shows a solid black
band because there are 30,000 RF cycles on screen.

## 3. Output

These come from the `.meas` block and are printed in `tx_am_27mhz.log` every run,
so you can tell at a glance whether a change broke something:

| Measurement | Value | Meaning |
|---|---|---|
| `fosc` | **27.1500 MHz** | carrier frequency — 5 kHz off the 27.145 target, 0.018 % |
| `pp_msg_peak` | 11.49 V | collector swing at the audio peak |
| `pp_msg_trough` | 3.86 V | collector swing at the audio trough |
| `mod_depth` | **0.497** | measured AM index vs the 0.5 asked for |
| `ant_pp` | 0.276 V | RF delivered into the 50 Ω antenna load |

The envelope ratio 11.49 / 3.86 = 2.98 against the theoretical
(1+m)/(1−m) = 3.00. Supply modulation of this oscillator is linear to about 1 %.

## 4. Knobs

All at the top of the netlist:

| Parameter | Default | Try |
|---|---|---|
| `fm` | 50 k | Message frequency. **1 kHz for a realistic audio tone** — also change `.tran` to `0 4m 0 2n` so you get three audio cycles, ~60 s run and a ~250 MB raw. |
| `m` | 0.5 | Modulation index. 0.8 shows the envelope nearly pinching off; **above ~0.85 the oscillator cuts off at the trough and you get over-modulation distortion**, which is worth showing in a report. `m=0` gives the unmodulated carrier. |
| `Ctrim` | 20 p | CV1. **12 p → 30 p moves the carrier about 7 kHz** — that is the netting range you'll use on the bench to match the two units. Below ~10 p the oscillator stops. |
| `Lp` | 1.10 µ | L1 primary. See §6. |
| `VTXDC` | 4.5 | The LM386 idles at Vs/2. Raise to 9 to see what the oscillator does on the raw battery rail — amplitude roughly doubles. |

## 5. Reasoning behind a fake carrier source

**This circuit has no unmodulated carrier node.** The oscillator *is* the
modulator: audio rides on Q1's supply rail, so every RF node in the transmitter
is already modulated. There is nowhere to probe a bare carrier.

`Vcar` is therefore an ideal 27.145 MHz source connected to nothing, present only
so pane 2 has a clean carrier to compare against. It is not part of the circuit
and carries no current. If you would rather show the real thing, set `m=0` and
re-run — pane 3 then shows the genuine unmodulated carrier the circuit produces.

## 6. Modelling choices you should know about before quoting these results

Four places where the simulation is deliberately not the physical circuit:

1. **The crystal is modelled "soft".** `Cmot = 20 fF`, `Rmot = 25 Ω` → Q ≈ 11,700.
   A real 27 MHz crystal is Q ≈ 25,000–80,000. A high-Q oscillator takes
   *hundreds of milliseconds* of simulated time to build up from numerical noise,
   which is not practical to run. Lower Q costs nothing here except start-up time
   and phase noise, neither of which this simulation is about.
2. **There is a start-up kick** — `Ikick`, one 20 ns 20 mA pulse into the tank at
   t = 1 µs. It just skips the build-up. Delete it and run to 5 ms to see genuine
   start-up from noise.
3. **`fpull = 13.4 kHz`.** The crystal model's *series* resonance is set 13.4 kHz
   above the wanted carrier so that the oscillator, with CV1 and the tank pulling
   it, lands on 27.145 MHz. This is normal practice — real crystals are specified
   for load resonance at a stated CL, not for bare series resonance.
4. **L1 is ideal** (no copper loss, no self-capacitance) and the antenna is a flat
   50 Ω. A real 1 m whip at 27 MHz is more like 1–2 Ω radiation resistance in
   series with −j1000, so it loads the tank *less* than 50 Ω does. The 50 Ω figure
   is the pessimistic case; `ant_pp` is not a real radiated-power prediction.

## 7. Two things the simulation told us about the actual design

**R3 = 56 Ω is load-bearing, and lowering it is wrong on both sides of the radio.**
`BUILD.md` used to say "no hiss / no bursts → drop R7 (56 R) a little to raise
the loop gain". **That advice has been removed**, because it is wrong twice over:
on the RX, sweeping R7 from 8 Ω to 56 Ω changes nothing that matters (see
`RX_SIM_NOTES.md` §3); and on the TX it is actively harmful. R3 sits directly
across C3, the Colpitts divider capacitor (Xc3 = 71 Ω at 27 MHz). Sweeping it:

| R3 | Collector swing, settled |
|---|---|
| 56 Ω (as drawn) | **6.8 V pp** |
| 39 Ω | 5.4 V pp |
| 27 Ω | 0.6 V pp, still creeping up |
| 15 Ω | dead |

Lowering R3 shorts out the emitter tap and stops the crystal oscillator. **Do not
"improve" the TX by dropping R3** — the 56 Ω in the schematic is the right value,
and if you need to adjust loop gain on the transmitter, move `Lp` instead.

**The tank wants to be 1.0–1.15 µH, tuned slightly high of the crystal.**
The simulation says L1's primary should be about **1.10 µH** — note this is *not*
the same as L2 on the receiver, which wants 1.14 µH (`BOM.md` §6):

| Lp | Collector swing | fosc |
|---|---|---|
| 1.00 µH | 7.9 V pp | 27.129 MHz |
| 1.15 µH | 6.8 V pp | 27.114 MHz |
| 1.32 µH | 2.9 V pp | 27.104 MHz |

Note how little the frequency moves — **a 32 % change in tank inductance shifts
the carrier by 0.09 %.** That is the proof it is crystal-controlled and not just
an LC oscillator; a free-running LC would have moved about 15 %. It also means you
cannot tune this thing with the coil — the coil sets *amplitude*, CV1 sets
*frequency*.

---

*Verified against LTspice 26.0.2 on 2026-09-20. Netlist derived from
`out/wt27_v5.net`; node names match the schematic's net names where they exist.*
