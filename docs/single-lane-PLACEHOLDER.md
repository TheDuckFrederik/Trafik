# Single Lane Trafik

## Purpose

Single Lane Trafik has the two independently controlled pedal-loop positions and one fixed device path. It omits only the device/lane selector and lane indicators.

> **Circuit design:** [single-lane-circuit.md](single-lane-circuit.md) is the contact-level circuit design for this model (4 DPDT relays, 7 CMOS ICs, passive audio, 9 V rail) with the full state/relay/LED truth table. It is captured as the KiCad 10 schematic `SingleLane/SingleLane.kicad_sch` (netlist-verified against the design tables; ERC clean). It is a functional design, not a built and measured unit.
>
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

The contact-level design in [single-lane-circuit.md](single-lane-circuit.md) finalizes the relay count at **4 DPDT relays** (TQ2-9V class) and a 7-IC CMOS control board implementing the shared-loop allocation with Pre-FX priority. Manufacturer-specific pinouts still must be verified against the exact datasheets before board layout; see [electronics-reference.md](electronics-reference.md) for part-selection criteria and [build-guide.md](build-guide.md) for the pre-fabrication gate and test procedure.

## Estimated materials

The [BOM](bom-PLACEHOLDER.md) gives the current illustrative category-level estimate and retailer examples. Its relay and control-board quantities for Single Lane were updated (2026-10-13) to match the contact-level design; 2/4-Lane quantities remain preliminary allowances. The Single Lane estimate is $131.20 before tax and shipping ($150.88 including a 15% reserve); it excludes tools, labor, and an external supply. Recalculate the BOM if the circuit design changes.

## Acceptance checks

Verify the Off/A/B/A→B sequence independently at both positions, correct A-before-B order, duplicate-loop priority and applied LEDs, the fixed TO/FX route, bypass on loss of power, and safe isolation of unused contacts. Complete all checks in the [build guide](build-guide.md) before connecting devices.
