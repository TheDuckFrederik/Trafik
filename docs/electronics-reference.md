# Trafik electronics reference

This is the shared parts-selection reference for all three variants. It describes one hardware-only approach: momentary footswitches, fixed-function CMOS counters/logic, transistor coil drivers, and electromechanical audio relays. There is no Arduino, microcontroller, firmware, or software-controlled audio switching.

This reference is not a completed pin-level schematic. The relay contact matrix for the shared pedal-loop allocation and each model's lane routes must be completed for the actual relay selected before fabrication. See the [architecture](architecture.md#fabrication-assumptions-and-release-gate), [variant specifications](single-lane-PLACEHOLDER.md), and [build guide](build-guide.md).

## Common and variant quantities

| Part category | Single Lane | 2 Lane | 4 Lane | Notes |
| --- | ---: | ---: | ---: | --- |
| Isolated mono ¼-inch TS jack | 8 | 11 | 17 | One IN, N TO, 2N FX jacks, 4 loop jacks |
| Momentary normally-open footswitch | 2 | 3 | 3 | Pre-FX, Pre-TO; add Device on multi-lane models |
| Four-state loop counter channel | 2 | 2 | 2 | Off, A, B, A→B |
| Modulo lane counter channel | 0 | 1 | 1 | N=2 or N=4; lane 1 at reset |
| State/lane indicator LED | 8 | 10 | 12 | Four per loop position plus one per lane |
| LED series resistor | 8 | 10 | 12 | One per LED, calculate for chosen LED/current |
| Relay contacts/relays | Design-specific | Design-specific | Design-specific | Final quantity depends on verified contact matrix |
| Enclosure | 1 | 1 | 1 | 1590B/DD/XX-class are fit-check starting points |
| Regulated 9 V supply input | 1 | 1 | 1 | Center-negative 2.1 mm jack; supply external |

The relay totals previously shown in the BOM are budget allowances only. Do not order from those numbers until the actual contact-level design is complete.

## Control logic

- **Switch inputs:** sealed or rugged momentary, normally-open, SPST footswitch contacts. They carry only the logic input signal and do not carry audio.
- **Debounce/edge shaping:** CD40106B Schmitt-trigger inverter, or an equivalent CMOS Schmitt device rated for the selected logic supply. An RC network can suppress bounce; select the time constant from the switch characteristics and verify one count per press. Do not use a 74HC14 directly on 9 V; the HC family has a lower maximum supply voltage.
- **Loop counters:** one CD4017B decade counter (or equivalent fixed-function counter/decoder) per position. Use Q0=Off, Q1=A, Q2=B, Q3=A→B; route Q4 to RESET so the next stable state is Q0. Add a defined power-on reset to Q0.
- **Lane counter:** a separate counter channel with a defined startup reset to lane 1. For 2 lanes, reset on the first unused count after lane 2; for 4 lanes, reset after lane 4. Confirm the selected counter's reset polarity and propagation behavior from its datasheet.
- **Allocation logic:** derive applied Pre-TO A/B requests by gating each Pre-TO request with the inverse of the corresponding Pre-FX request. Drive LEDs from the applied state. The relay network must ensure that a physical loop's send and return connect to no more than one position.
- **Logic/relay separation:** CMOS outputs must not drive coils directly. Use one suitably rated transistor driver per required coil, sized from measured/calculated coil current. Add a flyback diode across each DC coil (cathode to +9 V) unless the chosen driver/relay suppression scheme specifies otherwise. Include base/gate resistors and defined off-state bias.

The counter and allocation equations define requested/applied states, not every relay contact connection. Create a truth table from all possible pairs of loop requests and translate it to the chosen relay's actual contact diagram before PCB or perfboard wiring. See the priority equations in [architecture.md](architecture.md#loop-states-and-assignment).

## Audio relay selection

Choose sealed, non-latching signal relays with DC coils and contacts suitable for low-level instrument audio. A nominal 9 V coil can be used on the protected 9 V rail only if the data sheet permits the actual coil voltage over the supply tolerance. Select the contact form (DPDT or larger as needed) from the completed switching matrix; do not assume every function needs a DPDT relay or use the current BOM allowance as a design count.

For each candidate relay, confirm:

1. Coil voltage, resistance/current, pickup/dropout voltage, and allowable temperature.
2. Contact arrangement and terminal numbering from the manufacturer's drawing.
3. Contact material and suitability for low-level/dry-circuit signal switching.
4. Contact resistance, insulation/isolation, and make/break behavior.
5. Package, solder footprint, and whether the coil can be safely driven by the selected transistor.

Do not rely on a generic relay module's silkscreen or an unrelated manufacturer's pinout. Verify the coil and every contact pole with the datasheet and a multimeter before wiring audio.

## LEDs, resistors, and power

Use four distinct state indicators per loop position: Off (white), A (green), B (red), A→B (amber or bi-color). Use one white/blue LED per device lane. Exactly one state LED per position and one lane LED on multi-lane models should be lit.

Calculate each resistor using `R = (Vrail − Vf) / Iled`. For example, with 9 V, `Vf=2 V`, and 5 mA, `R=1.4 kΩ`; 1.5 kΩ is a reasonable standard starting value. Verify the actual LED forward voltage, resistor power (`P=I²R`), brightness, and the maximum allowed current. Use an individual resistor for each LED; do not share one resistor among LEDs.

Use an external regulated, isolated 9 V DC supply and 2.1 mm center-negative jack. No numeric supply rating is specified until the selected relay coil current and maximum simultaneously energized coils are known. Estimate worst-case demand as logic current plus the sum of energized-coil currents plus LED current; provide margin, then confirm draw with a current-limited bench supply. Add reverse-polarity protection and local decoupling, and check relay pickup voltage after any protection-device drop.

## Audio jacks, enclosure, and wiring

- Use mono unbalanced TS jacks, preferably insulated-body solder-lug types, to control sleeve grounding.
- Use a die-cast aluminum enclosure sized by an actual full-size panel mock-up. Approximate 1590B/DD/XX classes are not guaranteed fit.
- Use stranded insulated hookup wire for control wiring and short audio runs; use low-capacitance shielded cable for long or noise-sensitive audio runs.
- Keep audio tips/contact wiring away from coil, clock, reset, and LED wiring. Twist coil supply/return pairs; cross control and audio runs at right angles if they must cross.
- Follow the one-point audio/enclosure bond in [architecture.md](architecture.md#power-grounding-and-wiring). Insulate unused relay terminals and protect solder joints with heat-shrink.

## Manufacturer substitutions

Manufacturer substitutions are acceptable only when the replacement meets the required coil, contact, voltage, current, mounting, and isolation specifications. Search distributors such as Mouser or DigiKey for relays, CMOS ICs, and protection parts; Tayda or Musikding for common pedal jacks, LEDs, enclosures, and hardware; and local suppliers for wire/fasteners. Verify current datasheets and availability before ordering.
