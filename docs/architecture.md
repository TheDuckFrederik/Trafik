# Trafik architecture

> **Platform decision (2026-10-13):** Trafik is an **active, relay-switched, firmware-controlled** platform. The audio path is passive from the instrument jack to the first loop return and buffered thereafter; switching is electromechanical relays; control is microcontroller-based. The earlier fully-passive, fixed-function-CMOS design (no firmware) is retained as **Revision A** in [single-lane-circuit.md](single-lane-circuit.md) — a verified circuit design kept as the bench reference and passive fallback. This page specifies the active platform; its contact-level circuit has **not** been designed yet.

## Product family

Trafik is one platform sold as a lane count in one of two form factors, made to order:

| Parameter | Value |
| --- | --- |
| Device lanes | 1–8, made to order (custom lane counts within 1–8 on request) |
| Form factors | Stompbox (floor, 1590-class, 9 V pedal supply) and rack (1U chassis, internal linear supply, ±15 V audio rails) |
| Form-factor guidance | Stompbox suits 1–4 lanes by panel density; rack suits any count 1–8 |
| Footswitches | Momentary normally-open SPST (2-pin), one per loop position, plus device/lane control where applicable |
| Pedal loops | Always 2 (Loop A, Loop B), shared between the two loop positions |

Mono ¼-inch TS throughout, instrument-level audio only. Never connect speaker-level or mains signals.

## Signal and panel conventions

All audio connectors are mono, unbalanced ¼-inch TS. "Send" and "Return" are named from Trafik's point of view:

| Label | Direction |
| --- | --- |
| Instrument IN | Input from instrument |
| TO 1…N | Output to the selected device's instrument input |
| FX Return 1…N | Input from the selected device's FX Send |
| FX Send 1…N | Output to the selected device's FX Return |
| Loop A/B Send | Output to the external pedal chain |
| Loop A/B Return | Input from the external pedal chain |

Physical layout names are fixed: Instrument IN at the right, FX Return bank at the upper left, FX Send bank at the upper right, Pedal Loop B above Pedal Loop A on the left, and TO outputs at the bottom (stompbox) or rear panel (rack). The large circles in the stompbox layout are footswitches, never jacks or routing nodes.

## Signal paths

For selected lane `n`:

```text
Instrument IN → Pre-FX loop position → TO n → device n input
device n FX Send → FX Return n → Pre-TO loop position → FX Send n → device n FX Return
```

Only the selected lane's TO output and corresponding FX pair connect to the audio paths. Unselected lane tips are isolated.

## Audio architecture: passive front, buffered rest

The platform rule:

> **Passive from the instrument jack to the first loop return; buffered from every loop return onward; instrument-chain output buffered ahead of the lane selector; FX Send output passive (line-source by design).**

The buffer map is fixed at **three unity buffers regardless of lane count**:

| # | Location | Purpose |
| --- | --- | --- |
| 1 | Loop A Return (inside the unit) | Drives the Loop A → Loop B feed and everything downstream when Loop A is active; serves both chains because loops are shared |
| 2 | Loop B Return (inside the unit) | Drives the chain output whenever Loop B is last; serves both chains |
| 3 | Instrument chain output, before the lane (TO) selector | Drives the long output cable in the all-loops-off state; one stage feeds every TO port |

**The fuzz rule (deliberate constraint):** nothing upstream of a loop return is ever buffered. A fuzz (or any impedance-interacting pedal) placed first in Loop A sees the instrument's passive signal path, exactly as if wired directly, modulo relay contact resistance. Loop B's first pedal sees a buffered feed whenever Loop A precedes it — place impedance-critical pedals in Loop A.

Rationale per location:

- **Loop Returns buffered:** in the rack format, the loop send/return runs span rack to pedalboard; a return buffer makes every downstream stretch low-impedance and covers the Loop A → Loop B round trip.
- **Instrument output buffered:** the all-loops-off state is the only one where the raw instrument signal would otherwise drive a long output cable.
- **FX Send output unbuffered:** the FX chain's source is always either the device's FX Send (a line-level output) or a loop-return buffer, so it is never high-impedance. A buffer there is redundant.

Buffer stage spec: unity-gain, non-inverting, 1 MΩ input bias, AC-coupled in and out with bleed resistors so relay transitions stay pop-free; on the rack's ±15 V rails, headroom is a non-issue. On the stompbox format's single 9 V rail, select a rail-to-rail-input/output op-amp or validate headroom explicitly — the TL072-on-9 V limitation is documented in the Revision A prototype notes. The three stages fit one quad op-amp, leaving a spare amplifier (candidate: always-on tuner feed, taken after the loop positions and before buffer 3).

