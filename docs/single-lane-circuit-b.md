# Single Lane Trafik circuit — Revision B (active platform)

> **Design status — Revision B, exact-pin design (2026-10-20), not yet drawn or built.** The complete pin-by-pin connection tables (final reference designators) are in [single-lane-circuit-b-netlist.md](single-lane-circuit-b-netlist.md) — draw the schematic from those. This is the complete design for the active Single Lane product: every part, value, connection, and pin assignment needed to draw the schematic and build the board. The audio contact topology, state machine, and truth tables are inherited unchanged from the netlist-verified Revision A ([single-lane-circuit.md](single-lane-circuit.md), `SingleLane/SingleLane.kicad_sch`); Revision A's CMOS control is replaced by a microcontroller, and the three-buffer architecture of [architecture.md](architecture.md) is added. Nothing here has been physically verified: bench-test per the build gate before connecting gear.

## 1. System summary

| Block | Implementation |
| --- | --- |
| Audio switching | 5 × DPDT non-latching relays, Panasonic TQ2-9V class (4 loop inserts + 1 output-buffer bypass) |
| Buffers | 3 × unity followers, one quad CMOS op-amp (LMC660AIN class) on the 9 V rail |
| Control | ATmega328P-PU (DIP-28), 5 V, internal 8 MHz oscillator, Arduino toolchain |
| Drivers | 5 × 2N3904 low-side + 1N4148 flyback per coil |
| Inputs | 2 × momentary SPST-NO footswitches to GND (onboard stomps; optional remote jacks, §8) |
| Indication | 8 LEDs (white/green/red/amber per position), driven directly by MCU pins |
| Power | 9 V DC center-negative in → 1N5817 reverse protection → 9V_PROT rail (coils, op-amp) → 78L05 → 5 V logic |
| Worst-case draw | ≈ 65 mA (3 coils + 2 LEDs + logic + op-amp); use a ≥100 mA isolated supply |

Signal chains (functional):

```text
CHAIN 1 (instrument):  J1 IN → [K1: Loop A @ Pre-FX] → [K2: Loop B @ Pre-FX] →
                       [K5: output buffer / hardwire bypass] → J2 TO 1
CHAIN 2 (device FX):   J4 FX Return 1 → [K3: Loop A @ Pre-TO] → [K4: Loop B @ Pre-TO] → J3 FX Send 1
LOOPS:                 J5/J6 = Loop A Send/Return (shared by K1, K3)
                       J7/J8 = Loop B Send/Return (shared by K2, K4)
                       Each loop Return enters its buffer before reaching any relay contact.
```

## 2. Audio section

### 2.1 Loop insert stages (K1–K4)

Identical contact pattern to Revision A (functional EN50005 names; verify against the TQ2 datasheet terminals):

| Relay | COM_A | NO_A | NO_B | COM_B | NC_A–NC_B bridge | Coil on when |
| --- | --- | --- | --- | --- | --- | --- |
| K1 | `IN_TIP` | `LA_SEND` | `BUF1_OUT` | `PFX_MID` | `LINK_K1` | Loop A applied at Pre-FX |
| K2 | `PFX_MID` | `LB_SEND` | `BUF2_OUT` | `OUT3_IN` | `LINK_K2` | Loop B applied at Pre-FX |
| K3 | `FXR1_TIP` | `LA_SEND` | `BUF1_OUT` | `PRETO_MID` | `LINK_K3` | Loop A applied at Pre-TO |
| K4 | `PRETO_MID` | `LB_SEND` | `BUF2_OUT` | `FXS1_TIP` | `LINK_K4` | Loop B applied at Pre-TO |

Differences from Rev A: the loop **Return** jacks no longer land on NO_B directly — each return jack feeds its buffer, and the **buffer output** (`BUF1_OUT`/`BUF2_OUT`) feeds both relays' NO_B contacts (shared loop, both chains, one buffer each). Loop **Send** jacks still connect straight to NO_A (passive — fuzz rule).

### 2.2 Buffer stages (U2, one LMC660AIN quad)

Three identical unity stages; the fourth amplifier is tied off as a biased follower (spare — tuner-out roadmap).

```text
Buffer n (n = 1: Loop A Return, 2: Loop B Return, 3: output):

  jack/relay node --[Cin 1uF film]--+--> U2 +in        U2 out --[Rout 100R]--[Cout 1uF film]--+--> BUFn_OUT
                                    |                  (follower)                            |
                              [Rbias 1M] to VBIAS                                          [Rpd 1M] to GND
```

| Stage | Input comes from | `BUFn_OUT` goes to |
| --- | --- | --- |
| Buffer 1 | J6 Loop A Return tip (via Cin; jack side has Rpd 1M to GND) | K1.NO_B and K3.NO_B |
| Buffer 2 | J8 Loop B Return tip (same) | K2.NO_B and K4.NO_B |
| Buffer 3 | K5.NO_A | K5.NO_B |

