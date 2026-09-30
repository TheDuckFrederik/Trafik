# Trafik

Trafik is a family of passive-audio, relay-switched pedal-loop routers for selecting a device lane and placing two external pedal loops at defined points in the signal path. The control circuit uses fixed-function CMOS logic; the audio path is switched by electromechanical relays. There is no firmware.

## Product family

| Model | Device lanes | Footswitches | Use |
| --- | ---: | ---: | --- |
| [Single Lane Trafik](docs/single-lane-PLACEHOLDER.md) | 1 | 2 | Two pedal-loop positions; no device selector |
| [2 Lane Trafik](docs/two-lane-PLACEHOLDER.md) | 2 | 3 | Two device lanes and two pedal-loop positions |
| [4 Lane Trafik](docs/four-lane-PLACEHOLDER.md) | 4 | 3 | Four device lanes and two pedal-loop positions |

## Documentation

- [Architecture and common electrical definitions](docs/architecture.md)
- [Single Lane Trafik](docs/single-lane-PLACEHOLDER.md)
- [2 Lane Trafik](docs/two-lane-PLACEHOLDER.md)
- [4 Lane Trafik](docs/four-lane-PLACEHOLDER.md)
- [Bill of materials, vendors, and estimates](docs/bom-PLACEHOLDER.md)
- [Fabrication and build guide](docs/build-guide.md)

## Design basis

The panel and control descriptions in this package follow the agreed layout: Instrument IN on the right; FX Return jacks at upper left; FX Send jacks at upper right; Loop B above Loop A on the left; device TO jacks along the bottom; and two pedal-position footswitches across the middle. The 2- and 4-lane models add a device-selection footswitch in the center. The three large circles in the middle are footswitches, not audio-routing nodes.

The descriptions define a buildable functional design, but are not a substitute for a verified, model-specific wiring schematic or a dimensioned drill template. Confirm connector footprints, relay pinouts, control-board connections, and enclosure clearances on a full-size mock-up before drilling or wiring. Pricing is a planning estimate in USD, not a supplier quotation.
