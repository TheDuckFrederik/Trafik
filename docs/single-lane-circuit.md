# Single Lane Trafik circuit design — Revision A (passive, archived)

> **Archived 2026-10-13.** The product architecture pivoted to the active platform (three buffers, microcontroller control; see [architecture.md](architecture.md)). This passive, fixed-function-CMOS design is retained in full as the verified bench reference, the source of the relay contact matrix and allocation truth table, and the fallback design. Its KiCad capture (`SingleLane/SingleLane.kicad_sch`) remains netlist-verified with a clean ERC. Nothing on this page describes the active product circuit.

> **Design status — Revision A (2026-10-13), not yet built.** This is a complete functional, contact-level design verified against every routing, allocation, power-loss, and indication requirement in [architecture.md](architecture.md) and [single-lane-PLACEHOLDER.md](single-lane-PLACEHOLDER.md). It has **not** been physically built or measured. The design is captured as a KiCad 10 schematic at `SingleLane/SingleLane.kicad_sch` (label-based connectivity; its exported netlist matches the pin-to-net tables in this document exactly, and it passes ERC with zero violations). Before fabrication: verify every pin number against the exact manufacturer datasheet for the parts you actually buy, and pass the build-gate checks in [build-guide.md](build-guide.md) §6–§7.

## Scope

This document is the circuit design for the **Single Lane Trafik product**: two independently controlled loop positions (Pre-FX, Pre-TO), one fixed device path, eight jacks, two momentary footswitches, eight state LEDs, 9 V pedal power, no firmware.

It supersedes the "relay count cannot be finalized" caveat in [single-lane-PLACEHOLDER.md](single-lane-PLACEHOLDER.md) with a concrete design: **4 DPDT relays and 7 CMOS ICs**. It is separate from the earlier [single-loop bench prototype](single-loop-prototype.md), which proved the debounce/flip-flop/driver pattern on one buffered loop; this product circuit is fully passive in the audio path and needs no op amp and no 5 V regulator.

Key differences from the bench prototype, and why:

| Prototype choice | Product choice here | Reason |
| --- | --- | --- |
| TL072 input buffer | No buffer (passive audio) | Product family is passive-audio per README/architecture; nothing in the routing needs gain or impedance conversion. |
| LM7805 + 5 V rail + 74HC14 | No regulator; 9 V rail; **CD40106BE** Schmitt | 74HC14 is rated 2–6 V and cannot run on the 9 V rail; CD4000B-series parts (CD4017/CD4001/CD40106) are specified to 18 V. Dropping the regulator removes a part, heat, and headroom questions. |
| 5 V relay coil (TQ2-5V) | 9 V coil (TQ2-9V class) | Coils run straight off the protected 9 V rail; ~14 mA per coil instead of ~28 mA. |
| CD4013 toggle, 2 states | CD4017 ring counter, 4 states per position | Each position cycles Off → A → B → A→B → Off. |

## Design summary

| Block | Implementation | Count |
| --- | --- | ---: |
| Loop-insert relays | DPDT non-latching, 9 V coil (Panasonic TQ2-9V class) | 4 |
| Loop-position counters | CD4017BE wired as a 4-state ring (Q4 → RESET) | 2 |
| Debounce | CD40106BE hex Schmitt, one gate per footswitch | 1 |
| Interlock + LED decode | CD4001BE quad NOR (16 of 16 gates used) | 4 |
| Relay drivers | 2N3904 NPN low-side + 1N4148 flyback per coil | 4 |
| State LEDs | 8 diffused LEDs (white/green/red/amber × 2 positions), 2.2 kΩ series each | 8 |
| Momentary footswitches | SPST-NO soft-touch stomps; contacts carry no audio | 2 |

**Why 4 relays are sufficient.** There are two loop positions and two physical loops, so there are exactly four "insert this loop at this position" functions: Loop A at Pre-FX (K1), Loop B at Pre-FX (K2), Loop A at Pre-TO (K3), Loop B at Pre-TO (K4). Each is one DPDT configured as a standard series insert/bypass stage. The two relays that share a physical loop (K1 & K3 for Loop A; K2 & K4 for Loop B) tie their NO contacts to the same send/return jacks; the allocation interlock guarantees the two coils are never energized together, so the shared jacks are never driven from both chains. The earlier 10-relay BOM allowance was a pre-design guess.

