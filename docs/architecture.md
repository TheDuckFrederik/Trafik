# Trafik architecture

## Product family

Trafik routes instrument-level audio through two external pedal loops and, on the multi-lane models, to one selected device lane. “Lane” means one destination/device position, not a stereo channel. All models use mono, unbalanced, ¼-inch TS audio connections.

| Model | Device lanes | Footswitches | Pedal-loop controls |
| --- | ---: | ---: | --- |
| Single Lane | 1 | 2 | Pre-FX and Pre-TO |
| 2 Lane | 2 | 3 | Pre-FX and Pre-TO |
| 4 Lane | 4 | 3 | Pre-FX and Pre-TO |

Trafik can be built with either of two audio-switching implementations, described fully in [Circuit Design Principles](#circuit-design-principles) below: a default, all-mechanical rotary-switch design with no control electronics, or an alternative relay-based design using latching/non-latching electromechanical signal relays driven by fixed-function CMOS logic (momentary footswitches advance hardware counters; relay contacts carry audio). Neither option uses a microcontroller, programmable logic, or firmware. The relay option requires a regulated 9 V DC pedal supply for control and relay coils; the mechanical option requires no power for switching and, at most, a small DC/battery supply for LEDs. The audio signal is always passive and separate from any control power except through switch/relay contacts.

## Panel and connector convention

- **Instrument IN:** instrument signal input.
- **TO 1…N:** output to the selected device’s instrument input. Only the selected lane is connected.
- **FX Return 1…N:** input from the selected device’s FX Send.
- **FX Send 1…N:** output to that device’s FX Return.
- **Loop A Send / Return** and **Loop B Send / Return:** connect the corresponding external pedal chain. Send is an output from Trafik; Return is an input to Trafik.

For each lane, connect the amplifier/device FX Send to Trafik’s matching **FX Return**, and Trafik’s matching **FX Send** to the amplifier/device FX Return. This direction convention is stated from Trafik’s point of view and avoids swapping the send and return cables.

The physical arrangement is Instrument IN on the right; FX Return bank upper left; FX Send bank upper right; Loop B above Loop A on the left; TO bank along the bottom; and footswitches across the center. Loop-state indicators sit near the TO bank and between the FX banks. The large circles shown in the middle are the three footswitches, not jacks or routing nodes.

## Circuit Design Principles

Trafik supports two interchangeable ways to implement the four-state loop selection and (on multi-lane models) the lane selection. Both use only passive/electromechanical parts — neither uses a microcontroller or firmware. Pick one option per unit; do not mix them within the same loop-select or lane-select stage.

- **Option A — Mechanical stepping switch (default/simplest).** A non-shorting (break-before-make), multi-pole, multi-throw rotary switch performs the routing directly with its contacts; the shaft position **is** the state. There is no control power required for switching, and the unit can be built and used with **no DC supply and no battery at all** if the LEDs are omitted or replaced with passive continuity lamps. This is the recommended starting point for a first, all-passive build.
- **Option B — Relay-based (alternative, for quieter switching).** Latching or non-latching DPDT signal relays carry the audio; fixed-function CMOS counters/decoders (not a microcontroller — no programmable logic and no firmware) advance on each footswitch press and drive the relay coils through transistor drivers. This avoids the small amount of switch noise and shaft wear a rotary switch can introduce, at the cost of requiring a regulated 9 V DC supply for the control circuit at all times. The remainder of this document (Signal paths, Footswitch and indicator logic below, and the [build guide](build-guide.md)) describes Option B in full; the variant documents describe Option A in the same level of detail.

Both options share the following common building blocks, reused identically across all three variants:

- **Loop-select stage.** Each Pre-FX/Pre-TO position is a single 4-state insert stage: Off (straight through), Loop A only, Loop B only, or Loop A → Loop B in series. In Option A this is one non-shorting rotary switch per position (minimum 3 poles × 4 throws: one pole carries the incoming signal to the correct next node, one carries Loop A Return onward, one carries Loop B Return onward; a spare deck, if present, can carry LED commons). In Option B it is a bank of DPDT relays selected by the counter/decoder output for that state. See [electronics-reference.md](electronics-reference.md) for the exact switch/relay specification.
- **Jack conventions.** TS mono jacks throughout; Send is an output from Trafik, Return is an input to Trafik; direction is always stated from Trafik's point of view (see above).
- **Grounding scheme.** Use insulated-body or isolated jacks and bond every jack sleeve to a single star ground point together with the switch/relay common return and the enclosure. Do not let jack bodies create a second, parallel ground path through the enclosure metal.
- **LED convention.** One LED per loop-select state (Off, A, B, A→B) and, on 2/4 Lane models, one LED per lane. Colors: **green = Loop A engaged**, **red = Loop B engaged**, **amber/bi-color (green+red together) = Loop A→B**, **unlit = Off**; lane LEDs are a single color (e.g. white or blue) with only the selected lane's LED lit. In Option A, each LED (with its own series resistor) is wired straight to the rotary switch throw contact for its state/lane, so the switch position lights the LED with no decoding logic. In Option B, LEDs are driven from the applied-state decoder outputs (after the loop-allocation interlock), not from the raw counter, so the display always matches the audio path actually in circuit.
- **Power convention.** Option A needs no power for switching; if LEDs are fitted, they can run from either a 9 V battery (isolated, switched or wired through the input jack's switching contact to save battery life) or a external 9 V DC supply. Option B always needs an external, regulated, isolated, center-negative 9 V DC supply (2.1 mm barrel) rated for the measured relay-coil and logic current, because the counters must stay powered to hold state. A build that uses Option A with a battery, or with no LEDs at all, requires no DC jack.

## Signal paths

In the selected lane, the main instrument path is:

`Instrument IN → Pre-FX loop position → selected TO output → device input`

The device FX path is:

`device FX Send → selected FX Return input → Pre-TO loop position → selected FX Send output → device FX Return`

Unselected lane outputs and FX jacks are electrically isolated by relay contacts. The loop selector inserts Loop A, Loop B, both in series (A first, then B), or neither at its assigned position. A loop not selected at either position is bypassed.

Each physical pedal loop can occupy only one signal position at a time. If a loop is requested at both positions, the Pre-FX position has priority and the loop is excluded from Pre-TO; indicators show the applied routing. This interlock prevents one pedal chain from being connected into both paths. It does not combine the device’s instrument and FX paths.

## Footswitch and indicator logic

The two outer momentary footswitches each cycle the loop combination for their position:

| Press since previous state | Selection |
| ---: | --- |
| 1 | Loop A |
| 2 | Loop B |
| 3 | Loop A → Loop B |
| 4 | Off |
| 5 | Loop A, then repeat |

Each position has four state indicators: Off, A, B, and A→B. Only the applied state is lit. The device footswitch on the 2- and 4-lane models advances through available lanes in numerical order. One lane indicator is lit at a time. The single-lane model has no device footswitch; its sole lane remains selected.

**This section describes Option B (relay-based).** The control PCB implements the cycle with debounced momentary inputs and fixed-function CMOS counters; counter outputs drive transistor relay drivers. Power-up reset selects Off for both loop positions and lane 1 on multi-lane models. Switching is break-before-make where contacts can otherwise momentarily join outputs. A regulated, isolated, center-negative 9 V pedal supply is assumed; verify the supply polarity against the finished wiring before connection. For Option A (mechanical, default), the rotary switch position directly is the state — see the "Circuit Design" section in each variant document.

## Shared design and model-specific changes

All variants share the mono signal convention, two external loops, four-state loop selection, relay-switched audio, control power input, loop-state indicators, and basic construction process. Lane count changes the enclosure, TO/FX jack counts, lane-select circuit outputs, lane indicator count, and relay/contact count. Single Lane omits the lane selector and lane indicators.

See the individual model specifications for jack counts, panel layout, control states, estimated bill-of-material totals, and the "Circuit Design" section in each for the full Option A/B wiring detail. See the [electronics reference](electronics-reference.md) for the consolidated, deduplicated part-category specification shared across variants, and the [BOM](bom-PLACEHOLDER.md) for the priced component estimates and alternatives.

## Electrical/build limits

- This is an instrument-level switcher, not a speaker-level or mains-voltage device.
- Use insulated TS jacks or isolate jack grounds from the enclosure consistently; do not create unintended ground paths through multiple jack sleeves.
- Keep audio wiring short, shielded where appropriate, and physically separated from relay coils and counter wiring.
- Use relay contacts rated for low-level audio; do not use the relay coil rating as a proxy for contact suitability.
- No exact enclosure hole coordinates are specified. Verify the actual enclosure, jacks, switches, relay board and cable bend radii with a full-size layout before drilling.
- Validate each assembled unit with the continuity and signal-path checks in the [build guide](build-guide.md).
