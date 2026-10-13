# Trafik bill of materials and sourcing

> **Platform note (2026-10-13):** The product family pivoted to the active platform (three buffers, microcontroller control, stompbox + 1U rack formats, 1–8 lanes made to order — see [architecture.md](architecture.md)). The estimates below price the **Revision A passive Single Lane design only**; they are retained as a baseline and do not include the active platform's op-amps, MCU, or rack hardware. Recalculate per SKU once the active contact-level designs exist.

## Price basis

All amounts are **USD planning estimates as of 30 September 2026**, before sales tax and shipping. They are not live quotes; retailer stock, shipping, tax, and regional pricing vary. The base estimate uses a mix of low-volume parts from Tayda Electronics and a standard enclosure from a Hammond/1590-series retailer. For safety-critical fit and electrical ratings, use the specified part class rather than selecting on price alone.

This BOM prices the Revision A passive Single Lane design described in [single-lane-circuit.md](single-lane-circuit.md). For part-category specifications, see [electronics-reference.md](electronics-reference.md).

Prices assume single-unit DIY builds. Tools, labor, finishing, custom PCB fabrication, and an external 9 V pedalboard supply are excluded. Add 15% to the material subtotal as a sourcing reserve for shipping, small-quantity price changes, and incidental hardware.

## Parts common to all models

| Part | Specification | Vendor/source examples | Estimate per unit |
| --- | --- | --- | ---: |
| Audio jack | ¼-inch mono TS, isolated or insulated-body type, solder-lug | Tayda; Mouser/DigiKey (Neutrik/Rean); Switchcraft | $2.00 |
| Momentary footswitch | Latching-free, normally-open SPST contact, rugged stomp switch | Tayda; Musikding; Mouser (mechanical equivalents) | $10.00 |
| Signal relay | DPDT, non-latching, 9 V coil, low-level audio contact rating, sealed preferred | Mouser/DigiKey (Omron, Panasonic, signal-relay equivalents) | $2.20 |
| Control components | CMOS counter, debounce/timing parts, logic, relay-driver transistors, protection diodes, resistors and capacitors | Mouser/DigiKey; Tayda for common passives | Included in board/passives lines |
| Indicator LED | 3 mm or 5 mm, diffused, color per panel legend | Tayda; Mouser/DigiKey | $0.30 |
| Control PCB | Through-hole fixed-function control and driver board, sized to variant relay count | Low-volume PCB fabrication; hand-wired perfboard alternative | Variant estimate below |
| Hook-up wire | 24–26 AWG stranded insulated wire; shielded audio cable for longer/high-impedance runs | Tayda; Mouser/DigiKey; local electronics supplier | Variant estimate below |
| DC input jack | 2.1 mm barrel, center-negative panel jack | Tayda; Mouser/DigiKey | $2.00 |
| Fasteners/insulation | Jack nuts, switch nuts, PCB standoffs, insulating washers, heat-shrink | Tayda; local hardware supplier | Variant estimate below |

Part categories for the active platform (microcontroller, buffers, rack hardware) are defined in [electronics-reference.md](electronics-reference.md); the table above reflects the Revision A passive design.

Use a regulated, isolated, center-negative 9 V supply rated for at least 100 mA as a separate pedalboard supply. Confirm the assembled circuit’s actual current draw before choosing the supply. A suitable external adapter typically adds approximately **$15–30**; it is not included in the totals.

## Single Lane Trafik

Eight TS jacks: Instrument IN, TO 1, FX Send 1, FX Return 1, and Loop A/B Send/Return. Two loop-position footswitches; no lane switch. Quantities for this variant match the contact-level design in [single-lane-circuit.md](single-lane-circuit.md) (updated 2026-10-13): 4 relays and a 7-IC CMOS control board.

| Item | Qty. | Unit estimate | Extended |
| --- | ---: | ---: | ---: |
| 1590B-size aluminum enclosure or equivalent | 1 | $35.00 | $35.00 |
| ¼-inch TS mono jacks | 8 | $2.00 | $16.00 |
| Momentary footswitches | 2 | $10.00 | $20.00 |
| DPDT audio signal relays | 4 | $2.20 | $8.80 |
| CMOS control/driver PCB | 1 | $22.00 | $22.00 |
| Discrete logic, drivers, protection and passives | 1 lot | $12.00 | $12.00 |
| Indicator LEDs | 8 | $0.30 | $2.40 |
| Hook-up/shielded wire | 1 lot | $8.00 | $8.00 |
| DC jack | 1 | $2.00 | $2.00 |
| Hardware, standoffs, insulation, heat-shrink | 1 lot | $5.00 | $5.00 |
| **Materials subtotal** |  |  | **$131.20** |
| 15% sourcing reserve |  |  | **$19.68** |
| **Estimated materials with reserve** |  |  | **$150.88** |