**Worst-case power:** the interlock caps energized coils at 2 (you cannot apply more than two loops when there are only two loops), so worst-case draw ≈ 2 coils × ~14 mA + 2 LEDs × ~3 mA + CMOS quiescent ≈ **36 mA**. Use a regulated isolated 9 V center-negative supply rated ≥100 mA and measure the finished unit.

## Audio switching

### Signal chains

```text
LEGEND: --- audio   [part]   (net name)   all jack sleeves -> AGND   AGND joins GND at one star

CHAIN 1 — instrument path                     CHAIN 2 — device FX path
J1 Instrument IN                              J4 FX Return 1
  |                                             |
  v                                             v
K1 (Loop A @ Pre-FX insert)                   K3 (Loop A @ Pre-TO insert)
K2 (Loop B @ Pre-FX insert)                   K4 (Loop B @ Pre-TO insert)
  |                                             |
  v                                             v
J2 TO 1                                       J3 FX Send 1
```

### Relay insert stage (identical pattern for K1–K4)

Each DPDT relay is a series insert/bypass stage. Functional contact names only — map to the physical terminals of the exact relay datasheet:

| Pole | COM | NC (coil off: bypass) | NO (coil on: loop inserted) |
| --- | --- | --- | --- |
| A | stage input | `LINK_Kn` | loop send net |
| B | stage output | `LINK_Kn` (joins pole A NC) | loop return net |

Bypass path: stage input → COM_A → NC_A → `LINK_Kn` → NC_B → COM_B → stage output.
Insert path: stage input → COM_A → NO_A → loop Send jack → external pedals → loop Return jack → NO_B → COM_B → stage output.

### Contact matrix

| Relay | Function | COM_A | NO_A | NO_B | COM_B | Coil on when |
| --- | --- | --- | --- | --- | --- | --- |
| K1 | Loop A at Pre-FX | `IN_TIP` | `LA_SEND` | `LA_RET` | `PFX_MID` | `X1` = Loop A requested at Pre-FX |
| K2 | Loop B at Pre-FX | `PFX_MID` | `LB_SEND` | `LB_RET` | `TO1_TIP` | `X2` = Loop B requested at Pre-FX |
| K3 | Loop A at Pre-TO | `FXR1_TIP` | `LA_SEND` | `LA_RET` | `PRETO_MID` | `A2P` = Loop A applied at Pre-TO (after interlock) |
| K4 | Loop B at Pre-TO | `PRETO_MID` | `LB_SEND` | `LB_RET` | `FXS1_TIP` | `B2P` = Loop B applied at Pre-TO (after interlock) |

`LA_SEND` and `LA_RET` each connect to the NO contacts of **both** K1 and K3 (same for Loop B with K2/K4). An open NO contact is isolated internally, so the unselected position never loads the loop.

**Unpowered / power-loss state:** all coils off → all COMs on NC → `IN_TIP → K1 → K2 → TO1_TIP` and `FXR1_TIP → K3 → K4 → FXS1_TIP` straight through, both loops bypassed, all loop jacks isolated. This meets the architecture's power-loss requirement exactly, and because the audio path is fully passive, signal still passes with DC removed (unlike the buffered bench prototype).

**Pull-down resistors (1 MΩ to GND)** on `IN_TIP`, `TO1_TIP`, `FXR1_TIP`, `FXS1_TIP`, `LA_SEND`, `LA_RET`, `LB_SEND`, `LB_RET`, `PFX_MID`, `PRETO_MID` keep every switched node DC-referenced so contact transfer does not dump charge into the audio (switching-pop control). There is no DC anywhere in the audio path.

**Known transient:** when a loop is *reallocated* (requested at Pre-TO, then also requested at Pre-FX), the energized coil must release while the other pulls in. Relay release is typically faster than operate, but worst-case overlap of ~1–2 ms can briefly share the loop between the two chains. It is not damaging at instrument level. If audible, the fix is a turn-on delay on the K1/K2 driver networks — not required for Revision A.

### Full state × relay × LED truth table

Applied Pre-TO is the requested state minus loops already taken by Pre-FX. This table is the acceptance truth table for the build-guide checks:

