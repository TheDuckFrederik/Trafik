# Trafik fabrication and build guide

This guide covers the shared assembly process for all three variants. Use the model-specific jack counts and panel arrangement in the variant documents and the [BOM](bom-PLACEHOLDER.md). Trafik carries instrument-level audio only. Do not connect speaker outputs or mains voltage to the unit.

## 1. Confirm the design and parts

Before buying or drilling:

1. Select the variant and confirm its number of TO and FX jacks.
2. Select the exact enclosure, jacks, relay, footswitches, LEDs and DC jack. Download their dimension drawings and datasheets.
3. Verify that the chosen relay contact form and board footprint implement the routing matrix and bypass/interlock behavior in the [architecture](architecture.md). The BOM gives quantities and cost allowances, not an interchangeable relay pinout.
4. Prepare a full-size panel drawing using the actual component dimensions. Check jack nuts, washers, socket plugs, foot clearance, internal depth, wire bend radius, PCB standoffs and enclosure lid.
5. Confirm that the isolated 9 V supply can supply the measured total relay-coil and logic current. Use a regulated, center-negative supply only if the completed power input is wired for that polarity.

Do not proceed with a guessed relay pinout or an unverified control-board schematic. Mark relay coil and normally-open/normally-closed contact terminals from the exact manufacturer datasheet.

## 2. Enclosure layout and drilling

1. Remove the enclosure lid and protect the finish with masking tape.
2. Mark the perimeter and center of every panel component from the dimensioned drawing. Keep the physical layout: Instrument IN on the right, FX Return bank upper left, FX Send bank upper right, Loop B above Loop A on the left, TO bank along the bottom, and the switches across the middle.
3. Place the center device switch only on the 2- and 4-lane models. Leave clear space for each state-indicator group beside the TO bank and between the FX banks.
4. Check for internal collisions before drilling: relay/control PCB, lid ribs, jack bodies, switch bodies, wiring bundles and nuts must all clear one another.
5. Center-punch the marks. Drill small pilot holes first, then enlarge with a step drill or correctly sized bit. Use the component maker’s panel cutout dimensions; do not assume all ¼-inch jacks or stomp switches share a mounting-hole size.
6. Deburr both sides, remove all metal swarf, and clean the enclosure before installing electronics. Metal chips can short relay contacts or the control PCB.
7. If using metal-body jacks, use insulating shoulder washers where specified to prevent unintended sleeve-to-enclosure bonds. Establish one deliberate ground-bonding scheme for all signal sleeves and the enclosure.

The listed enclosure families are estimates, not guaranteed fit. A crowded 4-lane panel should be checked with actual plugs inserted and switches operated before committing to a finish.

## 3. Install panel hardware

1. Fit the audio jacks loosely and orient their solder lugs for accessible, short wiring.
2. Label each jack before wiring: Instrument IN; TO n; FX Send n; FX Return n; Loop A Send/Return; Loop B Send/Return. Labels must follow the Trafik-side direction convention.
3. Tighten jacks without rotating their bodies or damaging insulating washers.
4. Install the momentary normally-open footswitches. The outer switches are Pre-FX and Pre-TO; the middle switch is Device on the 2/4-lane units.
5. Install the DC jack with clearance from all audio terminals. Mark its polarity on the board and inside the enclosure.
6. Install LEDs with the correct polarity and fit retaining hardware. Label loop states Off, A, B, A→B; label device LEDs 1…N.
7. Fit the control/relay board on insulating standoffs. Keep relay contact terminals physically separate from the DC and counter wiring.

## 4. Wire the control and power circuit

1. Wire the footswitch contacts only to the debounced CMOS counter inputs; footswitches must not carry audio.
2. Configure each loop-position counter for the four-state sequence Off → A → B → A+B → Off. Configure the lane counter for 1…N and wrap at the number of lanes.
3. Add a defined power-up reset to the counters: loop positions Off, device lane 1. Do not leave CMOS inputs floating.
4. Connect counter decoders to appropriately rated transistor relay drivers. Fit coil suppression diodes or the protection specified by the relay/driver design, respecting coil polarity.
5. Drive the state LEDs from the applied-state decoder after loop-allocation interlock, so the display reflects the audio contacts rather than only the requested counter state.
6. Connect the DC input to the protected 9 V control rail with reverse-polarity protection and local decoupling. Confirm the completed board’s current draw before connecting an external supply.
7. Keep logic/coil wiring away from high-impedance audio input wiring. Twist relay coil pairs where practical and route audio at the other side of the enclosure.

