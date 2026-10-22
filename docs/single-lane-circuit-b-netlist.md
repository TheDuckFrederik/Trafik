# Single Lane Trafik Rev B — component list and pin-by-pin connections

> **Authoritative for schematic drawing (2026-10-20).** Every pin of every component, its net, and every other pin on that net. Reference designators here are final and supersede the loose `Cin/Cout` labels in [single-lane-circuit-b.md](single-lane-circuit-b.md) §6. Relay pins use functional EN50005 names (11/21 = COM, 12/22 = NC, 14/24 = NO, A1/A2 = coil) — map to the TQ2-9V datasheet terminals before layout. This table set is also the verification baseline: the hand-drawn KiCad netlist will be diffed against it pin-for-pin.

Notation: `pin → NET`. When a net has other members they are listed in §3 (Net directory). NC = leave unconnected (place a no-connect flag in KiCad).

## 1. Component list

| Ref | Part | Value / package | Function |
| --- | --- | --- | --- |
| U1 | ATmega328P-PU | DIP-28 | MCU, 5 V, internal 8 MHz |
| U2 | LMC660AIN | DIP-14 | quad op-amp: buffers 1–3 + spare |
| U3 | 78L05 | TO-92 (1=IN, 2=GND, 3=OUT) | 5 V logic regulator |
| K1–K5 | TQ2-9V | DPDT non-latching, 9 V coil | loop inserts ×4 + output-buffer bypass |
| Q1–Q5 | 2N3904 | TO-92 (1=E, 2=B, 3=C) | coil low-side drivers |
| D1–D5 | 1N4148 | DO-35 (1=K, 2=A) | coil flyback |
| D6 | 1N5817 | DO-41 (1=K, 2=A) | reverse-polarity protection |
| LED1, LED5 | white 5 mm | (1=K, 2=A) | Off indication |
| LED2, LED6 | green 5 mm | | Loop A |
| LED3, LED7 | red 5 mm | | Loop B |
| LED4, LED8 | amber 5 mm | | A→B |
| SW1, SW2 | momentary SPST-NO stomp | 2-pin | Pre-FX / Pre-TO |
| J1–J8 | mono ¼″ TS, insulated | T / S | audio jacks |
| J9 | 2.1 mm DC jack | pin 1 = sleeve (+), pin 2 = center (−) | center-negative |
| J10 | 2×3 pin header | AVR ISP | programming |
| J11 | 1×2 pin header (optional) | self-test strap | PC3 to GND |
| R1, R5 | 470 Ω | white LED series |
| R2, R3, R4, R6, R7, R8 | 1 kΩ | colored LED series |
| R9–R13 | 4.7 kΩ | driver base series |
| R14–R18 | 100 kΩ | driver base pull-downs (fail-safe) |
| R19, R20 | 100 kΩ | VBIAS divider |
| R21, R22 | 1 kΩ | switch input series |
| R23 | 10 kΩ | RESET pull-up |
| R24–R32 | 1 MΩ | audio-node pulldowns (IN, PFX_MID, OUT3_IN, TO1, FXR1, FXS1, PRETO, LA_SEND, LB_SEND) |
| R33, R34 | 1 MΩ | loop-return jack-side pulldowns |
| R35–R37 | 1 MΩ | buffer output pulldowns |
| R38–R40 | 1 MΩ | buffer input bias → VBIAS |
| R41–R43 | 100 Ω | buffer output series |
| C1, C2 | 100 nF | switch pin filters |
| C3 | 100 nF | RESET |
| C4, C5 | 100 nF | VCC / AVCC decoupling |
| C6 | 100 nF | AREF |
| C7 | 10 µF ≥16 V | +5 V bulk (at U1) |
| C8 | 10 µF ≥16 V | VBIAS bypass (+ to VBIAS) |
| C9 | 100 nF | VBIAS HF |
| C10 | 100 nF | U2 rail decoupling |
| C11–C13 | 1 µF film | buffer input coupling |
| C14–C16 | 1 µF film | buffer output coupling |
| C19 | 100 µF ≥16 V | 9V_PROT bulk (+ to rail) |
| C20 | 100 nF | 9V_PROT HF |
| C21 | 100 nF | +5 V at U3 output |

