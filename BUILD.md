# 27 MHz Walkie-Talkie — Build and Debug Manual

Two identical units on veroboard. Rewritten 2026-09-21 as a step-by-step manual.

| You need… | Look in |
|---|---|
| The circuit | `out/wt27_WIRED.pdf` (full wiring) and `out/wt27_BLOCKS.pdf` (grouped by block) |
| What to buy | `BOM_offline_nawabpur.pdf` (market) or `BOM.csv` (online) |
| Which parts must be accurate | `PARTS_TOLERANCE.md` |
| Why the receiver needs the quench add-on | `sim/RX_SIM_NOTES.md`, `QUENCH_MOD.md` |

Figures marked **(sim)** come from the LTspice simulations in `sim/`. They are what
a correct circuit should do, not guarantees about your particular parts.

---

## Contents

1. How to use this manual
2. Understand the circuit first (10 minutes, saves hours)
3. Tools and bench setup
4. Before you solder: check every part
5. Board layout, strip cuts and RF rules
6. **Build stages** — each one with steps, a test, expected readings, and "if it doesn't work"
   - 6.0 Prepare the board · 6.1 Power + audio · 6.2 Crystal test jig ·
     6.3 RX detector · 6.4 Quench add-on · 6.5 Transmitter ·
     6.6 PTT switch · 6.7 Second unit · 6.8 Tune the pair · 6.9 Antennas and range test
7. Winding the coils
8. Antennas
9. Measurement techniques
10. **Troubleshooting** — symptom → cause → fix
11. Record sheet

---

## 1. How to use this manual

**Build in stages and test each one before starting the next.** Every stage can be
tested with only what you have already built, so when something fails you know it
is in the part you just added. This is the single most important habit here: if you
solder the whole radio and then switch it on, a fault could be anywhere.

Three rules that apply to every stage:

1. **Measure DC voltages before looking at RF.** A wrong DC voltage explains most
   faults, and a multimeter is quicker than a scope.
2. **Change one thing at a time**, then re-test.
3. **Write your readings in the record sheet (§11).** When unit B misbehaves, comparing
   it against unit A's numbers finds the fault in minutes.

Label the two boards **A** and **B**, and keep their parts in separate bags.

---

## 2. Understand the circuit first

The radio has **four blocks** and **one switch** that rewires them between listening
and talking.

```
                         ┌──────────── PTT switch (4 poles) ────────────┐
                         │                                               │
  RECEIVE (PTT released) │  antenna → L2 → Q2 detector → audio → C12 ──┐ │
                         │                                              ▼ │
                         │                      volume RV1 → LM386 → speaker
                         │                                                │
  TRANSMIT (PTT pressed) │  mic → C13 → volume RV1 → LM386 ──┐            │
                         │                                   ▼            │
                         │        LM386 output = Q1's supply → Q1 + crystal → L1 → antenna
                         └────────────────────────────────────────────────┘
```

| Block | Parts | What it does |
|---|---|---|
| **Power** | BT1, SW1, C16, C19, C20 | 9 V battery, on/off switch, supply smoothing |
| **Audio** | U1 (LM386), RV1, C14, C15, C17, C18, R11 | Amplifies either the received audio (to the speaker) or the mic (to the transmitter). Gain 200. RV1 is the volume on receive **and the mic level on transmit**. |
| **Transmitter** | Q1, Y1 (crystal), CV1, L1, R1–R4, C1–C5 | Crystal oscillator at 27 MHz. **Its supply comes from the LM386 output**, so your voice varies the supply and therefore the RF amplitude. That *is* the AM modulation — there is no separate modulator. |
| **Receiver** | Q2, L2, R5–R9, C6–C11 | Super-regenerative detector: an oscillator switched on and off 32,000 times a second. How quickly it restarts depends on the incoming signal, and that becomes audio at R9/C11. **Needs the quench add-on (§6.4).** |
| **Mic** | MK1, R10, C13 | Electret microphone, powered through R10 |

** PTT switch  (4 poles):**

| Pole | Common pin connects to | Released = RECEIVE | Pressed = TRANSMIT |
|---|---|---|---|
| SW2-A | Antenna | → L2 (receiver) | → L1 (transmitter) |
| SW2-B | +9 V | → receiver supply | → nothing (receiver off) |
| SW3-A | Volume pot input | ← receiver audio (C12) | ← microphone (C13) |
| SW3-B | LM386 output | → speaker (C17) | → transmitter supply (L1 pin 1, R1) |


---

## 3. Tools and bench setup

| Tool | Used for |
|---|---|
| Soldering iron, 25–40 W, fine tip, and solder | Everything |
| Multimeter (continuity, volts, **mA**) | Strip cuts, DC checks, current draw |
| Component tester | Transistor type and pinout, capacitor values |
| Oscilloscope **with a 10× probe** | RF and audio waveforms |
| Frequency counter | Crystal and transmitter frequency |
| 3–4 mm drill bit (twisted by hand) | Cutting strips |
| Side cutters, small pliers, blade or fine sandpaper | Leads, enamel stripping |
| Magnifier and a torch | Inspecting for solder bridges |
| **Plastic** trimming tool (or a sharpened plastic pen cap) | Adjusting CV1 and coils — a metal screwdriver changes the tuning while it touches |
| Hot glue, sticky tape, candle wax | Fixing coils |