## Multi-Lane Trafik (2–8 lanes, made to order)

Under the made-to-order model, multi-lane units are parameterized by lane count N rather than fixed SKUs. Materials scale approximately as: jacks = 3N+5; loop-insert relays fixed at 4 plus lane-selection relay groups growing with N; lane LEDs N; larger enclosure or 1U rack chassis; same two loop-position footswitches plus the device/lane switch; same control board plus lane drivers. Two illustrative points from the earlier fixed-SKU estimates (passive design basis, DIY pricing):

| Item | 2 Lane (illustrative) | 4 Lane (illustrative) |
| --- | ---: | ---: |
| Enclosure/chassis | $42.00 (1590DD-class) | $58.00 (1590XX-class) |
| ¼-inch TS mono jacks (3N+5) | $22.00 (11) | $34.00 (17) |
| Momentary footswitches (3) | $30.00 | $30.00 |
| DPDT audio signal relays | $30.80 (14) | $48.40 (22) |
| Control/driver PCB | $28.00 | $34.00 |
| Discrete logic, drivers, protection, passives | $13.00 | $18.00 |
| Indicator LEDs | $3.00 | $3.60 |
| Wire, DC jack, hardware | $28.00 | $28.00 |
| **Materials subtotal** | **$188.80** | **$254.00** |
| **With 15% reserve** | **$217.12** | **$292.10** |

For the active platform (buffers, MCU, rack hardware for 1U units) and for lane counts other than 2 or 4, rebuild the estimate from the [electronics reference](electronics-reference.md) once the contact-level design for that configuration exists. At 8 lanes, allow for 29 jacks and verify chassis density.

## Shared vs. variant-specific parts

**Shared:** TS audio-jack type; four Loop A/B jacks; loop-position footswitches; four loop-state LEDs per position; power input; relay driver and protection parts; three-buffer set; wiring and mounting consumables.

**Configuration-specific (by lane count N and form factor):** enclosure or rack chassis; jack count (3N+5); lane-selection relay groups and the device/lane footswitch; lane indicator count; control-board capacity; rail hardware. Single Lane omits the device-selection hardware.

The relay counts and control-PCB prices above are budgeting allowances for the described switching functions, not a manufacturer-specific circuit design or a final relay pin-for-pin netlist. Before ordering, select the actual relay model and verify its contact configuration against the final wiring matrix; increase quantities for spares if the build is your first relay assembly.

## Retailer comparison and alternatives

| Source | Best use | Tradeoff |
| --- | --- | --- |
| **Tayda Electronics** | Low-cost jacks, LEDs, switches, wire, common passive components | Good low-volume pricing; check dimensions and contact specifications carefully, and consolidate orders to control shipping |
| **Mouser / DigiKey** | Named-brand relays, CMOS ICs, drivers, protection components, connectors | Strong datasheets and traceability; usually higher shipping/price for a tiny mixed order |
| **Musikding** | Pedal enclosures, footswitches, jacks and pedal-building hardware, especially European sourcing | Convenient pedal-specific stock; selection and per-piece pricing vary |
| **Hammond enclosure distributors** | Aluminum die-cast enclosure sizes such as 1590B/DD/XX equivalents | Consistent mechanical dimensions; finish and regional availability affect cost |
| **Switchcraft / Neutrik (Rean)** | Higher-grade audio jacks | More robust options than generic jacks, at a substantial per-jack cost increase; check panel depth and isolation |
| **Local electronics/hardware retailer** | Wire, fasteners, heat-shrink and small consumables | Avoids shipping and supports exact in-person fit checks; prices and stock vary |

For jack alternatives, choose insulated-body TS jacks to simplify sleeve-ground control; metal-body jacks are mechanically robust but can bond every sleeve to the enclosure. For relays, choose by contact configuration, low-level signal suitability, coil voltage, availability and verified footprint—not by coil voltage alone. The low-cost BOM assumes generic suitable parts; branded replacements can add roughly $1–4 per jack and $1–3 per relay.

## Total-cost summary

| Variant | Material subtotal | With 15% reserve | Add external supply if needed |
| --- | ---: | ---: | ---: |
| Single Lane | $131.20 | $150.88 | +$15–30 |
| Multi-Lane, N=2 (illustrative) | $188.80 | $217.12 | +$15–30 |
| Multi-Lane, N=4 (illustrative) | $254.00 | $292.10 | +$15–30 |

All totals are passive-Revision-A-basis planning estimates as of 2026-10-13. Active-platform costs (buffers, MCU, rack hardware) and lane counts other than 1, 2, or 4 are not included; rebuild per configuration once its contact-level design exists.

These totals exclude taxes, tools, labor, finishing, and custom fabrication. The enclosure sizes are starting points only: confirm actual component dimensions, jack clearance, and footswitch spacing before purchase.
