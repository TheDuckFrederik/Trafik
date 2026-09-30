# 4 Lane Trafik

## Purpose

4 Lane Trafik is the largest family variant. It selects one of four device paths and provides the same two independently controlled pedal-loop positions as the other models.

## Connections and physical layout

Starting enclosure: approximately 1590XX-class, subject to a full-size fit check with every intended plug installed. Use seventeen isolated mono ¼-inch TS jacks:

| Count | Jack labels |
| ---: | --- |
| 1 | Instrument IN |
| 4 | TO 1, TO 2, TO 3, TO 4 |
| 4 | FX Return 1–4 |
| 4 | FX Send 1–4 |
| 2 | Loop A Send / Return |
| 2 | Loop B Send / Return |

Keep Instrument IN on the right; FX Returns at the upper left; FX Sends at the upper right; Loop B above Loop A on the left; and TO 1–4 along the bottom. Arrange Pre-FX, Device, and Pre-TO footswitches across the center. Provide four state LEDs for each loop position, four lane LEDs, and clearance for the 9 V DC jack/control board. The large circles on the physical layout represent footswitches, not routing nodes.

## Signal flow and controls

For selected lane `n`:

```text
Instrument IN → Pre-FX loop position → TO n → device n input
device n FX Send → FX Return n → Pre-TO loop position → FX Send n → device n FX Return
```

Only lane `n` is connected; every other TO and FX tip is isolated. The center momentary normally-open footswitch cycles lane 1 → 2 → 3 → 4 → 1. Each outer switch independently cycles Off → A → B → A→B → Off, with Loop A first in the series state.

The physical Loop A/B jacks are shared between the two positions. If both request the same loop, Pre-FX takes priority, and Pre-TO applies only loops not already assigned to Pre-FX. LEDs indicate applied routing. See [architecture.md](architecture.md#loop-states-and-assignment) for the exact allocation rule.

| Device-switch press | Selected lane | Lane LED |
| ---: | --- | --- |
| Power-on reset | 1 | Lane 1 |
| 1 | 2 | Lane 2 |
| 2 | 3 | Lane 3 |
| 3 | 4 | Lane 4 |
| 4 | 1 | Lane 1 |

## Electronics and power

Use the shared fixed-function CMOS/relay implementation described in [architecture.md](architecture.md#hardware-control-and-switching): two debounced four-state loop counters, a modulo-4 lane counter, relay drivers/contact groups, twelve state/lane LEDs with individual resistors, and common 9 V input/protection/decoupling parts. There is no microcontroller, firmware, or software-controlled switching.

Lane selection must switch TO, FX Return, and FX Send as one selection, break-before-make, and leave unselected lane tips isolated. The contact network must implement the duplicate-loop allocation rule and the documented unpowered lane-1/bypass state. Relay/contact quantities and terminal pinouts cannot be finalized from generic DPDT descriptions; complete and verify the manufacturer-specific contact matrix before board layout. See [electronics-reference.md](electronics-reference.md) for selection criteria and [build-guide.md](build-guide.md) for fabrication and checkout.

## Estimated materials

See the [BOM](bom-PLACEHOLDER.md) for category-level quantities and retailer examples. Its relay/control-board quantities are preliminary allowances, not a validated netlist-based count. The estimate is $254.00 before tax and shipping ($292.10 including a 15% reserve); it excludes tools, labor, and an external supply. Recalculate it after the contact-level circuit is complete.

## Acceptance checks

Verify all four lanes' paired TO/FX paths and isolation of every unselected lane; all loop states, ordering, duplicate-loop priority and applied LEDs; lane counter wraparound; break-before-make switching; and bypass/lane-1 behavior on power loss. Follow the [build guide](build-guide.md) before connecting devices.