**20 MHz scope reads 27 MHz signals about 40 % low.** Frequency readings are
fine; amplitude readings are not. All "on your scope" figures below allow for this.



---

## 4. Before solder: check every part



| Part | Check | Reject if |
|---|---|---|
| **Every BC547** | Component tester: type NPN, hFE, **and which leg is which** | hFE under 150, or the tester can't identify it |
| Crystals | Count the pins | **4 pins** — that's an oscillator module and won't work here. You need a **2-pin** crystal |
| C2/C7 (33 pF) | Capacitance | More than +10 % high — **pick ones that read low**. This is the one capacitor that can stop the transmitter |
| Other small ceramics | Capacitance | More than ±20 % off |
| Electrolytics | Capacitance | More than 40 % below the marking |
| Resistors | Resistance | More than ±20 % off (they almost never are) |
| Speakers | Resistance | Not roughly 6–8 Ω |
| Electret mics | Tester or meter | Open circuit both ways |

Write each transistor's pinout on a strip of tape next to it. **TO-92 pinouts vary
between manufacturers** — commonly C-B-E left to right with the flat face towards
you, but trust the tester, not the datasheet.

### Which way round

| Part | How to tell |
|---|---|
| Electrolytic capacitor | Stripe = negative. Longer leg = positive |
| **C12 and C13** (1 µF / 0.47 µF, if electrolytic) | **C13: + towards the microphone.** **C12: + towards R9/C11.** The schematic draws them non-polar, but the parts you buy will usually be electrolytic |
| C14 | + to LM386 pin 1 |
| C15 | + to LM386 pin 7, − to ground |
| C16, C20 | + to +9 V |
| C17 | + to the switch/LM386 side, − to the speaker |
| 1N4148 diode | Black band = cathode |
| LM386, NE555 | Notch or dot = pin 1. Viewed from the top, pins count **anticlockwise** |
| Electret mic | The leg connected to the metal case is **ground** |
| CV1 trimmer | Find the leg connected to the adjusting screw (rotor) with a meter — **that leg goes to the ground side (the eb node)**, so your tool doesn't detune it |
| Crystal, ceramic caps, resistors | No polarity |

```
   LM386 (top view)              NE555 (top view)
   ┌───╮╭───┐                    ┌───╮╭───┐
 1 │GAIN  GAIN│ 8              1 │GND    VCC│ 8
 2 │-IN  BYPAS│ 7              2 │TRIG  DISC│ 7
 3 │+IN    +Vs│ 6              3 │OUT   THRS│ 6
 4 │GND    OUT│ 5              4 │RST   CTRL│ 5
   └──────────┘                  └──────────┘
```

---

## 5. Board layout, strip cuts and RF rules

### Floorplan

One board per unit, strips running **left–right**, about 100 × 60 mm or bigger
(the 14.5 × 6.5 cm board is fine). Don't try to make it small — spacing is what
makes RF work on veroboard.

```
        col 1 ....... 12 | 13 .. 19 | 20 ....... 31 | 32 ... 38
      +-----------------+----------+---------------+----------+
row 1 |                 |          |               |          |
  .   |   TX RF ISLAND  |  EMPTY   | RX RF ISLAND  |  POWER   |
  .   |   Q1 L1 Y1 CV1  |  GUARD   | Q2 L2  (+555) |  SW1 BT1 |
row 8 |   R1-R4 C1-C5   |   GAP    | R5-R8 C6-C10  |  C19 C20 |
      +-----------------+----------+---------------+----------+
row 9 |>>>>>>>>>>  G R O U N D   B U S  (3 strips)  <<<<<<<<<<|
row11 +---------------------------------------------------------+
  .   |   AUDIO:  RV1 -> U1 (LM386) -> C14 C15 C16 C17 R11 C18  |
row18 +---------------------------------------------------------+
row19 |   TAG STRIP: flying leads to PTT, ANT, SPKR, MIC, BATT   |
row24 +---------------------------------------------------------+
```

Why it looks like this:
1. **TX and RX islands ~50 mm apart** with an empty gap — a receiver next to a
   transmitter oscillator gets swamped by it.
2. **A solid ground bus** across the middle — ground quality decides whether RF
   stages behave.
3. **Audio as far from RF as possible** — the LM386 has a gain of 200; its output
   near the detector causes howling.

Put the 555 quench circuit (§6.4) at the RX end of the board, near the power section,
**not** inside the TX island.

### Strip cuts

Cut with a 3–4 mm drill bit twisted by hand until the copper ring is gone.

| Where | Cuts | Why |
|---|---|---|
| Under U1 (LM386) and the 555 | One between each pair of opposite pins (4 per IC) | Otherwise pin 1 shorts to pin 8, 2 to 7, and so on |
| Either side of every RF node | 1 each side | Keeps RF nodes short (see rules below) |
| Between the 3 ground-bus strips | None — **link them together** with wire at several points | Makes one solid ground |
| Under Q1 / Q2 | See below | |

**Transistors:** the three legs are 2.54 mm apart, so if you push them into one
strip, C, B and E are shorted together. **Bend the legs out so each one lands on a
different strip.**

**Check every cut with the meter on continuity** before fitting parts. A cut that
didn't quite go through is the most common stripboard fault by far.

### RF rules (these decide whether it works)

At 27 MHz a long copper strip behaves like a small antenna and a capacitor.

