# Trafik architecture

## Product family

Trafik routes instrument-level audio through two external pedal loops and, on the multi-lane models, to one selected device lane. “Lane” means one destination/device position, not a stereo channel. All models use mono, unbalanced, ¼-inch TS audio connections.

| Model | Device lanes | Footswitches | Pedal-loop controls |
| --- | ---: | ---: | --- |
| Single Lane | 1 | 2 | Pre-FX and Pre-TO |
| 2 Lane | 2 | 3 | Pre-FX and Pre-TO |
| 4 Lane | 4 | 3 | Pre-FX and Pre-TO |

The design uses latching electromechanical signal relays driven by fixed-function CMOS logic. Momentary footswitches advance hardware counters; relay contacts carry audio. It requires a regulated 9 V DC pedal supply for control and relay coils, but has no digital audio processing or firmware. The audio signal is passive and is disconnected from the power circuit except through relay contacts.

## Panel and connector convention

- **Instrument IN:** instrument signal input.
- **TO 1…N:** output to the selected device’s instrument input. Only the selected lane is connected.
- **FX Return 1…N:** input from the selected device’s FX Send.
- **FX Send 1…N:** output to that device’s FX Return.
- **Loop A Send / Return** and **Loop B Send / Return:** connect the corresponding external pedal chain. Send is an output from Trafik; Return is an input to Trafik.

For each lane, connect the amplifier/device FX Send to Trafik’s matching **FX Return**, and Trafik’s matching **FX Send** to the amplifier/device FX Return. This direction convention is stated from Trafik’s point of view and avoids swapping the send and return cables.

The physical arrangement is Instrument IN on the right; FX Return bank upper left; FX Send bank upper right; Loop B above Loop A on the left; TO bank along the bottom; and footswitches across the center. Loop-state indicators sit near the TO bank and between the FX banks. The large circles shown in the middle are the three footswitches, not jacks or routing nodes.

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

The control PCB implements the cycle with debounced momentary inputs and fixed-function CMOS counters; counter outputs drive transistor relay drivers. Power-up reset selects Off for both loop positions and lane 1 on multi-lane models. Switching is break-before-make where contacts can otherwise momentarily join outputs. A regulated, isolated, center-negative 9 V pedal supply is assumed; verify the supply polarity against the finished wiring before connection.

## Shared design and model-specific changes

All variants share the mono signal convention, two external loops, four-state loop selection, relay-switched audio, control power input, loop-state indicators, and basic construction process. Lane count changes the enclosure, TO/FX jack counts, lane-select circuit outputs, lane indicator count, and relay/contact count. Single Lane omits the lane selector and lane indicators.

See the individual model specifications for jack counts, panel layout, control states, and estimated bill-of-material totals. See the [BOM](bom-PLACEHOLDER.md) for the component estimates and alternatives.

## Electrical/build limits

- This is an instrument-level switcher, not a speaker-level or mains-voltage device.
- Use insulated TS jacks or isolate jack grounds from the enclosure consistently; do not create unintended ground paths through multiple jack sleeves.
- Keep audio wiring short, shielded where appropriate, and physically separated from relay coils and counter wiring.
- Use relay contacts rated for low-level audio; do not use the relay coil rating as a proxy for contact suitability.
- No exact enclosure hole coordinates are specified. Verify the actual enclosure, jacks, switches, relay board and cable bend radii with a full-size layout before drilling.
- Validate each assembled unit with the continuity and signal-path checks in the [build guide](build-guide.md).