| # | Pre-FX req | Pre-TO req | K1 | K2 | K3 | K4 | Chain 1 (IN→TO 1) | Chain 2 (FXR1→FXS1) | Pre-FX LED | Pre-TO LED |
| ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Off | Off | · | · | · | · | direct | direct | white | white |
| 2 | Off | A | · | · | ● | · | direct | A | white | green |
| 3 | Off | B | · | · | · | ● | direct | B | white | red |
| 4 | Off | A→B | · | · | ● | ● | direct | A then B | white | amber |
| 5 | A | Off | ● | · | · | · | A | direct | green | white |
| 6 | A | A | ● | · | · | · | A | direct (A stolen) | green | white |
| 7 | A | B | ● | · | · | ● | A | B | green | red |
| 8 | A | A→B | ● | · | · | ● | A | B (A stolen) | green | red |
| 9 | B | Off | · | ● | · | · | B | direct | red | white |
| 10 | B | A | · | ● | ● | · | B | A | red | green |
| 11 | B | B | · | ● | · | · | B | direct (B stolen) | red | white |
| 12 | B | A→B | · | ● | ● | · | B | A (B stolen) | red | green |
| 13 | A→B | Off | ● | ● | · | · | A then B | direct | amber | white |
| 14 | A→B | A | ● | ● | · | · | A then B | direct (A stolen) | amber | white |
| 15 | A→B | B | ● | ● | · | · | A then B | direct (B stolen) | amber | white |
| 16 | A→B | A→B | ● | ● | · | · | A then B | direct (both stolen) | amber | white |

● = coil energized. Rows 6, 8, 11, 12, 14–16 exercise the Pre-FX-priority interlock; the Pre-TO LED shows the **applied** state, never the overridden request. Loop A is always before Loop B in series because K1 precedes K2 (and K3 precedes K4) in the chain — the ordering is wired, not logical.

## Control logic

All control ICs run from the protected 9 V rail (`+9V_PROT`). CD4000B inputs switch at ≈0.7 × VDD, so the design keeps every counter output lightly loaded (see "Loading discipline" below).

### Debounce (U3 CD40106BE)

Per footswitch: SW to GND, R 100 kΩ pull-up to `+9V_PROT`, C 100 nF switch node to GND, into one CD40106 inverter; inverter output → counter CLK. Press pulls the node low (fast, through the switch), release lets it rise with τ = 10 ms; press bounce cannot re-cross the Schmitt threshold and release bounce cannot re-trigger a count within realistic bounce times. One clean rising CLK edge per press. If a particular switch double-counts, increase C to 220 nF.

Pin usage (DIP-14, verify against the datasheet): gate 1 = pins 1→2 (SW1), gate 2 = pins 3→4 (SW2). VDD pin 14, GND pin 7. Unused inputs pins 5, 9, 11, 13 → GND; outputs unconnected.

### Four-state counters (U1, U2 CD4017BE)

U1 = Pre-FX position, U2 = Pre-TO position. DIP-16 pin numbers (verify datasheet): VDD 16, VSS 8, CLK 14, CLOCK INHIBIT 13 (**tie to GND — never leave floating**), RESET 15, Q0=3, Q1=2, Q2=4, Q3=7, Q4=10.

- State assignment: Q0 = Off, Q1 = A, Q2 = B, Q3 = A→B.
- Four-state ring: wire Q4 (pin 10) directly to RESET (pin 15). The fourth press advances to Q4, which immediately resets to Q0 — the Off → A → B → A→B → Off cycle.
- Power-on reset: C 100 nF from `+9V_PROT` to RESET, R 100 kΩ RESET to GND. At plug-in the cap couples the rail step onto RESET, forcing Q0 (Off) for ~10 ms.
- Q5–Q9 and carry-out: unconnected.

**Carried-forward caveat from the architecture:** an RC reset is a startup aid, not a precision supervisor. On a very slow supply ramp the counters can come up in a random state. Test with slow and interrupted ramps as the build guide requires; if any test fails, add a proper reset supervisor before shipping the design. Isolated pedal supplies ramp in well under 10 ms, so this is expected to pass, but it must be demonstrated, not assumed.

### Request bits (diode-OR)

Counter outputs are push-pull and must never be wired together; requests are OR-ed with 1N4148 diodes into 100 kΩ pull-down nodes:

