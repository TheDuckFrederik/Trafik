# Single Lane Trafik

## Purpose

Single Lane Trafik has the two independently controlled pedal-loop positions and one fixed device path. It omits only the device/lane selector and lane indicators. It is the first SKU of the active platform and its entry point: the same buffer architecture, control core, and loop-allocation logic as the multi-lane models.

> **Revision A (passive) reference:** [single-lane-circuit.md](single-lane-circuit.md) documents the fully verified passive/CMOS predecessor design (4 DPDT relays, 7 CMOS ICs, no firmware), captured in `SingleLane/SingleLane.kicad_sch`. It is archived as the bench reference, allocation truth-table source, and passive fallback. The active product circuit has not been designed yet.

## Connections and physical layout

Starting enclosure: approximately 1590B-class, subject to a full-size fit check. Use eight isolated mono ¼-inch TS jacks:

| Count | Jack labels |
| --- | --- |
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

TO 1 and the FX pair are the fixed device path. The two outer momentary normally-open (2-pin) footswitches independently cycle their loop-position requests through Off → A → B → A→B → Off. A+B always means A first, then B.

The physical Loop A and Loop B send/return pairs are shared between the positions. When the same loop is requested at both positions, Pre-FX takes priority; Pre-TO applies only loops not already assigned to Pre-FX. The state LEDs show applied routing, not an overridden request. See the allocation rule in [architecture.md](architecture.md#loop-states-and-assignment).

## Electronics and power

Implements the active platform described in [architecture.md](architecture.md): three unity buffers (Loop A Return, Loop B Return, instrument output ahead of the lane selector — here a single fixed output, with its fail-safe bypass relay), passive signal path from the instrument jack to the first loop return, microcontroller control of the two loop-position states and the Pre-FX-priority allocation, transistor relay drivers, and break-before-make relay audio switching. Two momentary SPST footswitches (onboard, or remote via two TS jacks on the optional patch panel); eight state LEDs driven to show the applied state.

Power: regulated isolated 9 V center-negative pedal supply sized above the measured worst-case draw (relays, logic, buffers). On the 9 V rail, use a rail-to-rail op-amp for the buffers or validate headroom explicitly.

The contact-level circuit for the active platform is specified with exact parts, values, and pin assignments in [single-lane-circuit-b.md](single-lane-circuit-b.md) (Revision B, 2026-10-20) — schematic drawing and bench verification pending. Refer to [electronics-reference.md](electronics-reference.md) for part-selection criteria and [build-guide.md](build-guide.md) for the pre-fabrication gate and test procedure. The Revision A passive circuit remains the reference for the relay contact matrix and allocation truth table.

## Estimated materials

The [BOM](bom-prototype.md) currently prices the Revision A passive design ($131.20 materials / $150.88 with reserve). The active platform BOM (buffers, MCU, drivers) is pending its circuit design; recalculate before ordering.

## Acceptance checks

Verify the Off/A/B/A→B sequence independently at both positions, correct A-before-B order, duplicate-loop priority and applied LEDs, the fixed TO/FX route, fail-safe passive-through on loss of power or firmware fault, and safe isolation of unused contacts. Complete all checks in the [build guide](build-guide.md) before connecting devices.
