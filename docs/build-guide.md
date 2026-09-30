# Trafik fabrication and build guide

This guide applies to Single Lane, 2 Lane, and 4 Lane Trafik. Use the variant pages for jack counts and panel names, the [architecture](architecture.md) for signal and grounding rules, and the [electronics reference](electronics-reference.md) for component selection. Trafik is for instrument-level audio only: never connect mains voltage or an amplifier speaker output.

## 1. Design release: complete before buying or drilling

The repository defines the functions and acceptance criteria, but does not contain a verified pin-numbered schematic, relay-contact matrix, PCB layout, or dimensioned drill template. These are required design inputs, not optional finishing details.

1. Choose a variant and list its jacks, footswitches, LEDs, power connector, enclosure, and board dimensions.
2. Select actual part numbers and save current manufacturer data sheets/dimension drawings for the relays, jacks, switches, LEDs, DC jack, and enclosure.
3. Draw a schematic and relay contact matrix covering every lane and every combination of loop-position requests. The matrix must implement the Pre-FX priority rule, preserve A-before-B ordering, isolate unselected lanes, and define the unpowered state as lane 1 with both loops bypassed.
4. Check contact terminal numbers, make/break behavior, coil current, driver ratings, flyback suppression, reset polarity, and measured/calculated worst-case supply demand. Have the completed matrix independently checked before wiring audio.
5. Create a full-size panel drawing from the exact component dimensions. Check cable/plug clearance, footswitch spacing, board and lid clearance, wire bend radius, and access to nuts.
6. Recalculate the BOM from the schematic. The current relay counts/prices are budget allowances, not a build-ready netlist.

Do not proceed to final wiring if a contact assignment, power-loss state, or footprint remains guessed.

## 2. Enclosure layout and drilling

1. Remove the lid and protect the enclosure finish with masking tape.
2. Transfer the full-size layout. Keep Instrument IN on the right, FX Return at the upper left, FX Send at the upper right, Pedal Loop B above A on the left, and TO outputs along the bottom. Put Pre-FX and Pre-TO footswitches across the middle; add Device in the center only for 2/4 Lane. The large circles on the layout are footswitches.
3. Position the four loop-state indicators beside their associated controls/route areas and lane indicators beside Device. Place the DC jack on a side panel, away from audio wiring.
4. Check all clearances using the actual parts and inserted plugs. Do not assume a nominal enclosure family will fit the 4-lane jack count.
5. Center-punch, drill pilot holes, then enlarge to each manufacturer's specified cutout. Deburr both sides and remove every metal chip before installing electronics.
6. Isolate jack bodies from the enclosure where necessary. Plan exactly one enclosure-to-audio-ground bond.

## 3. Install and label panel hardware

Install jacks loosely first, orienting solder lugs for short, accessible wiring. Label every jack from Trafik's point of view: Instrument IN, TO n, FX Return n, FX Send n, Loop A Send/Return, Loop B Send/Return. Install the momentary normally-open Pre-FX and Pre-TO switches and (for multi-lane models) the center Device switch. Fit the LEDs with consistent polarity and labels: Off, A, B, A→B; lane numbers 1…N. Mount the board on insulated standoffs and keep coil/contact terminals clear of the enclosure.

## 4. Assemble control, indication, and power

