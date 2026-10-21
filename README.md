# Trafik

Trafik is a family of **active, relay-switched pedal-loop routers** for selecting a device lane and placing two external pedal loops at defined points in the signal path: passive from the instrument jack to the first loop return, buffered thereafter. Audio is switched by electromechanical relays; control is firmware-based with fail-safe passive-through behavior on power loss or fault. Units are **made to order**.

## Product family

One platform, sold as a lane count in one of two form factors:

| Model | Device lanes | Formats | Use |
| --- | ---: | --- | --- |
| Single Lane Trafik | 1 | Stompbox | Two pedal-loop positions; one fixed device path; the entry point of the platform |
| Multi-Lane Trafik | 2–4 standard (custom up to 8 on order) | Stompbox (2–4) or 1U rack (2–8) | Adds device/lane selection; studios can order a specific lane count |

Both formats share the same control core, buffer architecture, and loop-allocation logic; lane count is a build parameter, not a redesign. Pedal loops are always two (A, B), shared between the Pre-FX and Pre-TO positions with Pre-FX priority.

## Documentation

- [Architecture and common electrical definitions](docs/architecture.md) — active platform spec: buffer map, fuzz rule, loop allocation, control requirements, formats
- [Single Lane Trafik](docs/single-lane-prototype.md)
- [Multi-Lane Trafik (2–8 lanes, made to order)](docs/multi-lane-prototype.md)
- [Single Lane circuit design, Revision B (active platform — exact parts, values, pin map)](docs/single-lane-circuit-b.md)
- [Single Lane Rev B — component list and pin-by-pin connections](docs/single-lane-circuit-b-netlist.md)
- [Single Lane circuit design, Revision A (passive, archived reference)](docs/single-lane-circuit.md) — the fully verified passive/CMOS predecessor; captured in `SingleLane/SingleLane.kicad_sch` (KiCad 10; netlist-verified, clean ERC)
- [Single-loop buffer/relay prototype circuit and wiring diagram](docs/single-loop-prototype.md) (bench prototype lineage)
- [Single-loop prototype visual wiring diagram (SVG)](docs/single-loop-prototype.svg)
- [Electronics reference (part categories)](docs/electronics-reference.md)
- [Bill of materials, vendors, and estimates](docs/bom-prototype.md)
- [Fabrication and build guide](docs/build-guide.md)

## Design basis

The panel and control descriptions follow the agreed layout: Instrument IN on the right; FX Return jacks at upper left; FX Send jacks at upper right; Loop B above Loop A on the left; device TO jacks along the bottom (stompbox) or rear (rack); and the pedal-position footswitches across the middle. The large circles in the stompbox layout are footswitches, not audio-routing nodes.

The active platform is currently a specification: its contact-level circuit has not been designed, and no unit has been built. The descriptions define a buildable functional design, but are not a substitute for a verified, model-specific wiring schematic or a dimensioned drill template. Confirm connector footprints, relay pinouts, control-board connections, and enclosure clearances on a full-size mock-up before drilling or wiring. Pricing is a planning estimate in USD, not a supplier quotation.
