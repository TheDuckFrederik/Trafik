# 4 Lane Trafik

## Purpose

The 4 Lane Trafik is the full product configuration: four selectable device paths, two independently controlled pedal-loop positions, and a center lane-selection footswitch. It uses the three-foot-switch layout established for the family.

## Panel layout and connections

Use a metal enclosure approximately 1590XX size or equivalent, subject to a full-size fit check.

- Right side: one **Instrument IN** jack.
- Top-left bank: **FX Return 1–4**.
- Top-right bank: **FX Send 1–4**.
- Left side: Loop B Send/Return in the upper position and Loop A Send/Return below it.
- Bottom bank: **TO 1–4**.
- Across the middle: left **Pre-FX**, center **Device**, right **Pre-TO** momentary footswitches.
- Indicators: four loop-state indicators for each pedal position near the TO bank and between the FX banks; four device-lane indicators beside the center switch.
- DC input on a side panel, away from audio conductors.

There are seventeen audio jacks total: Instrument IN, four TO outputs, eight device FX jacks, and four pedal-loop jacks. Use mono ¼-inch TS connectors. Panel labeling and jack direction follow the [architecture](architecture.md).

The three large middle circles in the agreed layout are the footswitches. The small rectangles beside the TO bank and between the FX banks are pedal-loop state indicators; they are not audio connections.

## Routing

The selected lane is the only connected device path:

`Instrument IN → Pre-FX position → TO n → device n input`

`device n FX Send → FX Return n → Pre-TO position → FX Send n → device n FX Return`

The center switch selects lane 1, 2, 3, or 4; only the corresponding TO output and FX pair are connected. All other lane jacks remain isolated. Each outer switch selects Off, Loop A, Loop B, or A→B at its signal position. Loop A is first in the series chain. Each physical loop can appear at only one position; if both positions request it, Pre-FX has priority and the indicator reports the actual applied state.

## Footswitch and LED behavior

- **Left / Pre-FX:** first press A; second B; third A→B; fourth Off; repeat.
- **Center / Device:** selects 1 → 2 → 3 → 4 → 1.
- **Right / Pre-TO:** first press A; second B; third A→B; fourth Off; repeat.

Each loop position has four state indicators labeled Off, A, B, and A→B. Four lane indicators identify the selected device path. Only the active state/lane is lit. At power-up, both loop positions reset Off and lane 1 is selected.

## Hardware implementation

Use debounced momentary footswitches, fixed-function CMOS counters and decoders, transistor relay drivers, and low-level audio relays. The lane counter wraps after lane 4. Use relay groups to switch each lane’s TO and FX pair as one selection, with break-before-make behavior. Unselected lanes must remain isolated. The loop allocation interlock is required to prevent either pedal chain being inserted into both signal paths; derive the state LEDs from the interlocked outputs. Relay contacts provide a passive audio path; the 9 V supply operates only logic and coils.

## Build quantity and estimate

See the [BOM](bom-PLACEHOLDER.md) for line-item parts and retailer alternatives. The current planning estimate is **$254.00** before tax/shipping and **$292.10** with a 15% sourcing reserve. This estimate excludes tools, labor, and an external pedalboard power supply.

## Checkout

Verify every lane’s TO and corresponding FX route individually. Confirm the other three lanes remain isolated during each selection, then check both loop cycles, loop ordering, allocation interlock, indicators, power-up reset, and unpowered bypass. Complete the [build guide](build-guide.md) before connecting amplifiers or pedals.