`VBIAS` (= +4.5 V) is one shared reference: 100 kΩ from 9V_PROT + 100 kΩ to GND, bypassed with 10 µF ∥ 100 nF to GND. Op-amp supply: V+ = 9V_PROT, V− = GND, 100 nF decoupling at the package. The LMC660-class output swings close to the rails and its input range covers mid-rail, giving ≈±3 V of headroom around VBIAS — far beyond instrument level. (TL074 only with the documented headroom caveat; do not use 5.5 V-max RRIO parts like MCP6004 on this rail.)

Roll-off check: 1 µF into 1 MΩ ≈ 0.16 Hz at input; output 1 µF into ≥500 kΩ (amp input ∥ pulldown) ≈ 0.3 Hz — flat through audio.

### 2.3 Output buffer bypass (K5)

K5 is a fifth DPDT wired as another insert stage: **de-energized = hardwire**, energized = buffer 3 in circuit.

| Relay | COM_A | NO_A | NO_B | COM_B | NC_A–NC_B bridge | Coil on when |
| --- | --- | --- | --- | --- | --- | --- |
| K5 | `OUT3_IN` (K2.COM_B) | buffer 3 input node | `BUF3_OUT` | `TO1_TIP` | `LINK_K5` (hardwire) | MCU booted and healthy — always on in normal operation |

During MCU boot (pins Hi-Z, base pulldowns hold all drivers off) and on any power loss or firmware fault, K5 is de-energized: `OUT3_IN → LINK_K5 → TO1_TIP` — a literal wire. Loop-return buffers need no bypass: their loop relays de-energize and remove them from the circuit.

### 2.4 Unpowered / fault state (complete)

All coils off → `IN_TIP → K1 NC → K2 NC → K5 NC → TO1_TIP` and `FXR1_TIP → K3 NC → K4 NC → FXS1_TIP`, both loops and all buffers out of circuit, loop jacks isolated. Audio passes with the plug pulled. This is the fail-safe the platform promises.

### 2.5 Ground-referencing pulldowns (1 MΩ to GND)

On: `IN_TIP`, `PFX_MID`, `OUT3_IN`, `TO1_TIP`, `FXR1_TIP`, `FXS1_TIP`, `PRETO_MID`, `LA_SEND`, `LB_SEND` (9) plus the three buffer-stage output pulldowns and the two loop-return jack-side pulldowns counted in §2.2 — **14 × 1 MΩ total**. These keep every relay-switched node discharged so contact transfer is pop-free.

### 2.6 Truth table

Identical to Revision A's 16-row state table (see [single-lane-circuit.md](single-lane-circuit.md) §"Full state × relay × LED truth table") with one addition: **K5 is energized in every powered state.** The firmware implements the same allocation — Pre-FX priority, applied-state LEDs, A-before-B series order.

## 3. Control section (U1 ATmega328P-PU)

Why this MCU: 5 V logic drives the 2N3904 bases directly, DIP-28 is hand-buildable, 20 GPIO covers this design with spares, the hardware watchdog satisfies the fail-safe spec, and the Arduino toolchain keeps the firmware a first-project proposition. Internal 8 MHz oscillator: no crystal, PB6/PB7 free. Program via a 6-pin ISP header (fuse: internal 8 MHz).

### 3.1 Pin map (DIP-28)

| Pin | Port | Net / function | Notes |
| ---: | --- | --- | --- |
| 1 | PC6 | `RESET` | 10 kΩ to +5V, 100 nF to GND; ISP header |
| 2 | PD0 | `UART_RX` (spare) | debug/MIDI roadmap — 3-pin header optional |
| 3 | PD1 | `UART_TX` (spare) |  |
| 4 | PD2 | `SW1_N` | Pre-FX footswitch input |
| 5 | PD3 | `SW2_N` | Pre-TO footswitch input |
| 6 | PD4 | `DRV_K1` | → R→Q1 base (Loop A @ Pre-FX) |
| 7 | VCC | +5V | 100 nF + 10 µF at the pin |
| 8 | GND | GND |  |
| 9, 10 | PB6/7 | spare | no crystal; leave on a spare header |
| 11 | PD5 | `DRV_K2` | Loop B @ Pre-FX |
| 12 | PD6 | `DRV_K3` | Loop A @ Pre-TO |
| 13 | PD7 | `DRV_K4` | Loop B @ Pre-TO |
| 14 | PB0 | `DRV_K5` | output-buffer bypass — **inverted logic: HIGH = buffer in, LOW/floating = hardwire** |
| 15 | PB1 | `LED1` | Pre-FX white (Off) |
| 16 | PB2 | `LED2` | Pre-FX green (A) |
| 17 | PB3 | `LED3` | Pre-FX red (B) — shared with ISP MOSI (LED load is programmer-safe) |
| 18 | PB4 | `LED4` | Pre-FX amber (A→B) — ISP MISO |
| 19 | PB5 | `LED5` | Pre-TO white — ISP SCK |
| 20 | AVCC | +5V | 100 nF to GND |
| 21 | AREF | — | 100 nF to GND, otherwise unconnected |
| 22 | GND | GND |  |
| 23 | PC0 | `LED6` | Pre-TO green |
| 24 | PC1 | `LED7` | Pre-TO red |
| 25 | PC2 | `LED8` | Pre-TO amber |
| 26 | PC3 | `SELFTEST` | self-test strap: jumper to GND at boot = truth-table walk mode |
| 27, 28 | PC4/5 | spare (I²C roadmap) | pull-up footprints optional |

