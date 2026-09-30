# Single Lane Trafik

## Purpose

Single Lane Trafik has the two independently controlled pedal-loop positions and one fixed device path. It omits only the device/lane selector and lane indicators.

> **Separate bench prototype:** [single-loop-prototype.md](single-loop-prototype.md) documents an earlier, simpler one-loop buffer/relay circuit. It is not the complete Single Lane Trafik routing design described on this page.

## Connections and physical layout

Starting enclosure: approximately 1590B-class, subject to a full-size fit check. Use eight isolated mono ¼-inch TS jacks:

| Count | Jack labels |
| ---: | --- |
| 1 | Instrument IN |
| 1 | TO 1 |
| 1 | FX Return 1 |
| 1 | FX Send 1 |
| 2 | Loop A Send / Return |
| 2 | Loop B Send / Return |

Keep Instrument IN on the right; FX Return at the upper left; FX Send at the upper right; Loop B above Loop A on the left; and TO 1 at the bottom. Put Pre-FX and Pre-TO footswitches across the center. Each position has four separate labeled state LEDs. Provide clearance for the 9 V DC jack and control board. The large circles on the physical layout represent footswitches.

## Signal flow

```text
Instrument IN → Pre-FX loop position → TO 1 → device input
device FX Send → FX Return 1 → Pre-TO loop position → FX Send 1 → device FX Return
```

TO 1 and the FX pair are the fixed device path. The two outer momentary normally-open footswitches independently cycle their loop-position requests through Off → A → B → A→B → Off. A+B always means A first, then B.

The physical Loop A and Loop B send/return pairs are shared between the positions. When the same loop is requested at both positions, Pre-FX takes priority; Pre-TO applies only loops not already assigned to Pre-FX. The state LEDs show applied routing, not an overridden request. See the allocation rule and required control behavior in [architecture.md](architecture.md#loop-states-and-assignment).

## Electronics and power

Use the shared fixed-function CMOS/relay implementation described in [architecture.md](architecture.md#hardware-control-and-switching). This model requires two debounced loop-state counter channels, their relay drivers/contact groups, eight state LEDs with individual resistors, and the common 9 V input/protection/decoupling parts. There is no lane counter, lane switch, or lane LED.

The contact count and relay part number cannot be safely finalized from a generic DPDT description: the shared-loop assignment matrix, bypass state, and exact manufacturer pinout must be verified before board layout. Refer to [electronics-reference.md](electronics-reference.md) for part-selection criteria and [build-guide.md](build-guide.md) for the pre-fabrication gate and test procedure.

## Estimated materials

The [BOM](bom-PLACEHOLDER.md) gives the current illustrative category-level estimate and retailer examples. Its relay and control-board quantities are preliminary allowances, not a validated netlist-based count. The estimate is $142.40 before tax and shipping ($163.76 including a 15% reserve); it excludes tools, labor, and an external supply. Recalculate the BOM after the contact-level circuit is complete.

## Acceptance checks

Verify the Off/A/B/A→B sequence independently at both positions, correct A-before-B order, duplicate-loop priority and applied LEDs, the fixed TO/FX route, bypass on loss of power, and safe isolation of unused contacts. Complete all checks in the [build guide](build-guide.md) before connecting devices.
