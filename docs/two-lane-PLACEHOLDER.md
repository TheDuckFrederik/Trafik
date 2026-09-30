# 2 Lane Trafik

## Purpose

2 Lane Trafik provides two selectable device paths, with the same two independently controlled pedal-loop positions as Single Lane Trafik. Its third, center footswitch selects the device lane.

## Connections and physical layout

Starting enclosure: approximately 1590DD-class, subject to a full-size fit check. Use eleven isolated mono ¼-inch TS jacks:

| Count | Jack labels |
| ---: | --- |
| 1 | Instrument IN |
| 2 | TO 1, TO 2 |
| 2 | FX Return 1, FX Return 2 |
| 2 | FX Send 1, FX Send 2 |
| 2 | Loop A Send / Return |
| 2 | Loop B Send / Return |

Keep Instrument IN on the right; FX Returns at the upper left; FX Sends at the upper right; Loop B above Loop A on the left; and TO 1/2 along the bottom. Arrange the Pre-FX, Device, and Pre-TO footswitches across the center. Provide four state LEDs for each loop position, two lane LEDs, and clearance for the 9 V DC jack/control board. The large circles on the physical layout represent footswitches.

## Signal flow and controls

For selected lane `n`:

```text
Instrument IN → Pre-FX loop position → TO n → device n input
device n FX Send → FX Return n → Pre-TO loop position → FX Send n → device n FX Return
```

Only the selected TO output and its matching FX pair connect. The center momentary normally-open footswitch cycles lane 1 → lane 2 → lane 1. Each outer footswitch independently cycles its position through Off → A → B → A→B → Off; A+B is A then B.

The one physical Loop A and Loop B send/return pair is shared between positions. On a duplicate request, Pre-FX has priority; Pre-TO applies only loops not already assigned to Pre-FX. Each position's LEDs show applied routing. See [architecture.md](architecture.md#loop-states-and-assignment) for the allocation rule.

| Device-switch press | Selected lane | Lane LED |
| ---: | --- | --- |
| Power-on reset | 1 | Lane 1 |
| 1 | 2 | Lane 2 |
| 2 | 1 | Lane 1 |

## Electronics and power

Use the shared fixed-function CMOS/relay implementation described in [architecture.md](architecture.md#hardware-control-and-switching): two debounced four-state loop counters, a modulo-2 lane counter, relay drivers/contact groups, ten state/lane LEDs with individual resistors, and the common 9 V input/protection/decoupling parts. There is no software or microcontroller.

Lane switching must change TO, FX Return, and FX Send together and must be break-before-make so two lanes are never tied together. The chosen contacts must also implement the shared-loop allocation rule and the documented unpowered lane-1/bypass state. A generic DPDT description is not enough to determine the exact contact count or pinout; complete and verify the manufacturer-specific contact matrix before board layout. See [electronics-reference.md](electronics-reference.md) and [build-guide.md](build-guide.md).

## Estimated materials

See the [BOM](bom-PLACEHOLDER.md) for category-level quantities and retailer examples. Its relay/control-board quantities are preliminary allowances, not a validated netlist-based count. The estimate is $188.80 before tax and shipping ($217.12 including a 15% reserve); it excludes tools, labor, and an external supply. Recalculate it after the contact-level circuit is complete.

## Acceptance checks

Verify both lanes' TO and matching FX paths, isolation of the unselected lane, all four loop states at both positions, A-before-B order, duplicate-loop priority/LED behavior, lane wraparound, break-before-make selection, and bypass/lane-1 behavior on loss of power. Follow the [build guide](build-guide.md) before connecting devices.
