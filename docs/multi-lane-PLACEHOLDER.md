# Multi-Lane Trafik (2–8 lanes, made to order)

## Purpose

Multi-Lane Trafik adds the device/lane selector to the shared platform: `N` device lanes (2–8), each with a TO output and a matched FX Send/FX Return pair, plus the same two shared pedal-loop positions as the Single Lane. Lane counts are a **build parameter** — studios can order a specific number of lanes within 1–8, since the control core, firmware, and buffer architecture are lane-count agnostic; lanes add relay groups and jacks only.

## Form factors

| Format | Lanes | Notes |
| --- | --- | --- |
| Stompbox (1590-class, 9 V) | 2–4 | Panel-density limit; TO/FX banks along bottom and sides |
| 1U rack (internal supply, ±15 V audio rails) | 1–8 | Jacks on the rear panel; suits the long-cable studio rig; IEC inlet, logic + bipolar audio rails |

At 8 lanes the unit carries 3N+5 = 29 TS jacks — verify rear-panel density and connector clearance on a full-size mock-up before fabrication.

## Connections and layout

Per lane `n`: TO n (output to device n input), FX Return n (input from device n FX Send), FX Send n (output to device n FX Return). Plus Instrument IN, and Loop A/B Send/Return. Stompbox layout follows the family convention (Instrument IN right, FX Return upper left, FX Send upper right, Loop B above Loop A left, TO bank bottom, switches across the middle — device selector in the center). Rack format: audio on the rear, footswitch connections and indicators on the front or via the companion foot control.

Footswitches: momentary normally-open SPST (2-pin) — Pre-FX, Pre-TO, and the device/lane control. One single-color lane LED per lane, only the selected lane lit, in addition to the four state LEDs per loop position.

## Signal flow and allocation

Same as the family spec in [architecture.md](architecture.md):

```text
Instrument IN → Pre-FX loop position → lane selector → TO n → device n input
device n FX Send → FX Return n → Pre-TO loop position → FX Send n → device n FX Return
```

Loop positions cycle Off → A → B → A→B → Off independently; Pre-FX takes priority when both positions request the same loop; LEDs show the applied state. Lane selection is break-before-make; unselected lane tips are isolated. The three-buffer map is unchanged at any lane count: Loop A Return, Loop B Return, and the instrument-chain output **ahead of the lane selector** (one stage feeds all TO ports).

## Electronics and power

Active platform per [architecture.md](architecture.md): passive instrument path to the first loop return, three unity buffers, microcontroller control implementing the loop allocation and lane selection with watchdog and defined power-on/fail-safe states (power loss or firmware fault → passive straight-through on lane 1, loops bypassed, other lanes isolated), transistor relay drivers with flyback clamps, low-level signal relays.

Stompbox: 9 V isolated supply, rail-to-rail buffer op-amps or validated headroom. Rack: internal linear supply with ±15 V audio rails. A firmware self-test mode that walks the complete state × relay truth table is required for every made-to-order lane count.

The contact-level circuit has not been designed. Refer to [electronics-reference.md](electronics-reference.md) and the pre-fabrication gate in [build-guide.md](build-guide.md).

## Estimated materials

Not yet estimated for the active platform. Scale from the [Single Lane Revision A BOM](bom-PLACEHOLDER.md) as a planning aid only: jacks = 3N+5; loop-insert relays fixed at 4; lane-selection relay groups grow with N; buffers fixed at 3; lane LEDs N. Recalculate after the contact-level design exists.

## Acceptance checks

All Single Lane checks (per-position state cycles, A-before-B, duplicate-loop priority, applied LEDs, fail-safe behavior) **plus**: lane isolation (no continuity between TO outputs or FX paths of different lanes), TO/FX pair switching together per lane, break-before-make on lane changes, and the full state × relay truth table for the delivered lane count. Complete the [build guide](build-guide.md) checks before connecting devices.