1. Build from the released schematic, not from the generic block diagram alone. CMOS counters use one-hot states; debounce and shape each switch pulse so each press advances once. Provide a defined power-on reset to both loop Off states and lane 1.
2. Apply the four-state wrap/reset and lane modulo behavior specified in the [electronics reference](electronics-reference.md#control-logic). Do not leave CMOS inputs floating.
3. Build the Pre-FX/Pre-TO allocation logic so Pre-TO cannot select a loop already applied at Pre-FX. Drive loop LEDs from applied states after this allocation.
4. Drive each relay coil through a correctly sized transistor stage with the designed suppression component. Never drive a coil directly from a CMOS output. Verify suppression polarity and transistor pinout from the relevant data sheets.
5. Wire the 9 V input, reverse-polarity protection, and local decoupling exactly as designed. Confirm polarity, logic operating voltage, relay pickup voltage, worst-case current calculation, and continuity before attaching an external supply.
6. Keep control/coil/LED wiring separate from audio. Return coil/logic current to the supply star separately from audio returns.

## 5. Wire audio contacts and ground

Use the released pin-numbered contact matrix. Relay contact numbering is manufacturer-specific; a generic DPDT diagram is not enough.

| Function | Required result |
| --- | --- |
| Loop Off | Straight-through path; external send/return pair is not part of the active path |
| Loop A | Signal passes Loop A Send, external pedals, then Loop A Return |
| Loop B | Signal passes Loop B Send, external pedals, then Loop B Return |
| Loop A→B | Signal passes A Send/Return, then B Send/Return |
| Lane n selected | Instrument path goes to TO n; FX Return n/FX Send n pair is selected; other lane tips are isolated |
| Duplicate loop requested | Pre-FX has priority; that loop is not connected at Pre-TO |
| No DC power | Lane 1 passes audio; both loop positions bypass |

Join all TS sleeves at the planned single audio-ground point. Join the enclosure to audio ground once. Insulated jack bodies prevent extra sleeve/enclosure bonds. Keep coil and logic returns out of audio-signal return paths until the star point. Connect shields only at the end identified in the schematic. Inspect for solder bridges, cold joints, stray strands, exposed tips, and pinched wires.

## 6. Unpowered continuity checks

Disconnect the supply, instrument, pedals, and devices. Before power-on:

- [ ] Confirm the correct variant jack count, label, and physical direction on every jack.
- [ ] Remove metal swarf; inspect all solder joints, insulation, and wire strain relief.
- [ ] Confirm DC-jack polarity and no short between supply positive and ground.
- [ ] Check intended sleeve continuity and verify the enclosure has only the planned ground bond.
- [ ] Check switch/relay terminals against the released matrix; do not infer a relay state from coil pin placement.
- [ ] Verify the unpowered lane-1/both-loops-bypassed path and isolation of other lane tips.
- [ ] Verify Loop A then Loop B order and no path that connects a loop to both positions.
- [ ] Check that no active TO outputs or lane FX paths are joined.

## 7. Current-limited power-on and audio checks

1. Power from a current-limited bench supply set to the specified polarity/voltage. Set the current limit above the calculated startup requirement but below a level that could damage the wiring. Watch for unexpected current, heating, or odor; switch off immediately if observed.
2. Check the measured idle and maximum coil current against the design and supply rating. Confirm logic rail voltage and relay pickup/dropout.
3. Confirm startup indication: both loop positions Off and lane 1 selected. Cycle every footswitch slowly through all states and verify one transition per press and correct wraparound.
4. Use an instrument-level signal source/probe or an instrument and amplifier at low volume. Test Off, A, B, and A→B at both positions, one state at a time. Confirm bypass, loop order, and no intermittent connection.
5. On 2/4 Lane models, test each lane's TO and matching FX route separately; verify unselected lanes are isolated and lane selection never momentarily ties outputs together.
6. Request duplicate loop assignments and verify Pre-FX priority, Pre-TO applied-state LEDs, and no duplicate connection.
7. Listen for hum, crosstalk, crackle, and excessive switching pops. Correct grounding, contact, or wiring issues before increasing volume.
8. Remove DC power in a controlled test; verify lane 1 and loop bypass. Restore power and verify reset. Repeat with actual pedals/devices connected at low levels.

## 8. Final assembly and safety

Secure the board and wiring, insulate exposed terminals, add strain relief, and check that the lid cannot pinch conductors. Fit the lid and repeat footswitch, plug-clearance, polarity, and low-volume audio checks. Record the relay/IC/driver part numbers, measured current, schematic revision, and substitutions inside the build record. Label the enclosure with model, jack directions, lane numbers, and DC polarity. Do not connect speaker-level or mains signals to Trafik.