## Loop states and assignment

Each loop position has its own momentary footswitch and requested state. Presses cycle independently:

| State code | Requested loop routing |
| --- | ---: |
| 0 | Off; straight through |
| 1 | Loop A |
| 2 | Loop B |
| 3 | Loop A, then Loop B in series |

The next press returns to Off. Loop A is always first when both are inserted at one position.

There is one physical send/return pair for each pedal loop. A loop cannot be connected to both signal paths at once. To resolve simultaneous requests deterministically, Pre-FX has priority for each loop: the Pre-FX request is applied; Pre-TO applies only loops not already assigned to Pre-FX. In terms of request bits `A1`, `B1` (Pre-FX) and `A2`, `B2` (Pre-TO):

```text
Applied Pre-FX: A1, B1
Applied Pre-TO: A2 AND NOT A1, B2 AND NOT B1
```

The applied LED state may therefore differ from the Pre-TO requested state during a conflict. The control firmware must implement this allocation before driving both the relay contacts and state LEDs. A loop request at Pre-TO must not connect that loop's send/return if the same loop is applied at Pre-FX.

## Hardware control and switching

The selected implementation is **momentary normally-open SPST (2-pin) footswitches**, a **microcontroller** for debounce, state, and the loop-allocation logic, transistor relay drivers, and electromechanical signal relays.

Requirements on the control design:

- One clean count per press (software debounce); each position cycles Off → A → B → A→B → Off independently.
- Power-on state: both loop positions Off, lane 1 selected.
- The Pre-FX-priority allocation is enforced before relays and LEDs, per the equations above.
- **Fail-safe behavior:** on loss of power or firmware fault, all relays de-energize to a state that passes the instrument path straight through to lane 1's TO output, bypasses both loop positions, and leaves all other lane outputs isolated. Because the input-to-first-loop-return path is passive, audio continues to pass with DC removed. A watchdog and a defined boot/recovery state are required so a firmware hang cannot hold a relay mid-state indefinitely.
- Relay coils: drivers rated for pull-in current with correctly oriented flyback clamps; coil voltage per form factor (9 V stompbox rail / logic-rail control in the rack with appropriate drivers).
- Relay contacts, not logic pins, carry audio. Break-before-make switching for lane changes. Relays with low-level (gold/clad) contacts suitable for instrument signals.
- Lane count is a build parameter, not a redesign: the firmware and driver hardware are lane-count agnostic; lanes 2–N add relay groups and jacks only. A built-in self-test mode that walks the full state × relay truth table is required for made-to-order units.
- MIDI and preset storage are platform roadmap features (rack format first); they must not compromise the fail-safe requirements above.

The Revision A CMOS implementation ([single-lane-circuit.md](single-lane-circuit.md)) demonstrates the equivalent fixed-function logic for the Single Lane and remains the reference for the allocation truth table and relay contact matrix.

## Power, grounding, and wiring

- **Stompbox format:** regulated, isolated 9 V DC pedal supply, 2.1 mm center-negative, capacity above measured worst-case draw (relays + logic + three buffers). Reverse-polarity protection and local decoupling at every IC.
- **Rack format:** internal linear supply with IEC inlet, generating logic power and **±15 V audio rails** for the buffer stages. Serviceability and fusing per standard 1U practice.
- Prefer isolated TS jacks. Bond jack sleeves together at a single audio-ground point; bond the chassis to that point once. Return logic, coil, and supply currents to the supply-ground star separately from the audio-return wiring, joining at the defined star point. Do not use the chassis as the normal audio-current return.
- Keep the high-impedance passive front-end runs (Instrument IN to first loop return) short and away from clocks, LED leads, and coil wiring. Use shielded cable for long or noisy runs, shield bonded at the planned audio-ground end only.
- Secure wires against sharp edges and moving footswitch parts. Insulate unused relay contacts and exposed terminals.

## Fabrication assumptions and release gate

Enclosure sizes are starting points, not guaranteed fits. There is no verified PCB drawing, dimensioned drill template, or built unit for the active platform. Before fabrication, a builder must: confirm panel spacing and lid clearance with the selected parts; produce a pin-numbered schematic/contact matrix from the exact relay and connector datasheets; and pass the checks in the [build guide](build-guide.md). Do not treat the functional descriptions on these pages as a substitute for manufacturer-specific drawings.

## Verification

Use the unpowered continuity, powered state, lane-isolation, duplicate-loop allocation, LED, and power-loss checks in the [build guide](build-guide.md). For every made-to-order lane count, run the full state × relay truth table (the firmware self-test mode may generate this). Keep an instrument-level test source and amplifier at low volume until all switch states pass.
