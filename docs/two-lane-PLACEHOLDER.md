# 2 Lane Trafik

## Purpose

The 2 Lane Trafik is the multi-device version with two selectable device paths. It retains the two pedal-loop positions and adds a center device-selection footswitch. It has the same signal and loop behavior as the 4 Lane model, with two lanes rather than four.

## Panel layout and connections

Use a metal enclosure approximately 1590DD size or equivalent, subject to a full-size fit check.

- Right side: one **Instrument IN** jack.
- Upper-left bank: **FX Return 1** and **FX Return 2**.
- Upper-right bank: **FX Send 1** and **FX Send 2**.
- Left edge: Loop B Send/Return above Loop A Send/Return.
- Bottom bank: **TO 1** and **TO 2**.
- Across the middle: left **Pre-FX**, center **Device**, and right **Pre-TO** momentary footswitches.
- Indicators: four states for each loop position, located near the TO bank and between the FX banks; lane 1/2 indicators adjacent to the center switch.
- DC input on a side wall with clearance from audio wiring.

There are eleven audio jacks total: Instrument IN, two TO outputs, four device FX jacks, and four pedal-loop jacks. Use mono ¼-inch TS connectors. Label the FX jacks by lane and by direction from Trafik’s point of view as defined in the [architecture](architecture.md).

## Routing

Only one lane is connected at a time. For the selected lane:

`Instrument IN → Pre-FX position → TO n → device n input`

`device n FX Send → FX Return n → Pre-TO position → FX Send n → device n FX Return`

The unselected TO output and FX pair are isolated. Each outer footswitch selects Off, Loop A, Loop B, or A→B for its position. Loop A precedes Loop B when both are active. Each physical loop can be assigned to one position only; if both positions request the same loop, Pre-FX takes priority and the Pre-TO state indicator shows the applied, interlocked state.

## Footswitch and LED behavior

- **Left / Pre-FX:** cycles A → B → A+B → Off → repeat.
- **Center / Device:** selects lane 1 → lane 2 → lane 1.
- **Right / Pre-TO:** cycles A → B → A+B → Off → repeat.

Each loop position has four labeled indicators (Off, A, B, A→B). Two device indicators show the selected lane. One state per control is lit at a time. At power-up, both loop positions reset Off and lane 1 is selected. Momentary normally-open switches advance the hardware counters; they do not carry audio.

## Hardware implementation

Use fixed-function CMOS counters with debounced inputs, decoder/driver stages and electromechanical audio relays. Configure the device counter to wrap after lane 2. Relay contacts select the TO and matching FX Send/Return paths together; switching must be break-before-make so two devices are never temporarily tied together. Loop LEDs must reflect interlocked, applied routing. The unpowered relay state should bypass external loops and leave lane 1 as the passive fallback path.

## Build quantity and estimate

See the [BOM](bom-PLACEHOLDER.md) for line-item parts and retailer alternatives. The current planning estimate is **$188.80** before tax/shipping and **$217.12** with a 15% sourcing reserve. This estimate excludes tools, labor, and an external pedalboard power supply.

## Checkout

Test both lane positions and each paired TO/FX route with no device connected first. Check that switching never joins the two lanes, then verify all loop states, the duplicate-loop interlock, LEDs, power-up reset, and unpowered bypass. Use the [build guide](build-guide.md) for the full sequence.