```text
X1 (Pre-FX requests A) = U1.Q1 OR U1.Q3   via D1, D2
X2 (Pre-FX requests B) = U1.Q2 OR U1.Q3   via D3, D4
Y1 (Pre-TO requests A) = U2.Q1 OR U2.Q3   via D5, D6
Y2 (Pre-TO requests B) = U2.Q2 OR U2.Q3   via D7, D8
```

### Allocation interlock (U4 CD4001BE) — Pre-FX priority

The architecture's applied-request equations:

```text
Applied Pre-FX:  X1, X2 (unchanged — priority position)
Applied Pre-TO:  A2P = Y1 AND NOT X1
                 B2P = Y2 AND NOT X2
```

In NOR form (`NOT a = NOR(a,a)`; `a·¬b = NOR(b, ¬a)`):

| Gate | Inputs | Output | Meaning |
| --- | --- | --- | --- |
| U4a (pins 1,2→3) | Y1, Y1 | `NY1` = ¬Y1 | |
| U4b (pins 5,6→4) | Y2, Y2 | `NY2` = ¬Y2 | |
| U4c (pins 8,9→10) | X1, NY1 | `A2P` = Y1·¬X1 | Loop A applied at Pre-TO; drives K3 |
| U4d (pins 12,13→11) | X2, NY2 | `B2P` = Y2·¬X2 | Loop B applied at Pre-TO; drives K4 |

The interlock is combinational and sits **before** both the relay drivers and the Pre-TO LEDs, as the architecture requires: a loop requested at Pre-TO never has its send/return connected while Pre-FX holds the same loop.

### LED decode (U5–U7 CD4001BE)

Exactly one LED per position is on, showing the applied state:

| Gate | Inputs | Output → LED |
| --- | --- | --- |
| U5a (1,2→3) | A2P, A2P | `NA2P` = ¬A2P |
| U5b (5,6→4) | B2P, B2P | `NB2P` = ¬B2P |
| U5c (8,9→10) | NA2P, B2P | green Pre-TO (LED6) = A2P·¬B2P |
| U5d (12,13→11) | A2P, NB2P | red Pre-TO (LED7) = ¬A2P·B2P |
| U6a (1,2→3) | NA2P, NB2P | amber Pre-TO (LED8) = A2P·B2P |
| U6b (5,6→4) | A2P, B2P | white Pre-TO (LED5) = ¬A2P·¬B2P |
| U6c (8,9→10) | X1, X1 | `NX1` = ¬X1 |
| U6d (12,13→11) | X2, X2 | `NX2` = ¬X2 |
| U7a (1,2→3) | NX1, X2 | green Pre-FX (LED2) = X1·¬X2 |
| U7b (5,6→4) | X1, NX2 | red Pre-FX (LED3) = ¬X1·X2 |
| U7c (8,9→10) | NX1, NX2 | amber Pre-FX (LED4) = X1·X2 |
| U7d (12,13→11) | X1, X2 | white Pre-FX (LED1) = ¬X1·¬X2 |

**All 16 gates of U4–U7 are used; there are no spares.** Any future logic change requires a fifth CD4001 or a re-allocation. Unused inputs do not exist; on any partially-used CD4001 in a future revision, tie unused inputs to GND.

At power-up (both counters reset → Q0): X1=X2=Y1=Y2=0 → all coils off, LED1 (white) and LED5 (white) on. Correct.

### Relay drivers (Q1–Q4)

Per relay: 2N3904 NPN low-side.

| Element | Connection |
| --- | --- |
| Base resistor 10 kΩ | from drive node (X1, X2, A2P, or B2P) to base |
| Base pull-down 100 kΩ | base to GND (holds coil off during reset and whenever the driving node is not driven) |
| Emitter | GND |
| Collector | coil negative terminal |
| Coil positive terminal | `+9V_PROT` |
| Flyback 1N4148 | directly across the coil, **cathode to the coil positive terminal** |

Forced beta ≈ 14 mA / 0.8 mA ≈ 18 against a minimum hFE of ~70, so the driver saturates with margin. Relay must-operate (typically 75% of 9 V = 6.75 V) is comfortably met at the ~8.5 V the coil sees after the protection diode and VCE(sat).

### Loading discipline (do not break these margins)

CD4001B inputs need ≈0.7 × VDD ≈ 6.3 V to read a high, and CD4000B outputs droop under load. The design is only correct with these load rules:

- **Counter outputs (U1, U2) drive nothing but 1N4148 diodes.** Q3 sources two diode loads ≈1.5 mA worst case → output ≈8.1 V → request node ≈7.5 V, a >1 V noise margin over 6.3 V.
- **Every LED is driven from a dedicated CD4001 gate output** (≈3 mA), never from a counter output. Putting an LED on a counter output would droop it below the interlock's input threshold — exactly the fault the interlock exists to prevent.
- **Request nodes (X1, X2) additionally drive one NPN base** (≈0.8 mA through a diode); Y nodes drive only CMOS inputs.

If you modify this design, re-verify these margins before adding any load to an existing node.

## Power

```text
DC jack (2.1 mm, center-negative: center = PSU− = GND, sleeve = PSU+)
  sleeve === [D13 1N5817, anode->cathode] === (+9V_PROT) ===+=== [C19 100uF +] === GND
                                                           +=== [C20 100nF]   === GND
                                                           +=== [C21 10uF +]  === GND  (near relay/IC cluster)
```

- D13 reverse-protection drop ≈0.3 V at this current: coils still see ≈8.5–8.7 V, well above must-operate.
- Decoupling: 100 nF across VDD–VSS **at each of the seven ICs**, plus the rail capacitors above. Coil switching dips are small (worst step ≈28 mA) but keep the bulk capacitor near the relay coils.
- Worst-case budget: 2 coils × ~14 mA + 2 LEDs × ~3 mA + CMOS ≈ **36 mA**. Recommended supply: regulated isolated 9 V center-negative, ≥100 mA, measured on the finished unit.

## Grounding