(C17–C18 intentionally unused.)

## 2. Pin connections, component by component

### U1 — ATmega328P-PU

| Pin | Port | → Net |
| ---: | --- | --- |
| 1 | PC6/RESET | RESET |
| 2 | PD0/RXD | NC (spare — optional debug header) |
| 3 | PD1/TXD | NC (spare) |
| 4 | PD2 | SW1_MCU |
| 5 | PD3 | SW2_MCU |
| 6 | PD4 | DRV_K1 |
| 7 | VCC | +5V |
| 8 | GND | GND |
| 9 | PB6 | NC (spare) |
| 10 | PB7 | NC (spare) |
| 11 | PD5 | DRV_K2 |
| 12 | PD6 | DRV_K3 |
| 13 | PD7 | DRV_K4 |
| 14 | PB0 | DRV_K5 |
| 15 | PB1 | LED1_D |
| 16 | PB2 | LED2_D |
| 17 | PB3 | LED3_D *(also ISP MOSI — J10.4)* |
| 18 | PB4 | LED4_D *(also ISP MISO — J10.1)* |
| 19 | PB5 | LED5_D *(also ISP SCK — J10.3)* |
| 20 | AVCC | +5V |
| 21 | AREF | AREF |
| 22 | GND | GND |
| 23 | PC0 | LED6_D |
| 24 | PC1 | LED7_D |
| 25 | PC2 | LED8_D |
| 26 | PC3 | SELFTEST |
| 27 | PC4 | NC (I²C roadmap) |
| 28 | PC5 | NC (I²C roadmap) |

### U2 — LMC660AIN

| Pin | → Net | Role |
| ---: | --- | --- |
| 1 | BUF1_O | buffer 1 output (feedback ties pin 2 here) |
| 2 | BUF1_O | follower feedback |
| 3 | BUF1_IN | buffer 1 input |
| 4 | 9V_PROT | V+ |
| 5 | BUF2_IN | buffer 2 input |
| 6 | BUF2_O | follower feedback |
| 7 | BUF2_O | buffer 2 output |
| 8 | BUF3_O | buffer 3 output |
| 9 | BUF3_O | follower feedback |
| 10 | BUF3_IN | buffer 3 input |
| 11 | GND | V− |
| 12 | VBIAS | spare amp +in, parked |
| 13 | SPARE_FB | spare amp feedback |
| 14 | SPARE_FB | spare amp output, unloaded |

### U3 — 78L05

| Pin | → Net |
| ---: | --- |
| 1 (IN) | 9V_PROT |
| 2 (GND) | GND |
| 3 (OUT) | +5V |

### K1–K5 — TQ2-9V (functional pins)

| Ref | 11 (COM_A) | 12 (NC_A) | 14 (NO_A) | 21 (COM_B) | 22 (NC_B) | 24 (NO_B) | A1 | A2 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| K1 | IN_TIP | LINK_K1 | LA_SEND | PFX_MID | LINK_K1 | BUF1_OUT | K1_COIL_LO | 9V_PROT |
| K2 | PFX_MID | LINK_K2 | LB_SEND | OUT3_IN | LINK_K2 | BUF2_OUT | K2_COIL_LO | 9V_PROT |
| K3 | FXR1_TIP | LINK_K3 | LA_SEND | PRETO_MID | LINK_K3 | BUF1_OUT | K3_COIL_LO | 9V_PROT |
| K4 | PRETO_MID | LINK_K4 | LB_SEND | FXS1_TIP | LINK_K4 | BUF2_OUT | K4_COIL_LO | 9V_PROT |
| K5 | OUT3_IN | LINK_K5 | BUF3_SRC | TO1_TIP | LINK_K5 | BUF3_OUT | K5_COIL_LO | 9V_PROT |

### Q1–Q5 — 2N3904 (1=E, 2=B, 3=C)

All: pin 1 → GND, pin 2 → `Kn_BASE`, pin 3 → `Kn_COIL_LO` (Q1→K1 … Q5→K5).

### D1–D6 — diodes (1=K, 2=A)

D1–D5 (flyback): pin 1 → 9V_PROT, pin 2 → `Kn_COIL_LO` (D1→K1 … D5→K5).
D6 (reverse protection): pin 1 → 9V_PROT, pin 2 → PSU_9V_RAW.

