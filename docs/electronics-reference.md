# Trafik electronics reference

This file consolidates every distinct electronics part category used across the three Trafik variants, deduplicated, so a builder can search for and source parts (from any manufacturer) and compile their own bill of materials. It covers **what** to buy, not **where** or **for how much** — for priced examples and retailer alternatives, see the [BOM](bom-PLACEHOLDER.md). For the circuit-level context each part is used in, see [architecture.md](architecture.md#circuit-design-principles) and the "Circuit Design" section of each variant document ([Single Lane](single-lane-PLACEHOLDER.md#circuit-design), [2 Lane](two-lane-PLACEHOLDER.md#circuit-design), [4 Lane](four-lane-PLACEHOLDER.md#circuit-design)).

Two implementation options are described side by side: **Option A** (mechanical stepping switch, default/simplest, no control power required) and **Option B** (relay-based, alternative, for quieter switching). Build one option per unit; the quantities below assume a single, consistent choice.

## Option A — mechanical stepping switch parts

### Loop-select switch

4-position, non-shorting (break-before-make) rotary switch, minimum **3 poles × 4 throws (3P4T)**; a standard 4P4T/4-pole part with one deck left unused (or repurposed for LED commons) also works. Search for: *"4 position 3PDT (or 4PDT) non-shorting rotary switch, panel mount"*. One is needed per loop-select position (Pre-FX and Pre-TO).

**Foot actuation:** a plain panel-mount rotary switch needs a knob and is turned by hand, not by foot. For true foot-operated stepping, search for a **rotary "stepping"/ratchet footswitch mechanism** (a spring-loaded pedal that advances a ratchet-and-pawl or Geneva-style index one detent per press, coupled to the rotary switch shaft), sometimes sold as a "tap-to-advance" or "channel-stepper" mechanism in guitar-amp channel-switching hardware. As a simpler substitution, a panel-mount rotary switch with an oversized "chicken head" knob operated by hand (not foot) is acceptable for a first build; note this substitution changes the control from a footswitch to a hand-turned knob.

| Category | Single Lane | 2 Lane | 4 Lane |
| --- | ---: | ---: | ---: |
| Loop-select rotary switch (4-position, 3P4T) | 2 | 2 | 2 |

### Device (lane) select switch

- **2 Lane:** 2-position, minimum 3-pole, non-shorting switch (3P2T). Since only 2 throws are needed, this can be a 3PDT footswitch/toggle instead of a rotary part. Search for: *"3PDT toggle switch, on-on"* or *"2 position 3-pole rotary switch"*.
- **4 Lane:** 4-position, minimum 3-pole, non-shorting rotary switch (3P4T) — the same part class as the loop-select switch above, wired for lane routing instead of loop routing. Search for the same term as the loop-select switch.
- **Single Lane:** not used (no device selector).

| Category | Single Lane | 2 Lane | 4 Lane |
| --- | ---: | ---: | ---: |
| Device-select switch | 0 | 1 (3PDT or 3P2T) | 1 (3P4T rotary) |

### Substitution notes (Option A)

- A rotary switch with more poles/throws than the minimum (e.g. a 4P4T or 4P5T part) can always be used with unused decks/positions left disconnected or dedicated to LEDs.
- Shorting (make-before-break) rotary switches are **not** acceptable for the loop-select or device-select stage: they would briefly tie two audio paths together while stepping. Confirm "non-shorting"/"break-before-make" in the datasheet before buying.
- If a true foot-actuated stepping mechanism cannot be sourced, a hand-turned rotary knob is an acceptable substitute for a hand-built, non-foot-operated unit.

## Option B — relay-based parts (alternative)

### Signal relay

DPDT, non-latching, 9 V coil, contact rating suitable for guitar/instrument-level signal (low current, low-level switching contacts — not a mains/power relay), sealed contacts preferred to resist dust/oxidation on the small signal currents involved. Search for: *"DPDT signal relay, 9V coil, low level contacts"*.

### Control/driver components

Fixed-function CMOS counter/decoder ICs (not a microcontroller — no programmable firmware), debounce components (RC network or a dedicated debounce IC), transistor relay drivers, flyback/coil-suppression diodes, and support passives (resistors, capacitors). Search for: *"CMOS 4017/4018-class decade/Johnson counter"* and *"NPN small-signal transistor, relay driver"* as representative families; any equivalent fixed-function counter/decoder family is acceptable.

| Category | Single Lane | 2 Lane | 4 Lane |
| --- | ---: | ---: | ---: |
| DPDT signal relay | 10 | 14 | 22 |
| Control/driver PCB (or hand-wired perfboard) | 1 | 1 | 1 |

Relay counts above are a planning allowance for the bypass/insert/interlock functions described in each variant's "Hardware implementation (Option B)" section, not a pin-for-pin netlist; verify against the exact relay's contact form before ordering.

### Momentary footswitch (Option B only)

Rugged stomp switch, normally-open SPST contact, no mechanical latching (it only pulses the counter — it does not carry audio). Search for: *"momentary SPST footswitch, normally open, stomp switch"*.

| Category | Single Lane | 2 Lane | 4 Lane |
| --- | ---: | ---: | ---: |
| Momentary footswitch | 2 | 3 | 3 |

## Shared parts (both options)

### Audio jacks

Mono ¼-inch (6.35 mm) TS jack, isolated/insulated-body type preferred to simplify sleeve-ground control, solder-lug terminals. Search for: *"1/4 inch mono TS jack, insulated/isolated, panel mount, solder lug"*.

| Category | Single Lane | 2 Lane | 4 Lane |
| --- | ---: | ---: | ---: |
| Audio jack (Instrument IN, TO, FX Send/Return, Loop A/B Send/Return) | 8 | 11 | 17 |

### Indicator LEDs and resistors

3 mm or 5 mm diffused LED, colors per the [LED convention](architecture.md#circuit-design-principles): green = Loop A, red = Loop B, amber/bi-color = Loop A→B, unlit = Off; a single color (white/blue) for lane indicators. Pair each LED with a series resistor sized for the chosen supply voltage: **R = (Vsupply − Vf) / I**. Example at 9 V, 10 mA, Vf ≈ 2 V (red/green): R ≈ 700 Ω → use 680 Ω (nearest standard value). Search for: *"3mm/5mm diffused LED"* and *"1/4W carbon film or metal film resistor"*.

| Category | Single Lane | 2 Lane | 4 Lane |
| --- | ---: | ---: | ---: |
| Indicator LED | 8 | 10 | 12 |
| Series resistor (one per LED) | 8 | 10 | 12 |

### Enclosure

Diecast aluminum stompbox-style enclosure, sized to the jack/switch count for the variant. Search for: *"1590-series diecast aluminum enclosure"* (or equivalent), choosing a larger footprint as jack/switch count increases.

| Category | Single Lane | 2 Lane | 4 Lane |
| --- | ---: | ---: | ---: |
| Enclosure | 1 (≈1590B-class) | 1 (≈1590DD-class) | 1 (≈1590XX-class) |

### Wire

24–26 AWG stranded, insulated hook-up wire for control/LED wiring; shielded, low-capacitance instrument cable/wire for audio signal runs, especially longer runs or where the loop-select stage is far from its jacks. Search for: *"24 AWG stranded hook-up wire"* and *"shielded instrument cable, low capacitance"*.

### Power (LED supply, if fitted; required for Option B)

- **9 V battery:** standard 9 V alkaline/zinc-carbon battery with a snap connector; used with Option A when only battery power for LEDs is wanted.
- **9 V DC power jack:** 2.1 mm barrel, center-negative, panel-mount. Required for Option B (control power); optional for Option A (LED power only). Search for: *"2.1mm DC barrel jack, panel mount, center negative"*.

| Category | Single Lane | 2 Lane | 4 Lane |
| --- | ---: | ---: | ---: |
| DC power jack (optional on Option A; required on Option B) | 0–1 | 0–1 | 0–1 |

### Knobs, switch caps, and mounting hardware

- Knob/chicken-head cap for each panel-mount rotary switch (Option A) or footswitch cap/boot for each stomp switch (Option B).
- Jack nuts, washers (including insulating shoulder washers where a metal-body jack is used), switch mounting nuts, rubber feet, cable strain relief/grommet for the DC lead, and heat-shrink tubing for solder joints.

Substitutions: any rotary switch, jack, LED, resistor, or enclosure that meets the stated pole/throw count, contact rating, LED color, and physical panel-mount form factor is acceptable regardless of manufacturer. See the [BOM](bom-PLACEHOLDER.md) for one internally-consistent, priced example configuration (Option B) and retailer alternatives.
