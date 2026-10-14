# Trafik electronics reference

This file consolidates the distinct electronics part categories used across the Trafik family, deduplicated, so a builder can search for and source parts (from any manufacturer) and compile their own bill of materials. It covers **what** to buy, not **where** or **for how much** — for priced examples and retailer alternatives, see the [BOM](bom-prototype.md). For the circuit-level context, see [architecture.md](architecture.md) and the variant pages ([Single Lane](single-lane-prototype.md), [Multi-Lane](multi-lane-prototype.md)).

The reference reflects the **active platform**: relay-switched audio, microcontroller control, three unity buffers, passive signal path from the instrument jack to the first loop return. (The archived passive/CMOS Revision A parts are listed in [single-lane-circuit.md](single-lane-circuit.md).)

## Audio switching

### Signal relay

DPDT (or the contact form the lane design requires), non-latching, sealed preferred, contacts rated for low-level/instrument signals (gold or clad contacts — not a mains/power relay). Coil voltage per form factor: 9 V coil on the stompbox rail; rack units may use logic-rail coils with appropriate drivers. Search for: *"DPDT signal relay, low level contacts, non-latching"*.

| Category | Single Lane | Multi-Lane (N = 2–8) |
| --- | ---: | ---: |
| DPDT signal relay | 5 (4 loop inserts + 1 buffer-bypass) | 5 (4 loop inserts + 1 buffer-bypass) + lane-selection groups scaling with N |

Loop-insert relays are fixed at 4 for the whole family (two loops × two positions), plus one instrument-output buffer-bypass relay; only the lane-selection relay groups grow with lane count.

### Relay drivers and suppression

One driver per coil: NPN transistor or logic-level MOSFET rated above coil pull-in current, with a correctly oriented flyback diode directly across each coil. Base/gate resistors sized for hard saturation at the controller's logic level.

## Control

### Microcontroller

Any mainstream MCU family with enough GPIO for the variant (footswitch inputs, relay driver outputs, LED outputs, lane selection; I²C/UART spare for the MIDI/preset roadmap). Required firmware behaviors — debounce, the four-state cycle per position, Pre-FX-priority loop allocation before relay/LED drive, power-on state (both positions Off, lane 1), watchdog, and fail-safe de-energize of all relays on fault — are specified in [architecture.md](architecture.md#hardware-control-and-switching). A built-in self-test mode walking the full state × relay truth table is required for made-to-order units.

### Momentary footswitch (2-pin)

Rugged, normally-open SPST (2-pin), non-latching soft-touch stomp switch; contacts carry no audio. Search for: *"momentary footswitch / non-latching soft touch stomp switch"*. The click-and-return test distinguishes it from a latching switch.

| Category | Single Lane | Multi-Lane |
| --- | ---: | ---: |
| Momentary 2-pin footswitch | 2 | 3 |

## Buffers (3 per unit, all variants)

Unity-gain, non-inverting op-amp stages per the buffer map in [architecture.md](architecture.md#audio-architecture-passive-front-buffered-rest): Loop A Return, Loop B Return, instrument output ahead of the lane selector. One quad op-amp covers all three plus a spare (tuner-out candidate).

- Stompbox (single 9 V rail): rail-to-rail input/output op-amp, or validate TL072-class headroom explicitly.
- Rack (±15 V rails): TL072-class is fine.
- Each stage: 1 MΩ input bias, AC coupling in/out with bleed resistors (pop-free relay transitions).

Search for: *"rail-to-rail op-amp, single supply, DIP/SOIC"* and *"TL072"*.

## Shared parts

### Audio jacks

Mono ¼-inch (6.35 mm) TS jack, isolated/insulated-body preferred to simplify sleeve-ground control, solder-lug terminals. Jack count = 3N+5 per unit (IN, TO ×N, FX Send ×N, FX Return ×N, Loop A/B Send/Return). Search for: *"1/4 inch mono TS jack, insulated/isolated, panel mount, solder lug"*.

| Category | Single Lane | 2 Lane | 4 Lane | 8 Lane |
| --- | ---: | ---: | ---: | ---: |
| Audio jack | 8 | 11 | 17 | 29 |

### Indicator LEDs and resistors

3 mm or 5 mm diffused LED per the family convention: white = Off, green = Loop A, red = Loop B, amber = A→B (four per loop position), plus N single-color lane LEDs on multi-lane units. Series resistor per LED: **R = (Vsupply − Vf) / I**; do not assume equal forward voltages across colors.

| Category | Single Lane | Multi-Lane |
| --- | ---: | ---: |
| Loop-state LED | 8 | 8 |
| Lane LED | 0 | N |

### Enclosures / chassis

- Stompbox: diecast aluminum 1590-series, size to jack/switch count (1590B-class for Single Lane; larger for 2–4 lanes).
- Rack: 1U steel/aluminum chassis with rear-panel audio and internal linear supply (IEC inlet, ±15 V audio rails, logic rail).

### Power input

- Stompbox: 2.1 mm DC barrel jack, center-negative, panel-mount; regulated isolated 9 V supply rated above measured worst-case draw.
- Rack: IEC inlet with fuse and switch per standard 1U practice.

### Wire

24–26 AWG stranded hook-up wire for control/LED wiring; shielded, low-capacitance instrument cable for the **passive front-end** audio runs (Instrument IN to first loop return) and other noise-sensitive stretches. Keep the passive front-end runs short and away from clocks, LEDs, and coil wiring.

### Mounting hardware

Jack nuts and insulating shoulder washers where metal-body jacks are used, switch nuts, PCB standoffs, rubber feet, strain relief/grommets, heat-shrink.

Substitutions: any relay, jack, LED, op-amp, MCU, or switch that meets the stated contact form, rating, logic requirements, and panel form factor is acceptable regardless of manufacturer. Verify every pinout against the exact datasheet before ordering or applying power.