1. **Every RF node uses 3–5 holes of strip at most**, with a cut either side. This
   applies to the collector, base and emitter of Q1 and Q2.
2. **C4 and C9 (10 nF) sit right at the emitter end of R3/R7**, within two holes.
   They must be a short circuit at 27 MHz; a long lead defeats them.
3. **C2/C3 and C7/C8 get the shortest leads you can manage** — a few millimetres.
4. **L1 and L2: four ends on four separate strips**, fitted **last** so you can
   adjust them. Mount L1 and L2 **at right angles** to each other.
5. **Crystal and CV1 right next to Q1.**
6. **Each RF stage gets its own short link to the ground bus.** Don't run the TX
   ground through the RX ground.

---

## 6. Build stages

Each stage lists: **parts → steps → test → expected → if it doesn't work.**

### Testing before the PTT switch is fitted — temporary jumpers

Until §6.6 you have no PTT switch. Use short wires on the tag strip to make the
connections the switch would make:

| To test… | Connect (RECEIVE jumpers) | Connect (TRANSMIT jumpers) |
|---|---|---|
| Antenna | antenna → L2 pin 4 | antenna → L1 pin 4 |
| Receiver supply | +9 V → L2 pin 1 / R5 (later → 555 supply) | leave open |
| Volume pot input (RV1 pin 1) | ← C12 | ← C13 |
| LM386 output (pin 5) | → C17 → speaker | → L1 pin 1 / R1 |

Never have both sets connected at once.

### First power-up of any stage

1. **Before connecting the battery**, measure resistance from +9 V to ground. It
   should start low and climb as the capacitors charge, ending well above 1 kΩ.
   A steady reading under ~100 Ω means a short — find it before powering.
2. **Put the multimeter in series with the battery on its 200 mA range.**
3. Switch on and read the current. Guide figures:

| Built so far | Expected current |
|---|---|
| Power + audio | ~5–10 mA |
| + receiver (RX jumpers) | ~10–20 mA (up to ~25 mA with the 555) |
| + transmitter (TX jumpers) | ~7–12 mA, more while you speak |

**Over 50 mA: switch off at once.** Something is shorted, backwards, or oscillating
wildly. See §10.

---

### 6.0 Prepare the board (≈ 45 min)

1. Mark the floorplan on the component side with a pencil.
2. Make all the strip cuts from §5.
3. Link the three ground-bus strips together with bare wire or resistor-leg offcuts,
   at both ends and in the middle.
4. **Test:** continuity across every cut (should be open) and along the ground bus
   (should be 0 Ω end to end).
5. Hold the board up to a torch and look through the cuts — any copper whisker
   left will show.

### 6.1 Power supply and audio amplifier (≈ 1.5 h)

**Parts:** battery clip, SW1, C16, C19, C20, U1 (LM386 in a socket), C14, C15, C17,
C18, R11, RV1, speaker (flying leads).

**Steps:**
1. Fit the 8-pin socket (not the chip yet) over its 4 cuts.
2. Fit SW1 in the battery + lead, then C16, C19, C20 close to the socket.
3. Fit C14 (pins 1–8), C15 (pin 7 to ground), R11 + C18 in series from pin 5 to
   ground, and C17 from pin 5 towards the speaker.
4. Wire RV1: top end = input (pin 1), bottom end = ground (pin 3), wiper → LM386 pin 3.
5. Ground LM386 pins 2 and 4.
6. Make the RECEIVE jumper: LM386 pin 5 → C17 → speaker → ground.
7. **Do the resistance check, then insert the LM386**, then power up with the meter
   in series.

**Tests:**
| Where | Expected |
|---|---|
| LM386 pin 6 | ≈ 9 V (battery voltage) |
| **LM386 pin 5 (output)** | **≈ 4.5 V — about half the supply.** This is the key check |
| Pins 2, 3, 4 | ≈ 0 V |
| Current | ~5–10 mA |

**Sound test (no signal generator needed):** turn RV1 to maximum and touch the RV1
input pin with a finger. You should hear a **loud hum or buzz** — that's mains hum
picked up by your body. No hum = no audio path.

Better test: play music on a phone, headphone-socket output → a 1 µF capacitor →
RV1 input, phone ground → circuit ground. Music should play clearly, getting louder
with RV1.

