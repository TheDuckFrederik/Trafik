# Trafik bill of materials and sourcing

## Price basis

All amounts are **USD planning estimates as of 30 September 2026**, before sales tax and shipping. They are not live quotes; retailer stock, shipping, tax, and regional pricing vary. The base estimate uses a mix of low-volume parts from Tayda Electronics and a standard enclosure from a Hammond/1590-series retailer. For safety-critical fit and electrical ratings, use the specified part class rather than selecting on price alone.

This is an illustrative budget for the single hardware-only implementation described in [architecture.md](architecture.md#hardware-control-and-switching). For the shared component categories and selection constraints, see [electronics-reference.md](electronics-reference.md). Prices are not vendor quotes; confirm live stock, datasheets, and dimensions before ordering.

Prices assume single-unit DIY builds. Tools, labor, finishing, custom PCB fabrication, and an external 9 V pedalboard supply are excluded. Add 15% to the material subtotal as a sourcing reserve for shipping, small-quantity price changes, and incidental hardware.

## Parts common to all models

| Part | Specification | Vendor/source examples | Estimate per unit |
| --- | --- | --- | ---: |
| Audio jack | ¼-inch mono TS, isolated or insulated-body type, solder-lug | Tayda; Mouser/DigiKey (Neutrik/Rean); Switchcraft | $2.00 |
| Momentary footswitch | Latching-free, normally-open SPST contact, rugged stomp switch | Tayda; Musikding; Mouser (mechanical equivalents) | $10.00 |
| Signal relay | Contact form/quantity determined from completed matrix; 9 V coil only when datasheet permits | Mouser/DigiKey (Omron, Panasonic, equivalents) | $2.20 allowance |
| Control components | CMOS counter, debounce/timing parts, logic, relay-driver transistors, protection diodes, resistors and capacitors | Mouser/DigiKey; Tayda for common passives | Included in board/passives lines |
| Indicator LED | 3 mm or 5 mm, diffused, color per panel legend | Tayda; Mouser/DigiKey | $0.30 |
| Control PCB | Through-hole fixed-function control and driver board, sized to variant relay count | Low-volume PCB fabrication; hand-wired perfboard alternative | Variant estimate below |
| Hook-up wire | 24–26 AWG stranded insulated wire; shielded audio cable for longer/high-impedance runs | Tayda; Mouser/DigiKey; local electronics supplier | Variant estimate below |
| DC input jack | 2.1 mm barrel, center-negative panel jack | Tayda; Mouser/DigiKey | $2.00 |
| Fasteners/insulation | Jack nuts, switch nuts, PCB standoffs, insulating washers, heat-shrink | Tayda; local hardware supplier | Variant estimate below |

The electronics reference gives the required part specifications and warns which quantities remain provisional.

Use a regulated, isolated, center-negative 9 V supply. No fixed current rating is specified until relay models and the maximum energized-coil count are known. Calculate worst-case current from the selected data sheets and verify by measurement before selecting a supply. A suitable external adapter is budgeted at approximately **$15–30** and is not included in the totals.

## Single Lane Trafik

Eight TS jacks: Instrument IN, TO 1, FX Send 1, FX Return 1, and Loop A/B Send/Return. Two loop-position footswitches; no lane switch. Relay quantity is a planning allowance for loop selection, bypass and interlock.

| Item | Qty. | Unit estimate | Extended |
| --- | ---: | ---: | ---: |
| 1590B-size aluminum enclosure or equivalent | 1 | $35.00 | $35.00 |
| ¼-inch TS mono jacks | 8 | $2.00 | $16.00 |
| Momentary footswitches | 2 | $10.00 | $20.00 |
| Signal relays (preliminary allowance; see note) | 10 | $2.20 | $22.00 |
| CMOS control/driver PCB | 1 | $22.00 | $22.00 |
| Discrete logic, drivers, protection and passives | 1 lot | $10.00 | $10.00 |
| Indicator LEDs | 8 | $0.30 | $2.40 |
| Hook-up/shielded wire | 1 lot | $8.00 | $8.00 |
| DC jack | 1 | $2.00 | $2.00 |
| Hardware, standoffs, insulation, heat-shrink | 1 lot | $5.00 | $5.00 |
| **Materials subtotal** |  |  | **$142.40** |
| 15% sourcing reserve |  |  | **$21.36** |
| **Estimated materials with reserve** |  |  | **$163.76** |

## 2 Lane Trafik

Eleven TS jacks: Instrument IN, TO 1–2, FX Send/Return 1–2, and Loop A/B Send/Return. Three momentary footswitches, including lane selection.

| Item | Qty. | Unit estimate | Extended |
| --- | ---: | ---: | ---: |
| 1590DD-size aluminum enclosure or equivalent | 1 | $42.00 | $42.00 |
| ¼-inch TS mono jacks | 11 | $2.00 | $22.00 |
| Momentary footswitches | 3 | $10.00 | $30.00 |
| Signal relays (preliminary allowance; see note) | 14 | $2.20 | $30.80 |
| CMOS control/driver PCB | 1 | $28.00 | $28.00 |
| Discrete logic, drivers, protection and passives | 1 lot | $13.00 | $13.00 |
| Indicator LEDs | 10 | $0.30 | $3.00 |
| Hook-up/shielded wire | 1 lot | $12.00 | $12.00 |
| DC jack | 1 | $2.00 | $2.00 |
| Hardware, standoffs, insulation, heat-shrink | 1 lot | $6.00 | $6.00 |
| **Materials subtotal** |  |  | **$188.80** |
| 15% sourcing reserve |  |  | **$28.32** |
| **Estimated materials with reserve** |  |  | **$217.12** |

## 4 Lane Trafik

Seventeen TS jacks: Instrument IN, TO 1–4, FX Send/Return 1–4, and Loop A/B Send/Return. Three momentary footswitches and four lane indicators.

| Item | Qty. | Unit estimate | Extended |
| --- | ---: | ---: | ---: |
| 1590XX-size aluminum enclosure or equivalent | 1 | $58.00 | $58.00 |
| ¼-inch TS mono jacks | 17 | $2.00 | $34.00 |
| Momentary footswitches | 3 | $10.00 | $30.00 |
| Signal relays (preliminary allowance; see note) | 22 | $2.20 | $48.40 |
| CMOS control/driver PCB | 1 | $34.00 | $34.00 |
| Discrete logic, drivers, protection and passives | 1 lot | $18.00 | $18.00 |
| Indicator LEDs | 12 | $0.30 | $3.60 |
| Hook-up/shielded wire | 1 lot | $18.00 | $18.00 |
| DC jack | 1 | $2.00 | $2.00 |
| Hardware, standoffs, insulation, heat-shrink | 1 lot | $8.00 | $8.00 |
| **Materials subtotal** |  |  | **$254.00** |
| 15% sourcing reserve |  |  | **$38.10** |
| **Estimated materials with reserve** |  |  | **$292.10** |

## Shared vs. variant-specific parts

**Shared:** isolated TS jack type and four loop jacks; two loop-position footswitches; eight loop-state LEDs and resistors; 9 V input/protection/decoupling; fixed-function CMOS debounce/counters; relay drivers and protection parts; wire and mounting consumables.

**Variant-specific:** enclosure size; TO and device FX jack counts; lane-selection counter and center footswitch for 2/4 Lane; lane indicator count; control-board capacity; number of signal relays and wiring. Single Lane omits the device-selection hardware.

The relay counts and control-PCB prices above are budgeting allowances only. The repository does not yet contain a validated relay contact matrix, manufacturer-specific schematic, or PCB design, so these quantities cannot be treated as a complete build BOM. Before ordering, complete and independently check the contact-level design, select the actual relay, recalculate the relay/driver/board quantities, and add spares if appropriate. The listed subtotal will change when that design is completed.

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
| Single Lane | $142.40 | $163.76 | +$15–30, after calculating supply current |
| 2 Lane | $188.80 | $217.12 | +$15–30, after calculating supply current |
| 4 Lane | $254.00 | $292.10 | +$15–30, after calculating supply current |

These totals exclude taxes, tools, labor, finishing, custom fabrication, and any added parts revealed by the completed contact-level design. They are estimates rather than complete final build totals. Enclosure sizes are starting points only: confirm actual component dimensions, jack clearance, and footswitch spacing before purchase.
