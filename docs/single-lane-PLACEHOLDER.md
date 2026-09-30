# Single Lane Trafik

## Purpose

The Single Lane Trafik provides the same two pedal-loop insertion positions as the multi-lane models, but has one fixed device path and no device-selection footswitch. Two momentary footswitches independently select which pedal loops are active at the Pre-FX and Pre-TO positions.

## Panel layout and connections

Use a metal enclosure approximately 1590B size or equivalent, subject to a full-size fit check.

- Right side: one **Instrument IN** jack.
- Upper-left bank: one **FX Return 1** jack.
- Upper-right bank: one **FX Send 1** jack.
- Left edge: **Loop B Send** above **Loop B Return**, then **Loop A Send** above **Loop A Return**.
- Bottom bank: one **TO 1** jack.
- Across the middle: left **Pre-FX** footswitch and right **Pre-TO** footswitch.
- Indicators: four-state Pre-FX indicator near TO; four-state Pre-TO indicator between the FX banks.
- DC input and any status/ground-lift hardware go on an otherwise clear side panel.

There are eight audio jacks total: Instrument IN, TO 1, FX Send 1, FX Return 1, and the four Loop A/B send/return jacks. Use mono ¼-inch TS jacks.

## Routing

`Instrument IN → Pre-FX position → TO 1 → device input`

`device FX Send → FX Return 1 → Pre-TO position → FX Send 1 → device FX Return`

At each position the physical Loop A and Loop B paths can be bypassed, inserted individually, or placed in series in the order A then B. When both positions request the same physical loop, it is assigned to Pre-FX and omitted at Pre-TO. The indicators report the resulting applied state. This prevents the same external pedal chain from being inserted twice.

## Footswitch and LED behavior

Each footswitch advances its own position through the following sequence:

1. First press: Loop A
2. Second press: Loop B
3. Third press: Loop A → Loop B
4. Fourth press: Off
5. The next press returns to Loop A

Each position has four indicators, labeled Off, A, B, and A→B. Exactly one is lit to show the applied routing; if interlock priority changes a requested state, the LEDs show the state actually in circuit. At power-up, both positions reset to Off. There is no lane indicator or device-selection control; TO 1 and its FX pair are permanently the device path.

## Hardware implementation

Use two independent debounced CMOS four-state counters, relay drivers and low-level audio signal relays. Each loop position requires relay contacts for bypass/insert selection, and the loop-allocation interlock must prevent either physical loop from appearing in both positions. Use break-before-make switching on the selected device FX path. The loop LEDs are driven from the applied-state decoder, not the raw footswitch counter.

The loop counter advances on a momentary normally-open footswitch closure. On power loss, relays return to their unpowered bypass condition; the control reset starts both counters at Off when power is restored. Use the relay contact arrangement so a failure or loss of control power does not leave a pedal loop connected in series unexpectedly.

## Build quantity and estimate

See the [BOM](bom-PLACEHOLDER.md) for line-item parts and retailer alternatives. The current planning estimate is **$142.40** before tax/shipping and **$163.76** with a 15% sourcing reserve. This estimate excludes tools, labor, and an external pedalboard power supply.

## Checkout

Verify continuity and each loop state independently before connecting an amplifier. Then check the two insertion positions, loop-allocation interlock, silent/low-noise relay changes, power-up reset, and power-loss bypass using the [build guide](build-guide.md).