### LEDs (1=K, 2=A) and resistors

All LEDs: pin 2 → `LEDn_A`, pin 1 → GND. Rn: pin 1 → `LEDn_D`, pin 2 → `LEDn_A` (R1→LED1 … R8→LED8).

### Switches and jacks

| Ref | Pin 1 | Pin 2 |
| --- | --- | --- |
| SW1 | SW1_N | GND |
| SW2 | SW2_N | GND |
| J9 | PSU_9V_RAW (sleeve) | GND (center) |

| Jack | T (tip) | S (sleeve) |
| --- | --- | --- |
| J1 Instrument IN | IN_TIP | GND |
| J2 TO 1 | TO1_TIP | GND |
| J3 FX Send 1 | FXS1_TIP | GND |
| J4 FX Return 1 | FXR1_TIP | GND |
| J5 Loop A Send | LA_SEND | GND |
| J6 Loop A Return | LA_RET | GND |
| J7 Loop B Send | LB_SEND | GND |
| J8 Loop B Return | LB_RET | GND |

### Resistors (pin 1 → pin 2)

| Ref | Pin 1 | Pin 2 |
| --- | --- | --- |
| R1 | LED1_D | LED1_A |
| R2 | LED2_D | LED2_A |
| R3 | LED3_D | LED3_A |
| R4 | LED4_D | LED4_A |
| R5 | LED5_D | LED5_A |
| R6 | LED6_D | LED6_A |
| R7 | LED7_D | LED7_A |
| R8 | LED8_D | LED8_A |
| R9 | DRV_K1 | K1_BASE |
| R10 | DRV_K2 | K2_BASE |
| R11 | DRV_K3 | K3_BASE |
| R12 | DRV_K4 | K4_BASE |
| R13 | DRV_K5 | K5_BASE |
| R14 | K1_BASE | GND |
| R15 | K2_BASE | GND |
| R16 | K3_BASE | GND |
| R17 | K4_BASE | GND |
| R18 | K5_BASE | GND |
| R19 | 9V_PROT | VBIAS |
| R20 | VBIAS | GND |
| R21 | SW1_N | SW1_MCU |
| R22 | SW2_N | SW2_MCU |
| R23 | +5V | RESET |
| R24 | IN_TIP | GND |
| R25 | PFX_MID | GND |
| R26 | OUT3_IN | GND |
| R27 | TO1_TIP | GND |
| R28 | FXR1_TIP | GND |
| R29 | FXS1_TIP | GND |
| R30 | PRETO_MID | GND |
| R31 | LA_SEND | GND |
| R32 | LB_SEND | GND |
| R33 | LA_RET | GND |
| R34 | LB_RET | GND |
| R35 | BUF1_OUT | GND |
| R36 | BUF2_OUT | GND |
| R37 | BUF3_OUT | GND |
| R38 | VBIAS | BUF1_IN |
| R39 | VBIAS | BUF2_IN |
| R40 | VBIAS | BUF3_IN |
| R41 | BUF1_O | BUF1_RC |
| R42 | BUF2_O | BUF2_RC |
| R43 | BUF3_O | BUF3_RC |

### Capacitors (pin 1 → pin 2)

| Ref | Pin 1 | Pin 2 |
| --- | --- | --- |
| C1 | SW1_MCU | GND |
| C2 | SW2_MCU | GND |
| C3 | RESET | GND |
| C4 | +5V | GND |
| C5 | +5V | GND |
| C6 | AREF | GND |
| C7 (+) | +5V | GND |
| C8 (+) | VBIAS | GND |
| C9 | VBIAS | GND |
| C10 | 9V_PROT | GND |
| C11 | LA_RET | BUF1_IN |
| C12 | LB_RET | BUF2_IN |
| C13 | BUF3_SRC | BUF3_IN |
| C14 | BUF1_RC | BUF1_OUT |
| C15 | BUF2_RC | BUF2_OUT |
| C16 | BUF3_RC | BUF3_OUT |
| C19 (+) | 9V_PROT | GND |
| C20 | 9V_PROT | GND |
| C21 | +5V | GND |

### Headers

