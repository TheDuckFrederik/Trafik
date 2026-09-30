# Trafik architecture

## Product family

Trafik is a passive, mono instrument-level audio router. Its audio is switched by electromechanical signal relays; fixed-function CMOS circuitry handles momentary footswitches, state decoding, and relay drivers. There is no firmware, microcontroller, programmable logic, or software-controlled audio switching.

| Model | Device lanes | Audio jacks | Footswitches |
| --- | ---: | ---: | ---: |
| Single Lane Trafik | 1 | 8 | 2 |
| 2 Lane Trafik | 2 | 11 | 3 |
| 4 Lane Trafik | 4 | 17 | 3 |

The Single Lane model omits only the device/lane selector. It has the same two loop-position controls and loop states as the other models.

## Signal and panel conventions

All audio connectors are mono, unbalanced ¼-inch TS. “Send” and “Return” are named from Trafik's point of view:

| Label | Direction |
| --- | --- |
| Instrument IN | Input from instrument |
| TO 1…N | Output to the selected device's instrument input |
| FX Return 1…N | Input from the selected device's FX Send |
| FX Send 1…N | Output to the selected device's FX Return |
| Loop A/B Send | Output to the external pedal chain |
| Loop A/B Return | Input from the external pedal chain |

Connect a device's FX Send to the matching Trafik FX Return, and Trafik's matching FX Send to the device FX Return.

Physical layout names are fixed: Instrument IN at the right, FX Return bank at the upper left, FX Send bank at the upper right, Pedal Loop B above Pedal Loop A on the left, and TO outputs at the bottom. The large circles in the layout are footswitches, never jacks or routing nodes. On the multi-lane models, the device selector is the center footswitch; Pre-FX and Pre-TO are the outer footswitches.

## Signal paths

For selected lane `n`:

```text
Instrument IN → Pre-FX loop position → TO n → device n input
device n FX Send → FX Return n → Pre-TO loop position → FX Send n → device n FX Return
```

Only the selected lane's TO output and corresponding FX pair connect to the audio paths. Unselected lane tips are isolated. The audio circuit carries instrument-level signals only; it is not suitable for speaker outputs or mains voltage.

## Loop states and assignment

Each position has its own momentary footswitch and requested state. Presses cycle independently:

| State code | Requested loop routing |
| ---: | --- |
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

This gives the following applied Pre-TO state for every request pair; Pre-FX always keeps its requested state:

| Pre-FX request ↓ / Pre-TO request → | Off | A | B | A→B |
| --- | --- | --- | --- | --- |
| Off | Off | A | B | A→B |
| A | Off | Off | B | B |
| B | Off | A | Off | A |
| A→B | Off | Off | Off | Off |

The applied LED state may therefore differ from the Pre-TO requested state during a conflict. The control/relay design must implement this allocation before driving both the audio contacts and state LEDs. A loop request at Pre-TO must not connect that loop's send/return if the same loop is applied at Pre-FX.

## Hardware control and switching

The selected implementation is momentary normally-open footswitches, fixed-function CMOS counters/decoders, transistor relay drivers, and electromechanical signal relays. A practical control block uses a Schmitt-trigger debounce stage (for example, CD40106B at a supply voltage within its data-sheet limits), one four-state counter per loop position, and a modulo-N counter for lane selection (for example, CD4017B-family logic). Counter outputs are one-hot; power-on reset selects Off for both loop positions and lane 1. Use the actual IC datasheets to design reset, clock conditioning, decoupling, and any logic-level translation.

The relay coil voltage and contact configuration are selected from the actual relay datasheet. Each coil needs a driver rated for its pull-in current and an appropriately oriented flyback clamp. Relay contacts, not CMOS pins, carry audio. Use break-before-make switching for lane changes. The unpowered contact state must pass the instrument path to lane 1, bypass both loop positions, and leave all other lane outputs isolated.

The state-control sequence is not itself an audio schematic. The project currently has no verified, manufacturer-specific relay contact/netlist diagram or PCB drawing. A builder must complete and check a contact-level wiring matrix for the exact relay model before fabricating a control PCB or connecting audio. In particular, verify that the chosen contact groups implement the shared-loop allocation above and the stated power-loss state; do not infer terminal numbers from a generic DPDT drawing. The functional requirements and checkout criteria below are the acceptance specification for that work.

### State and indication

Each loop position has four separate labeled LEDs: Off (white), A (green), B (red), and A→B (amber or bi-color). Exactly one LED per position is on and follows the **applied** routing after allocation. Multi-lane models have one single-color lane LED per lane, with only the selected lane lit. Each LED needs an individual series resistor sized from its supply and forward-voltage datasheet:

`R = (Vrail − Vf) / Iled`

For example, 9 V, a 2 V LED drop, and 5 mA gives 1.4 kΩ; use a suitable standard value such as 1.5 kΩ and verify brightness/current. Do not assume every LED has the same forward voltage.

## Power, grounding, and wiring

- Use a regulated, isolated 9 V DC pedal supply, 2.1 mm center-negative connector, with current capacity above the measured worst-case draw. The current draw depends on the chosen relay coil resistance and the number energized simultaneously; calculate it from the relay datasheet, then measure the completed build. Do not select a supply based on an unverified generic current figure.
- Add reverse-polarity protection, local supply decoupling at each logic IC, and coil suppression appropriate to the driver circuit. Confirm that the protection device's voltage drop still leaves adequate relay pull-in voltage.
- Prefer isolated TS jacks. Bond jack sleeves together at a single audio-ground point; bond the enclosure to that point once. Return logic and coil current to the supply-ground star separately from the audio-return wiring, joining them at the defined star point. Do not use the enclosure as the normal audio-current return.
- Keep high-impedance audio runs short and separate from clocks, LED leads, and coil wiring. Use shielded cable for long/noisy runs and connect its shield at the planned audio-ground end to avoid multiple shield bonds.
- Secure wires against sharp edges and moving footswitch parts. Insulate unused relay contacts and exposed terminals.

## Fabrication assumptions and release gate

Enclosure sizes in the variant pages are starting points, not guaranteed fits. There are no dimensioned drilling templates, verified PCB files, or tested prototype measurements in this repository. Before fabrication, a builder must confirm panel spacing and lid clearance using the selected parts, then create a pin-numbered schematic/contact matrix from their datasheets. Do not treat the functional block and state tables as a substitute for those manufacturer-specific drawings.

## Verification

Use the unpowered continuity, powered state, lane-isolation, duplicate-loop allocation, LED, and power-loss checks in the [build guide](build-guide.md). Keep an instrument-level test source and amplifier at low volume until all switch states pass.