## 5. Wire the audio signal path

Work from a verified routing table or schematic for the exact relay used. Relay terminal numbers differ between models. The following is the required functional contact behavior, not a substitute for pin numbers:

| State | Required audio behavior |
| --- | --- |
| Loop Off | Signal bypasses the external send/return pair; send/return are isolated from the active signal path |
| Loop A | Signal passes through Loop A Send, external pedal chain, then Loop A Return |
| Loop B | Signal passes through Loop B Send, external pedal chain, then Loop B Return |
| A→B | Signal passes Loop A Send/Return first, then Loop B Send/Return |
| Lane n | Instrument path connects to TO n; FX Return n and FX Send n connect as the selected device FX path; all other lanes are isolated |

1. Wire signal grounds/sleeves according to the chosen grounding scheme. Avoid multiple accidental enclosure bonds.
2. Use short insulated conductors for audio. Use shielded cable for long or noise-sensitive runs; connect shields at the planned ground point only.
3. Wire the loop bypass/selection relay contacts, then test each loop state before adding device-lane wiring.
4. Wire each lane’s TO and matching FX pair to the lane-selection relay group. Ensure TO and the corresponding FX pair change together and use break-before-make switching.
5. Wire the duplicate-loop interlock so a physical loop cannot be connected at both positions simultaneously. Give Pre-FX the specified priority and feed the applied state back to the indicator decoder.
6. Separate contact wiring from relay coils, CMOS clock/reset lines and LED conductors. Cross audio and control bundles at right angles if crossing is unavoidable.
7. Inspect every joint for solder bridges, loose strands, cold joints, and exposed conductor that could touch the enclosure or an adjacent relay terminal.

## 6. Initial electrical checks

Do not connect an amplifier, pedal, instrument or power supply until completing the unpowered checks.

- [ ] Confirm all jack labels and model-specific jack counts.
- [ ] Confirm no solder bridges, clipped leads, stray wire strands or metal swarf remain.
- [ ] With a multimeter, verify no short between the 9 V rail and ground.
- [ ] Verify DC jack polarity, reverse-protection orientation and continuity to the control-board supply pins.
- [ ] With the unit unpowered, check that audio sleeves have the intended ground continuity and no unintended sleeve-to-tip shorts.
- [ ] Verify all normally closed/unpowered relay contacts give the intended bypass/passive lane-1 condition.
- [ ] Check each TS jack tip and sleeve against the routing table; confirm unselected outputs are not shorted to active outputs.
- [ ] Check loop A then loop B ordering in the A+B state.
- [ ] Request the same loop at both positions and verify the interlock assigns it to Pre-FX and reports the applied state on the LEDs.
- [ ] Verify no two device lanes have continuity between their TO outputs or FX paths.

## 7. Power-on and signal tests

1. Power the unit from a current-limited bench supply set to 9 V with the correct polarity. Start with a conservative current limit; investigate unexpected draw or heating rather than raising the limit blindly.
2. Confirm startup state: both loop positions Off and, where present, lane 1 selected. Check the matching LEDs.
3. Press each footswitch slowly through its complete cycle and verify one state transition per press, correct wraparound and no skipped/double counts.
4. Listen using an instrument-level test source and amplifier at low volume, or use an audio continuity/probe tool. Verify bypass, A, B, A→B, and bypass again at each position.
5. Test each device lane separately. Confirm its TO and matching FX pair work, and all other lanes remain isolated.
6. Test the allocation interlock with each loop requested at both positions. Verify the signal appears only at the priority position and LEDs indicate the applied routing.
7. Check for hum, oscillation, excessive switching pops, crosstalk, intermittent jacks and noise that changes when control/coil wires move. Correct grounding or wiring faults before increasing test volume.
8. Remove DC power during a controlled test and confirm the relay default state is safe and predictable. Restore power and confirm reset to Off/lane 1.
9. Repeat the checks with actual pedal chains and each device connected, initially with levels low.

## 8. Final assembly and verification

1. Secure the PCB, add strain relief to wiring, and ensure no conductor is pinched by the lid.
2. Fit the lid and repeat footswitch operation, plug insertion, DC polarity and low-volume audio checks.
3. Record the actual relay model, control-board revision, supply current, and any substitutions on the build record.
4. Mark the finished enclosure with the model name, jack directions, loop labels, lane numbers and 9 V DC polarity.
5. Before regular use, confirm no speaker-level signal is connected to any Trafik jack.