**Mic test:** fit R10 and the mic, and temporarily jumper C13 → RV1 input. Tap the
mic and you'll hear it. **With the speaker near the mic it will howl** — that's
normal acoustic feedback at gain 200, not a fault; point the speaker away or turn
RV1 down. (In the finished radio the speaker is disconnected while you talk, so this
can't happen then.)

**If it doesn't work:** §10.2 (audio).

### 6.2 Crystal test jig — optional but recommended (≈ 30 min)

A six-part oscillator on a scrap of board. It answers **"is my crystal really
27 MHz fundamental?"** before you depend on it, and later it's a **27 MHz signal
source for tuning the receiver**.

```
   +9V ──┬─────────────┬
         │             │ collector
        R1 33k      [BC547]
         ├─────────────┤ base
         │             │ emitter
        R2 15k         ├──────┬──── 100 pF ──── output (counter, or a 30 cm wire)
         │            30 pF   1 kΩ
    [CRYSTAL]          │      │
         │            30 pF   │
        GND            │      │
                      GND    GND
     (first 30 pF: base → emitter; second 30 pF: emitter → ground)
```

**Test:** counter on the output, 1 s gate.

| Reading | Meaning |
|---|---|
| **~26.95 MHz** | Correct. The jig reads ~47 kHz low (sim) because of its own capacitors — that's expected, not a fault |
| ~9 MHz | **3rd-overtone crystal.** The real transmitter's tuned tank may still pull it up to 27 MHz, but it's a warning — see §10.3 |
| Nothing | Check the transistor legs and the 9 V supply |

**Measure both crystals** in this jig and write down both readings. If they differ
by more than ~5 kHz, note it for §6.8.

### 6.3 Receiver detector (≈ 1.5 h) — the decision stage

**Read `sim/RX_SIM_NOTES.md` first.** The simulation predicts this stage won't work
as a super-regenerative receiver on its own. This stage finds out for real.

**Parts:** Q2, L2 (wind it per §7 — 10 turns spread over ~4.5 mm), R5–R9, C6–C11,
C12. **C6 = 33 nF** (the schematic says 39 nF, which you won't find — and 33 nF
simulates better). **Never use 47 nF or larger for C6**: with the quench fitted the
receiver goes deaf (sim: 1.44× response at 33 nF, 1.06× at 47 nF, none at 100 nF).

**Steps:**
1. Fit Q2 with its legs on three separate strips.
2. Fit R5/R6 (base bias), R7 + C9 at the emitter (C9 right there), R8, C6.
3. Fit C7/C8 with the shortest leads you can manage.
4. Fit R9, C11 (audio take-off) and C12 (+ towards R9/C11).
5. Fit L2 last: pin 2 → Q2 collector + C7, pin 1 → supply + R5, pin 3 → C10 → ground,
   pin 4 → antenna tag. Hot-glue the former down.
6. RECEIVE jumpers: +9 V → L2 pin 1, C12 → RV1 input, LM386 → speaker.

**Test A — DC (meter):**
| Where | Expected |
|---|---|
| Q2 base | ≈ 2.7–2.9 V |
| Q2 emitter | ≈ 2.0–2.6 V (shifts while oscillating) |
| Current | ~10–20 mA total |

**Test B — scope Q2 collector with the 10× probe, 10 µs/div then 50 ns/div:**

| What you see | What it means | Next |
|---|---|---|
| **A continuous 27 MHz sine**, steady amplitude (sim: 16 V p-p true, ~10 V on your scope) | **As predicted.** The detector oscillates non-stop and won't receive properly | **Go to §6.4 and fit the quench add-on** |
| **Bursts** — the 27 MHz switching on and off 20,000–50,000 times a second | It self-quenches after all — the real parts beat the simulation | Skip §6.4. Tell me: the simulation needs redoing |
| Flat line | Not oscillating | §10.4 |

**Test C — listen,** RV1 at maximum:
- **Loud rushing hiss** → super-regenerating. Good.
- **Near silence, maybe a faint whistle** → continuous oscillation, as predicted. Fit §6.4.

**Do NOT try to fix a non-bursting detector by changing R7, C9 or the supply
voltage.** All three were swept in simulation (R7 8–56 Ω, C9 10–470 nF, supply
1.2–9 V) and none of them produce quenching. R7 stays at 56 Ω.

### 6.4 Quench add-on — if §6.3 showed a continuous sine (≈ 1 h)

Full design and evidence: `QUENCH_MOD.md`. The 555 switches the detector's supply
fully on and off ~32,000 times a second, which is what makes it super-regenerative.

**Parts:** NE555 + socket, R1 = R2 = 33 kΩ, 1N4148, C1 = 680 pF (or 560 pF + 120 pF
in parallel), 10 nF, optionally 100 nF.

**Wiring:**

| 555 pin | Connects to |
|---|---|
| 8 (VCC) and 4 (RESET) | **The receiver supply — SW2-B pin 6** (the RX jumper for now). **Not** the always-on +9 V: this way the 555 and detector switch off completely while you transmit |
| 1 (GND) | Ground |
| 7 (DISCH) | Junction of R1 and R2. R1's other end → pin 8 |
| 6 (THRS) and 2 (TRIG) | Joined together; R2's other end; C1 → ground |
| **1N4148** | **Across R2: band (cathode) towards pins 6/2, other end towards pin 7.** This sets 50 % duty — without it the receiver is nearly deaf (sim: 1.04× vs 1.90× response) |
| 5 (CTRL) | 10 nF → ground |
| **3 (OUT)** | **L2 pin 1 and R5** — in place of the old +9 V jumper |
| (optional) | 100 nF from pin 8 to pin 1, right at the chip |

**Tests:**
| Where | Expected |
|---|---|
| 555 pin 8 | ≈ 9 V (on receive) |
| 555 pin 3, meter | ≈ 3.5 V (it averages a square wave) |
| **555 pin 3, scope** | **Square wave, ~0.3 V to ~7 V, period ~31 µs (32 kHz), high and low roughly equal** |
| Q2 collector, scope, 10 µs/div | **Bursts of RF** every ~31 µs |
| Speaker, RV1 at max | **Loud rushing hiss** |

**The receive test:** switch on the crystal jig (§6.2) with a 30 cm wire on its
output, 1–2 m from the receiver's antenna wire. **The hiss should go noticeably
quieter** — a super-regenerative receiver's hiss "quietens" when a carrier arrives.
Move the jig closer and further: the quietening should follow.

**Tuning L2:** with the jig on, squeeze or spread L2's turns for the **biggest
quietening** of the hiss. That puts the receiver on 27 MHz. Fix with wax.

**Tuning the quench:** try C1 = 470 pF (46 kHz), 680 pF (32 kHz), 1 nF (22 kHz).
Keep whichever gives the strongest hiss and the clearest reception. Keep R1 = R2
whichever you choose.

**If it doesn't work:** §10.5.

### 6.5 Transmitter (≈ 2 h)

**Parts:** Q1, crystal Y1, CV1, L1 (wind per §7 — 10 turns spread over ~5 mm),
R1–R4, C1–C5.

**Steps:**
1. Fit Q1 legs on three strips.
2. Fit R1/R2 (base bias), R3 + C4 at the emitter (C4 right there), R4.
3. Fit C2 (collector–emitter) and C3 (emitter–eb node) — shortest leads. **Use a C2
   that measured low.**
4. Fit the crystal branch: base → C1 → crystal → CV1 → eb node, all tight around Q1.
   CV1's rotor leg to the eb node. **Set CV1 to mid-travel.**
5. Fit C5 from L1 pin 3 to ground.
6. **Fit L1 last:** pin 2 → collector + C2, pin 1 → R1 + supply, pin 3 → C5,
   pin 4 → antenna tag. Hot-glue it.
7. **Dummy load:** a 56 Ω resistor from L1 pin 4 to ground (remove it before fitting
   an antenna).
8. Supply: for the very first test connect **L1 pin 1 straight to +9 V**. Once it
   works, change to the TRANSMIT jumpers (LM386 pin 5 → L1 pin 1).

**Test A — DC:**
| Where | Supplied from LM386 (4.5 V) | Supplied from +9 V direct |
|---|---|---|
| Q1 base | ≈ 1.34 V (sim) | ≈ 2.75 V |
| Q1 emitter | ≈ 0.69 V (sim) | ≈ 2.1 V |
| Q1 collector | ≈ 4.5 V | ≈ 9 V |

**Test B — scope, 10× probe:**
| Point | Sim (4.5 V supply) | On your 20 MHz scope | At 9 V direct |
|---|---|---|---|
| Across the 56 Ω dummy | 0.20 V p-p | ~0.12 V p-p | roughly double |
| Q1 collector | 7.6 V p-p | ~4.5 V p-p | roughly double |

**Test C — tune L1:** push the turns closer (more inductance) or spread them (less)
while watching the dummy-load amplitude. Stop at the **peak**. **If it won't start at
all, spread the turns or remove one** — in simulation the transmitter only ever
failed with *too much* inductance.

**Test D — frequency:** counter coupled loosely (§9.2). Expect about **27.0 MHz**.

**Test E — is it really the crystal?** Squeeze L1 hard, then spread it: the
frequency should barely move (sim: a 32 % change in L1 moved it 0.09 %). **If the
frequency follows the coil by hundreds of kHz, the crystal isn't in control** — see
§10.3.

**Test F — modulation:** change to TRANSMIT jumpers (mic → C13 → RV1, LM386 → L1
pin 1). Whistle or speak into the mic while watching the dummy load at 0.2 ms/div:
the RF band should **swell and shrink with your voice**. RV1 sets the depth — if the
band pinches to nothing on loud sounds, turn RV1 down (that's over-modulation).

Fix L1 with wax, re-check the peak after it cools.

**If it doesn't work:** §10.3.

### 6.6 The PTT switch (≈ 1 h)

**Identify the pins first.** A 4PDT switch has 12 pins: 4 poles of 3. With the meter
on continuity:
1. **Switch released:** find the 4 pairs of pins that are connected.
2. **Switch pressed:** find the 4 pairs connected now.
3. In each pole, the pin that appears in *both* pairs is the **common**. The pin it
   connects to when **released** is the **RECEIVE** side; when **pressed**, the
   **TRANSMIT** side.
4. Draw it on paper and label all 12 pins before soldering.

**Wire it per the table in §2.** Use **stranded hookup wire**, and colours: red for
+9 V, black for ground, a different colour for each of the others. Keep the antenna
pole's wires shortest.

**Remove all the temporary jumpers**, then test:
| Switch | Expected |
|---|---|
| Released | Hiss from the speaker (receiver working); transmitter silent |
| Pressed | No hiss; RF on the dummy load; your voice modulates it |

If you could only buy a **latching** (push-on/push-off) 4PDT, it works the same —
press to talk, press again to listen. A 4PDT slide switch also works.

**If it doesn't work:** §10.6.

### 6.7 Second unit (≈ 3–4 h)

Build unit B exactly the same way, stage by stage. It's much quicker now.

**Debug by comparison:** measure the same DC points and fill in the record sheet.
Any reading that differs noticeably from unit A points straight at the fault.

### 6.8 Tune the pair (≈ 45 min)

1. **Measure both transmitters** on the counter (§9.2). Same-batch crystals should
   be within ~1 kHz. If they differ by more than ~5 kHz, move them together with
   CV1 (full range ≈ 7 kHz, sim). Keep CV1 above roughly one-third of its travel —
   **below ~10 pF the oscillator stops** (sim).
2. **Tune each receiver onto the other unit's transmitter.** Unit B transmits
   (dummy load removed, short wire antenna), unit A receives: adjust A's L2 for the
   loudest, clearest audio. Then swap.

A super-regenerative receiver listens across a wide slice of the band (hundreds of
kHz), so **receiver tuning (L2) matters much more than matching the two
transmitters to the last kHz.**

### 6.9 Antennas and range test

1. **Wire antennas first** (§8.1): 2.7 m radiator plus 2.7 m counterpoise on each unit.
2. Fresh batteries. Start 2 m apart; talk both ways.
3. Double the distance each time until the audio breaks up. Test **both directions**.
   Write down the distances.
4. **Then the whips** (§8.2). With unit B transmitting, remove turns from unit A's
   loading coil one at a time for the loudest audio. Then do unit B's.
5. Re-test range with whips. **Expect it to be shorter than with the wire** — that's
   normal for a short antenna.

**Plan the demo around what you measure,** not what you hoped for.

---

## 7. Winding the coils

### Wire

At 27 MHz current only flows in the outer 12.7 µm of the wire, so **0.3 mm and
0.5 mm enamelled wire both work**. Thinner wire just has a little more loss, which
barely matters for L1/L2. For the antenna loading coil, thicker is better (§8.2).

0.3 mm won't hold its shape — always wind on a **plastic or paper former**
(pen barrel, rolled and glued paper tube). **Never leave metal inside a coil.**

### L1 and L2 — the pin map

| End | L1 (transmitter) | L2 (receiver) |
|---|---|---|
| pin 2 — primary start, **hot end** | Q1 collector + C2 | Q2 collector + C7 |
| pin 1 — primary finish, cold end | R1 + LM386 via SW3-B | R5 + RX supply (555 pin 3) |
| pin 3 — secondary | C5 → ground | C10 → ground |
| pin 4 — secondary | antenna via SW2-A | antenna via SW2-A |

### Turns for your former

Target ~1.10 µH (L1) / ~1.14 µH (L2). **Measure your former's diameter:**

| Former Ø | 0.3 mm wire | 0.5 mm wire |
|---|---|---|
| 8 mm | 11 turns | 13 turns |
| 9 mm | 10 turns | 11 turns |
| **10 mm** | **9 turns** | 10 turns |
| 11 mm | 8 turns | 9 turns |
| 12 mm | 8 turns | 9 turns |
| 13 mm | 7 turns | 8 turns |

(Wheeler's formula, ±10 % — close enough, because every coil is tuned on the bench.)

### Winding step by step

About 45 cm of 0.3 mm wire per coil.

1. Tape the start to the former with a **3 cm tail**. Tag it **"C"** (pin 2).
2. Wind **one turn more than the table** (e.g. 10 on a 10 mm former), tight and in
   one direction. Then **spread the turns**: ~5 mm long for L1, ~4.5 mm for L2.
3. Tape the finish, 3 cm tail, tag **"+"** (pin 1).
4. One layer of tape over the **"+" end**.
5. **2 turns on top of that tape at the "+" end**, same direction. Tag the tails
   **"G"** (pin 3) and **"A"** (pin 4).
6. Scrape 3–4 mm of enamel off each end until bright copper shows, and tin it. (Some
   enamel melts off with a hot iron and a blob of solder; if the solder won't stick,
   scrape more.)
7. **Meter check:** C↔+ ≈ 0 Ω, G↔A ≈ 0 Ω, and **between the windings: open**. If it
   beeps between windings, the enamel is damaged — rewind.

Your component tester can't usefully measure ~1 µH, so **coils are tuned by their
effect on the circuit**. Squeezing turns together = more inductance; spreading =
less. Fix with candle wax when done.

---

## 8. Antennas

All figures for 27.000 MHz (wavelength 11.10 m).

### 8.1 Quarter-wave wire — for testing

- **Cut 2.80 m, trim to about 2.70 m.** Being a few cm out costs very little.
- **Ordinary insulated stranded wire, 0.5–1 mm²** — not enamelled (snaps), not coax
  (there's no feed line).
- Run it **straight and up**, not coiled on the bench. Attach with a crocodile clip.
- **Add a counterpoise:** another 2.7 m wire from circuit ground, laid along the
  floor in the opposite direction. Without one, your hand becomes half the antenna.
- This is the reference: it proves the radio works before the whip's losses.

### 8.2 Telescopic whip + loading coil — for the demo

Buy a whip **at least 1.0 m extended; 1.2 m or longer is better.** Avoid the
0.6–0.8 m kind from cheap radios.

A short whip needs a coil in series at its base:

| Whip length | Loading coil | Radiation resistance |
|---|---|---|
| 0.75 m | 4.1 µH | ~1.8 Ω |
| **1.0 m** | **3.1 µH** | ~3.2 Ω |
| 1.2 m | 2.5 µH | ~4.6 Ω |
| 1.5 m | 1.8 µH | ~7.2 Ω |

For a 1 m whip on a 10 mm former: **~18 turns of 0.3 mm or ~22 turns of 0.5 mm**,
close-wound. **Wind 4–5 extra turns and remove them one at a time** for the loudest
reception (§6.9) — without an SWR meter, that's the tuning method.

Wire gauge matters most here: the coil's loss sits in series with only ~3 Ω of
radiation resistance. Rough estimate for a 1 m whip: 0.3 mm ≈ 27 % efficient,
0.5 mm ≈ 38 %, 1 mm ≈ 48 % (coil Q is an assumption). **Expect the whip to be
several dB worse than the wire.** That's normal.

---

## 9. Measurement techniques

### 9.1 Probe discipline

- **10× probe only on RF.** A 1× probe adds ~100 pF and stops a 27 MHz oscillator
  dead — you'll think you broke it.
- Clip the probe's ground lead to the ground bus right next to the point you're
  measuring.
- A 20 MHz scope reads 27 MHz **amplitude ~40 % low**. Frequency is fine.
- Touching a node with your finger, the probe or a metal tool changes RF behaviour.
  If something only works while you're touching it, see §10.7.

### 9.2 Reading frequency on the counter

A counter's input (~1 MΩ ‖ 20–30 pF plus cable) **will pull or stop an oscillator**
if connected to its collector. Couple loosely:
1. **Best:** from L1's secondary (pin 4, across the dummy load).
2. **Or:** 10–20 cm of wire on the counter input held a few cm from the coil. Bring
   it closer only until the reading is steady.
3. **Or:** the 10× probe on the collector and read frequency off the scope.

Set CV1 mid-range, let it run a minute, use a **1 s gate**.

### 9.3 Reading the crystal

A crystal on its own can't be measured — it has to oscillate. Use the jig in §6.2
(quick, and it can't run on an overtone) or the finished transmitter (true on-air
frequency).

| Jig capacitors | Jig reads, for an exactly 27.000 MHz crystal (sim) |
|---|---|
| **30 pF** | 26.953 MHz (−47 kHz), 2.3 V p-p output |
| 47 pF | 26.941 MHz (−59 kHz) |
| 100 pF | 26.933 MHz (−67 kHz) |

---

## 10. Troubleshooting

**How to debug anything:**
1. **Is it powered?** Battery voltage under load, then the supply pin of the stage.
2. **Are the DC voltages right?** Compare with the stage's table. A wrong DC voltage
   locates most faults on its own.
3. **Look at the board** with a torch and magnifier: solder bridges between strips,
   uncut strips, parts one hole out, dull or cracked joints.
4. **Swap the suspect part** — especially transistors.
5. **Compare with the working unit.**

### 10.1 Power and general

| Symptom | Likely cause | Check / fix |
|---|---|---|
| Nothing at all | Flat battery; clip reversed; SW1 wiring; cracked strip | Battery under load should be > 7.5 V. Meter the supply along the strip |
| **Current over 50 mA** | Short +9 V to ground; part backwards; LM386 oscillating | Switch off. Resistance +9 V→ground; check electrolytic and IC orientation; check C18/R11 are fitted |
| A part gets hot | Reversed transistor or IC; solder bridge | Switch off; check orientation and look for bridges near that part |
| Battery goes flat in hours | Something drawing current while "off"; oscillating stage | Measure current with SW1 off (should be 0) |
| Works, then stops after a few minutes | Battery voltage sagging; a dry joint warming up | Measure battery under load; reflow suspect joints |

### 10.2 Audio

| Symptom | Likely cause | Check / fix |
|---|---|---|
| Silent, pin 5 not ~4.5 V | LM386 in backwards; pin 6 not powered; pin 4 not grounded; bridge under the chip | Check the 4 cuts under the socket; check orientation |
| Silent, pin 5 is ~4.5 V | C17 open or backwards; speaker lead broken; SW3-B wiring; RV1 at zero | Follow the signal: finger on RV1 input should hum |
| Loud howl with mic connected (in §6.1 test) | Acoustic feedback — normal at gain 200 | Point the speaker away, turn RV1 down |
| **Howl or squeal in the finished radio** | PTT wiring leaves mic and speaker both connected; mic leads next to speaker leads | Recheck SW3 against §2; route mic and speaker leads on opposite sides; twist the mic pair |
| **"Putt-putt" motorboating** | Weak battery; C16/C20 missing or under-value | New battery; check the electrolytics (market ones often read far below marking) |
| Loud hum | Open or broken wire at RV1 input; ground lead missing | Reflow the RV1 connections and the ground to RV1 pin 3 |
| Distorted, rough audio | RV1 too high; battery low | Turn down; replace battery |
| Mic dead | Mic backwards (case leg must be ground); R10 missing; C13 backwards | Mic + should read ~4–8 V: 9 V = mic open, ~0 V = mic shorted or reversed |

### 10.3 Transmitter

| Symptom | Likely cause | Check / fix |
|---|---|---|
| **No oscillation, DC voltages correct** | L1 too much inductance; C2 too high; CV1 near minimum; 1× probe | Spread L1's turns or remove one; swap C2 for one reading lower; turn CV1 up; use the 10× probe |
| No oscillation, DC voltages wrong | Q1 legs wrong; bias resistors swapped; emitter strip not cut | Check pinout with tester; base ≈ 1.34 V at 4.5 V supply |
| No oscillation, everything checks | Fake or damaged transistor | **Swap Q1** — the #1 cause of a dead 27 MHz oscillator |
| Oscillates ~9 MHz | 3rd-overtone crystal running on its fundamental | L1 tuned near 27 MHz normally prevents this; if not, you need a fundamental crystal |
| **Frequency follows L1 by hundreds of kHz** | Crystal not in control (running as an LC oscillator): crystal open, C1 open, crystal branch not grounded through CV1/eb | Check the base–C1–crystal–CV1–eb path end to end |
| Frequency far from 27 MHz (e.g. 24 or 30 MHz) | Same as above | Same as above |
| Stops when the counter is connected | Counter loading the collector | Couple loosely (§9.2) |
| Works with 9 V direct, not from LM386 | LM386 pin 5 not reaching L1 pin 1; SW3-B wiring | Measure 4.5 V at L1 pin 1 with PTT pressed |
| Works on dummy load, stops with antenna | Antenna loads the tank too heavily or is detuned | Retune L1 with the antenna fitted; tune the loading coil; use wire antenna to compare |
| No modulation — RF steady when you speak | RV1 at zero; mic path via SW3-A wrong; LM386 not supplying L1 | Turn RV1 up; check C13 → SW3-A → RV1 |
| RF band disappears on loud sounds | Over-modulation | Turn RV1 down |
| Frequency drifts a lot when touched | Crystal not in control | Test E in §6.5 |

### 10.4 Receiver (before the quench add-on)

| Symptom | Likely cause | Check / fix |
|---|---|---|
| Continuous 27 MHz sine, no hiss | **As predicted by simulation** | Fit the quench add-on (§6.4) |
| No oscillation at all | L2 wrong; Q2 legs; C7/C8 swapped; missing ground on C9 | Swap Q2; check the pinout; C9 must sit at the emitter end |
| Oscillates far from 27 MHz (counter via a gimmick wire) | L2 needs adjusting | Spread L2 to go higher, squeeze to go lower |

### 10.5 Receiver with the quench add-on

| Symptom | Likely cause | Check / fix |
|---|---|---|
| 555 pin 3 no square wave | 555 not powered (pin 8/4 on RX supply?); C1 open; pins 2/6 not joined | Check pin 8 ≈ 9 V on receive; check the wiring table |
| Square wave but not ~50 % | 1N4148 missing or backwards | **Band towards pins 6/2** |
| Square wave, no RF bursts on Q2 | Pin 3 not reaching L2 pin 1 / R5; Q2 stage fault | Measure DC at L2 pin 1 (~3.5 V average) |
| Bursts but no hiss | Audio path: C12 or SW3-A | Finger test on RV1 input |
| Hiss but it doesn't quieten near the jig or the other unit | L2 off frequency; antenna not reaching L2 pin 4 via SW2-A | Tune L2; check SW2-A released → L2 pin 4 |
| Audible whine | Quench too low (1 nF → 22 kHz); C11 missing | Use 680 pF or 470 pF; check C11 |
| Hears the other unit only very close | L2 tuning; weak quench; antennas | Tune L2 carefully; try the other C1 values; use wire antennas |
| Loud hiss that barely changes with a signal present | **C6 too large** — 47 nF or 100 nF fitted instead of 33 nF | Measure C6; fit 22–39 nF (33 nF best) |

### 10.6 PTT switch

| Symptom | Likely cause | Check / fix |
|---|---|---|
| Receive works, transmit doesn't (or the reverse) | A pole's commons or throws swapped | Re-identify pins (§6.6) and check each pole against §2 |
| Hiss continues while pressed | SW2-B not cutting the RX supply | 555/RX supply must come **only** from SW2-B pin 6 |
| Speaker makes noise while transmitting | SW3-B not moving the LM386 output away from the speaker | Check SW3-B |
| Works only when the switch is held at an angle | Worn contacts or cracked solder joint | Reflow; replace the switch |

### 10.7 Range and "strange" problems

| Symptom | Likely cause | Check / fix |
|---|---|---|
| Works at 1 m, fails at 10 m | Antennas; no counterpoise; L2 off tune; weak batteries | Use wire + counterpoise; retune L2 with the other unit transmitting; fresh batteries |
| Much worse with whips than wire | Loading coil mistuned | Remove turns one at a time for loudest audio (§6.9) |
| A → B works, B → A doesn't | Fault is in A's receiver or B's transmitter | Swap roles one at a time to isolate |
| Only works while you touch the board | Dry joint; stage on the edge of oscillating; coil moving | Reflow; glue coils; retune |
| Changes when you move your hand near it | Normal a little on veroboard at 27 MHz; a lot = poor grounding | Improve ground links; keep hands off RF islands |
| Intermittent crackle | Solid-core wire fatigue on PTT/speaker; loose battery clip | Use stranded wire; check the clip |
| The two units interfere at close range | Normal — they're very strong at 1 m | Test at 5 m or more |

---

## 11. Record sheet

Comparing A with B is the fastest way to debug unit B.

| Measurement | Unit A | Unit B |
|---|---|---|
| Battery voltage under load | | |
| Current — receive | | |
| Current — transmit | | |
| LM386 pin 5 (≈ 4.5 V) | | |
| Mic + (≈ 4–8 V) | | |
| Q1 base / emitter | | |
| Q2 base / emitter (before quench) | | |
| Crystal in test jig | | |
| Transmitter frequency | | |
| Dummy-load amplitude (peak) | | |
| L1 final turns / length | | |
| L2 final turns / length | | |
| CV1 position | | |
| Quench C1 value chosen | | |
| Loading-coil turns | | |
| Range — wire antennas | | |
| Range — whips | | |