Per [architecture.md](architecture.md#power-grounding-and-wiring):

- All eight jack sleeves → `AGND`, bonded to `GND` at exactly one star point (the PSU-negative reference). Bond the enclosure once, at that point.
- Logic, LED, and coil returns route to `GND` separately from the audio sleeve returns; the enclosure is never the audio return.
- Keep input/loop audio wiring short and away from CLK, LED, and coil runs; twist coil pairs; cross bundles at right angles if unavoidable.
- A device connected to both TO 1 and the FX pair creates an external ground loop through Trafik's common ground — that is a property of the rig, not of Trafik. Solve it at the amps (ground lift / isolation), never by defeating Trafik's internal star.

## Suggested bill of materials

Specific starting suggestions; verify package, pinout, and ratings in the current manufacturer datasheet before ordering.

| Ref. | Qty. | Suggested part | Role |
| --- | ---: | --- | --- |
| U1, U2 | 2 | TI **CD4017BE** | Four-state ring counter per loop position (DIP-16) |
| U3 | 1 | TI **CD40106BE** | Hex Schmitt inverter; two debounce gates (DIP-14) |
| U4–U7 | 4 | TI **CD4001BE** | Quad NOR: interlock + LED decode (DIP-14) |
| K1–K4 | 4 | Panasonic **TQ2-9V** (non-latching DPDT, 9 V coil) | Loop insert stages. Do **not** substitute a latching (TQ2-L2) type: it breaks the de-energized-bypass power-loss state and burns the driver. |
| Q1–Q4 | 4 | onsemi **2N3904BU** | Coil low-side drivers (verify E/B/C lead order) |
| D1–D8 | 8 | onsemi **1N4148** | Request diode-OR |
| D9–D12 | 4 | onsemi **1N4148** | Coil flyback (cathode to coil +) |
| D13 | 1 | Vishay **1N5817-E3/54** | Series reverse-polarity protection |
| LED1/LED5 | 2 | 5 mm diffused **white** (e.g. Kingbright WP7113SWD) | Off indication, both positions |
| LED2/LED6 | 2 | 5 mm diffused **green** | Loop A |
| LED3/LED7 | 2 | 5 mm diffused **red** | Loop B |
| LED4/LED8 | 2 | 5 mm diffused **amber** | A→B |
| SW1, SW2 | 2 | Momentary **SPST-NO** soft-touch stomp switches | Pre-FX, Pre-TO (contacts carry no audio) |
| J1–J8 | 8 | Insulated-body mono ¼″ TS (e.g. Rean NYS215; or Switchcraft 12A **with** insulating shoulder washers) | IN, TO 1, FX Send 1, FX Return 1, Loop A/B Send/Return |
| DC jack | 1 | Switchcraft **722A** (verify center/sleeve tabs) | 2.1 mm, wired center-negative |
| R1–R8 | 8 | 2.2 kΩ ¼ W 1% | LED series (≈3 mA; white ≈2.7 mA — verify brightness) |
| R9–R12 | 4 | 10 kΩ ¼ W 1% | Driver base series |
| R13–R16 | 4 | 100 kΩ ¼ W 1% | Driver base pull-downs |
| R17, R18 | 2 | 100 kΩ ¼ W 1% | Switch pull-ups |
| R19, R20 | 2 | 100 kΩ ¼ W 1% | Counter reset pull-downs |
| R21–R24 | 4 | 100 kΩ ¼ W 1% | Request-node pull-downs |
| R25–R34 | 10 | 1 MΩ ¼ W 1% | Audio node pull-downs (pop control) |
| C1, C2 | 2 | 100 nF film | Switch debounce |
| C3, C4 | 2 | 100 nF | Counter power-on reset |
| C5–C11 | 7 | 100 nF | IC decoupling, one per IC at its pins |
| C12, C13 | 2 | 100 nF | — spare positions / bulk rail assist |
| C19 | 1 | 100 µF ≥16 V electrolytic | Rail bulk (observe polarity) |
| C20 | 1 | 100 nF | Rail HF bypass |
| C21 | 1 | 10 µF ≥16 V electrolytic | Local bulk near relay/IC cluster |
| — | 1 lot | DIP sockets, standoffs, hook-up wire, shielded cable for long audio runs, heat-shrink | Use IC sockets: 7 DIPs are far easier to replace than to re-solder in a product |

Panel fit: 8 jacks + 2 stomps + 8 LEDs + DC jack in a 1590B-class enclosure remains subject to the full-size fit check in [single-lane-PLACEHOLDER.md](single-lane-PLACEHOLDER.md); fall back to 1590BB-class if the mock-up is tight.

## Test additions for this circuit

Run the full [build guide](build-guide.md) §6–§7 procedure, then these design-specific checks:

1. **Truth table walk:** step both footswitches through all 16 rows of the table above; verify coils (audible click + LED) and continuity per chain at low level.
2. **Interlock rows specifically (6, 8, 11, 12, 14–16):** request a loop at Pre-TO that Pre-FX already holds; confirm the loop moves to Pre-FX, the Pre-TO LED shows the applied (not requested) state, and no relay pair K1+K3 or K2+K4 is ever energized together (meter the coil voltages).
3. **Power-on reset:** cycle power and slow ramps; both positions must come up Off (white LEDs) with all coils de-energized. If a slow ramp ever produces a random state, add a reset supervisor before shipping.
4. **Power loss during signal:** remove DC while signal flows; audio must continue through the passive bypass path.
5. **Reallocation transient:** hold Loop A at Pre-TO, then also request it at Pre-FX; listen for the handoff. A brief (~1–2 ms worst case) shared-loop overlap is expected and acceptable; a lasting overlap or audible thump is not — re-check the interlock wiring.

## Scalability note

The control core (debounce → CD4017 ring counters → diode-OR requests → NOR interlock/decode → NPN drivers) is the reusable atom for the 2- and 4-lane models: they add a modulo-N lane counter (another CD4017 ring), lane contact groups, and lane LEDs, while the loop-position logic is unchanged.

## Datasheet references

- [TI CD4017B](https://www.ti.com/lit/ds/symlink/cd4017b.pdf), [TI CD4001B](https://www.ti.com/lit/ds/symlink/cd4001b.pdf), [TI CD40106B](https://www.ti.com/lit/ds/symlink/cd40106b.pdf)
- [Panasonic TQ relay series](https://industry.panasonic.com/global/en/products/control/relay/signal/number/tq) — select the exact TQ2-9V datasheet; terminal arrangement is package/view-dependent
- [onsemi 2N3904](https://www.onsemi.com/pdf/datasheet/2n3903-d.pdf), [Vishay 1N5817](https://www.vishay.com/docs/88525/1n5817.pdf), onsemi 1N4148

A part number alone does not establish pin numbers. Verify datasheet revision, ordering suffix, pin-view orientation, and footprints at purchase and before applying power.