| Ref | 1 | 2 | 3 | 4 | 5 | 6 |
| --- | --- | --- | --- | --- | --- | --- |
| J10 (ISP 2×3) | LED4_D (MISO) | +5V | LED5_D (SCK) | LED3_D (MOSI) | RESET | GND |
| J11 (self-test) | SELFTEST | GND | — | — | — | — |

## 3. Net directory — every net and every pin on it

This is the "more than one" answer: for each net, all pins that share it.

| Net | Pins |
| --- | --- |
| **GND** | U1.8, U1.22, U2.11, U3.2, SW1.2, SW2.2, J1–J8.S, J9.2, J10.6, J11.2, Q1–Q5.1, LED1–LED8.1, R14–R18.2, R20.2, R24–R37.2, C1–C6.2, C7.2, C9.2, C10.2, C19.2, C20.2, C21.2 |
| **9V_PROT** | D6.1, R19.1, U2.4, U3.1, C10.1, C19.1(+), C20.1, K1.A2, K2.A2, K3.A2, K4.A2, K5.A2, D1.1, D2.1, D3.1, D4.1, D5.1 |
| **+5V** | U1.7, U1.20, U3.3, R23.1, J10.2, C4.1, C5.1, C7.1(+), C21.1 |
| **VBIAS** | R19.2, R20.1, C8.1(+), C9.1, R38.1, R39.1, R40.1, U2.12 |
| **PSU_9V_RAW** | J9.1, D6.2 |
| **RESET** | U1.1, R23.2, C3.1, J10.5 |
| **AREF** | U1.21, C6.1 |
| **IN_TIP** | J1.T, K1.11, R24.1 |
| **PFX_MID** | K1.21, K2.11, R25.1 |
| **OUT3_IN** | K2.21, K5.11, R26.1 |
| **TO1_TIP** | K5.21, J2.T, R27.1 |
| **FXR1_TIP** | J4.T, K3.11, R28.1 |
| **FXS1_TIP** | K4.21, J3.T, R29.1 |
| **PRETO_MID** | K3.21, K4.11, R30.1 |
| **LA_SEND** | K1.14, K3.14, J5.T, R31.1 |
| **LB_SEND** | K2.14, K4.14, J7.T, R32.1 |
| **LA_RET** | J6.T, R33.1, C11.1 |
| **LB_RET** | J8.T, R34.1, C12.1 |
| **LINK_K1…K5** | Kn.12 + Kn.22 (two relay pins each, bypass bridges) |
| **BUF1_IN** | C11.2, R38.2, U2.3 |
| **BUF2_IN** | C12.2, R39.2, U2.5 |
| **BUF3_IN** | C13.2, R40.2, U2.10 |
| **BUF1_O** | U2.1, U2.2, R41.1 |
| **BUF2_O** | U2.6, U2.7, R42.1 |
| **BUF3_O** | U2.8, U2.9, R43.1 |
| **BUF1_RC / 2 / 3** | R4n.2 + C1(3+n).1 (series junctions) |
| **BUF1_OUT** | C14.2, R35.1, K1.24, K3.24 |
| **BUF2_OUT** | C15.2, R36.1, K2.24, K4.24 |
| **BUF3_OUT** | C16.2, R37.1, K5.24 |
| **BUF3_SRC** | K5.14, C13.1 |
| **K1…K5_COIL_LO** | Kn.A1, Qn.3, Dn.2 |
| **K1…K5_BASE** | R(8+n).2, R(13+n).1, Qn.2 |
| **DRV_K1…K5** | U1 pin (6/11/12/13/14), R(8+n).1 |
| **SW1_N** | SW1.1, R21.1 |
| **SW1_MCU** | U1.4, R21.2, C1.1 |
| **SW2_N** | SW2.1, R22.1 |
| **SW2_MCU** | U1.5, R22.2, C2.1 |
| **LEDn_D** | U1 pin (15–16, 17, 18, 19, 23, 24, 25), Rn.1; LED3/4/5_D also J10.4 / J10.1 / J10.3 |
| **LEDn_A** | Rn.2, LEDn.2 |
| **SELFTEST** | U1.26, J11.1 |
| **SPARE_FB** | U2.13, U2.14 |