### 3.2 Switch inputs

Each footswitch: switch node → **1 kΩ series** → MCU pin; 100 nF from pin to GND; internal pull-up enabled in firmware (pressed = pin LOW). The series resistor plus the cap forms the ESD/RF path for panel wiring. Optional remote/patch-panel jacks parallel these nets through normalled jacks (§8).

### 3.3 Relay drivers (Q1–Q5)

Per channel: MCU pin → 4.7 kΩ → 2N3904 base; 100 kΩ base-to-GND (holds the coil **off** while the MCU boots with pins Hi-Z — this is what makes the fail-safe independent of firmware); emitter GND; collector → coil negative; coil positive → 9V_PROT; 1N4148 flyback directly across each coil, cathode to coil positive. Forced beta ≈ 15 vs hFE(min) ≈ 70 at 14 mA — saturated with margin. TQ2-9V must-operate (6.75 V) is met at ≈8.5 V.

### 3.4 LEDs

MCU pin → resistor → LED anode; cathode GND. Resistors: 470 Ω for the two white LEDs (≈4 mA), 1 kΩ for the six colored (≈3 mA). Sourcing 2 lit LEDs ≈ 8 mA total — within pin and package ratings.

## 4. Power

```text
DC jack (2.1 mm, center-negative: center = GND, sleeve = +9V raw)
  sleeve ──[D6 1N5817]── 9V_PROT ──┬── [C19 100µF] ── GND
                                   ├── [C20 100nF]
                                   ├── relay coils +, U2 V+
                                   └── [U3 78L05] ── +5V ── [C21 10µF] ── GND
                                                        └── [C22 100nF]
```

Worst-case budget: 3 coils × 14 mA (max simultaneous = two loops + K5) + 2 LEDs × 4 mA + MCU ≈ 10 mA + op-amp ≈ 3 mA ≈ **65 mA**. Reverse-protection drop ≈ 0.3 V leaves ≈ 8.7 V on 9V_PROT — fine for coils and headroom.

## 5. Firmware specification

1. **Init:** configure pins; watchdog on (~1 s); debrief boot; after ~100 ms energize K5 (`DRV_K5` HIGH); state = both positions Off. Until then, everything is hardwired through — power-on is graceful.
2. **Inputs:** debounce both switches ~25 ms; one event per press.
3. **State machine:** per position, cycle Off → A → B → A→B → Off independently.
4. **Allocation (before any output changes):** applied Pre-TO = requested minus loops held by Pre-FX (`A2P = Y1 && !X1`, `B2P = Y2 && !X2`) — then update relays and LEDs to the applied state per the Rev A truth table. K1+K3 (and K2+K4) can therefore never be on together.
5. **Indication:** exactly one LED per position, showing the **applied** state (Pre-TO may show Off while its request was overridden).
6. **Watchdog:** main loop kicks the WDT; a hang resets the MCU → boot state (relays off via pulldowns → hardwire) → K5 re-energizes → state Off. No relay can be held mid-state indefinitely.
7. **Self-test (PC3 strap to GND at boot):** walk the full 16-state truth table, 1 s per state, coils and LEDs per table — the made-to-order QC mode.
8. **Roadmap (do not compromise 1–6):** MIDI/presets on the spare UART/I²C; tuner-out on the spare op-amp.

## 6. Bill of materials (exact)

| Ref | Qty | Part | Value / role |
| --- | ---: | --- | --- |
| U1 | 1 | ATmega328P-PU (DIP-28) | MCU, internal 8 MHz, 5 V |
| U2 | 1 | LMC660AIN (DIP-14) | quad CMOS op-amp; 3 buffers + 1 spare |
| U3 | 1 | 78L05 (TO-92) | 5 V logic LDO |
| K1–K5 | 5 | Panasonic TQ2-9V | non-latching DPDT, 9 V coil |
| Q1–Q5 | 5 | 2N3904 | coil low-side drivers |
| D1–D5 | 5 | 1N4148 | coil flyback |
| D6 | 1 | 1N5817 | reverse-polarity protection |
| LED1, LED5 | 2 | 5 mm diffused white | Off indication |
| LED2, LED6 | 2 | 5 mm diffused green | Loop A |
| LED3, LED7 | 2 | 5 mm diffused red | Loop B |
| LED4, LED8 | 2 | 5 mm diffused amber | A→B |
| SW1, SW2 | 2 | momentary SPST-NO soft-touch stomp | 2-pin; press = to GND |
| J1–J8 | 8 | insulated mono ¼″ TS | IN, TO 1, FX Send/Ret 1, Loop A/B Send/Ret |
| J9 | 1 | 2.1 mm DC jack, center-negative |  |
| R1, R5 | 2 | 470 Ω ¼ W | white LED series |
| R2–R4, R6–R8 | 6 | 1 kΩ ¼ W | colored LED series |
| R9–R13 | 5 | 4.7 kΩ ¼ W | driver base series |
| R14–R18 | 5 | 100 kΩ ¼ W | driver base pulldowns |
| R19, R20 | 2 | 100 kΩ ¼ W | VBIAS divider (upper/lower) |
| R21, R22 | 2 | 1 kΩ ¼ W | switch input series |
| R23 | 1 | 10 kΩ ¼ W | RESET pullup |
| R24–R37 | 14 | 1 MΩ ¼ W | audio-node pulldowns (§2.5 + buffer stages) |
| R38–R40 | 3 | 1 MΩ ¼ W | buffer input bias |
| R41–R43 | 3 | 100 Ω ¼ W | buffer output series |
| C11–C13, C14–C16 | 6 | 1 µF film | buffer coupling (in / out) |
| C1, C2 | 2 | 100 nF | switch input filter |
| C3 | 1 | 100 nF | RESET |
| C4–C6 | 3 | 100 nF | VCC / AVCC / AREF |
| C7 | 1 | 10 µF ≥16 V | +5V local bulk |
| C8 | 1 | 10 µF ≥16 V | VBIAS bypass |
| C9 | 1 | 100 nF | VBIAS HF |
| C10 | 1 | 100 nF | U2 decoupling |
| C19 | 1 | 100 µF ≥16 V | 9V_PROT bulk |
| C20–C22 | 3 | 100 nF | rail HF / LDO in-out |
| — | 1 | 2×3 ISP header | programming |
| — | lots | DIP sockets (28/14), 1590B-class enclosure, standoffs, wire, shielded pair, heatshrink |  |

## 7. Verification plan

1. **Schematic (your hand-drawn KiCad):** I export the netlist with `kicad-cli` and diff every pin against an expected pin→net table generated from §2–§4 (same method that verified Rev A: 0 mismatches, clean ERC). Run ERC to zero violations.
2. **Bench, staged:** (a) unpowered continuity of both chains and isolation; (b) power without U1/U2 — rails, VBIAS ≈ 4.5 V, coils off; (c) install U2 — verify buffer unity gain and hardwire path via K5 by jumpering `DRV_K5`; (d) install U1 with firmware — walk the 16-state table (use the self-test strap), verify the interlock rows and the applied-LED behavior; (e) audio at low level: pops, A→B ordering, power-loss pass-through, fuzz-in-Loop-A sanity.
3. **Record:** measured current draw, rail voltages, relay timings — into a build record page.

## 8. Optional remote footswitch / patch panel (single lane)

Two normalled TS jacks (e.g. Switchcraft 12A-style switched-tip) parallel the switch inputs: tip normal contact ties `SW1_N`/`SW2_N` through to the onboard stomps; plugging a remote 2-pin momentary switch (tip = switch, sleeve = GND) breaks the normal and takes over. Unplugged = onboard stomps. Keep jack sleeves on the logic-ground return, not the audio star.

## 9. Drawing notes for the KiCad schematic

- Symbols: standard libs cover everything (`4xxx` no longer needed): MCU_Memory? use the ATmega328P from `MCU_Microchip_ATmega` lib; `Relay:Relay_DPDT`, `Amplifier_Operational:LMC660` (or generic quad), `Transistor_BJT:2N3904`, `Diode:1N4148`/`1N5817`, `Device:R/C/C_Polarized/LED`, `Connector_Audio:AudioJack2`, `Connector:Barrel_Jack`, `Switch:SW_SPST`, `power:GND/+9V/+5V/PWR_FLAG`.
- Power nets: name the protected rail `+9V` (or `+9V_PROT` via a renamed power symbol — one flag on each rail plus GND keeps ERC clean).
- Relay pins are functional EN50005 names (11/21 COM, 12/22 NC, 14/24 NO, A1/A2 coil) — map to the TQ2 datasheet before layout.
- When drawn, I verify (§7.1) before any copper.
