# Hardware Design Description

Document ID: TRT-HWD-001
Version: 0.1
Date: 2026-10-30
Status: Draft
Owner: TBD

## Purpose

This document describes the hardware: its parts, interfaces, power, manufacturing and test. It is a reusable template for hardware projects, filled in for TRT.

## Scope

The carrier board and the daughter-board platform. Enclosure and the magnetic connector part are TBD.

## Definitions

Carrier, daughter board, HWID: as in REQUIREMENTS.md.
ERC: electrical rules check in the schematic editor.
DNP: do not populate.
Status markings: Implemented, Validated, Not Validated, Planned, TBD.

## References

TRT_HW repository: hardware/carrier-board (README and KiCad project) and hardware/daughter-board (KiCad project).
TRT_CHAINING_SPEC_v0.1.md, ARCHITECTURE.md, REQUIREMENTS.md.

## Revision History

Revision 0.1, 2026-10-30, TRT project: baseline from schematics and README.

## Approval

Author, reviewer, approver: names, signatures and dates TBD.

## Main Content

### Hardware status

Carrier schematics, five sheets: Implemented; no review record, so Not Validated.
Carrier PCB: placement started; Not Validated.
Carrier fabrication and bring-up: Planned.
Daughter-board KiCad project, seven sheets including the root: Implemented; the schematic editor ERC reported zero violations; electrically unreviewed, so Not Validated.
Daughter-board PCB: outline and mounting holes only.
Magnetic connector: Planned.

### Carrier board

Microcontroller: STM32G474RET6, LQFP-64, Cortex-M4F, 170 MHz.
Power input: USB-C 5 V with 5.1 kilohm resistors on CC1 and CC2.
3.3 V regulator: the README says AMS1117-3.3, but the power sheet shows AP2112K-3.3. TBD: confirm.
Clock: 8 MHz crystal with 22 pF load capacitors.
Reset: push button with a 10 kilohm pull-up and 100 nF.
Debug: Tag-Connect TC2030 carrying SWDIO on PA13, SWCLK on PA14, SWO on PB3, NRST, 3.3 V and GND.
Analog supply: ferrite bead and 100 nF for VDDA and VREF+.
Fabrication target: JLCPCB, four layers, 50 by 50 mm, 0805 passives, SOT-23, SOD-123 and LQFP-64.

### Carrier peripherals

I2C1 on PB9 (SDA) and PB8 (SCL), with 2.2 kilohm pull-ups switched by PC15. PC15 must never be driven low.
SPI1 on PA5 (SCK), PA6 (MISO), PA7 (MOSI) and PB6 (CS).
Motor driver: low-side AO3400A on PB4 (TIM3_CH1) with a flyback diode and output connector J1.
LEDs: a power LED on 3.3 V and a user LED on PA5.
USB full speed on PA11 and PA12.
Boot: PB8 shares BOOT0. Option bytes nBOOT0_SEL = 0 and nBOOT0 = 1 are required.

### Carrier connector J2 (16 pins, 2 by 8, 2.00 mm)

Pin 1 SPI_CS (PB6). Pin 2 GND. Pin 3 SPI_MOSI (PA7). Pin 4 +5V.
Pin 5 SPI_MISO (PA6). Pin 6 TRT_GPIO_1 (PA1). Pin 7 SPI_SCK (PA5). Pin 8 +3.3V.
Pin 9 HWID_0 (PC0). Pin 10 TRT_GPIO_2 (PA4). Pin 11 HWID_1 (PC1). Pin 12 SDA (PB9).
Pin 13 HWID_2 (PC2). Pin 14 SCL (PB8). Pin 15 HWID_3 (PC3). Pin 16 TRT_GPIO_3 (PB0).

The carrier PCB holds both a 2 by 8 socket on the left edge and a 2 by 8 header on the right edge. TBD: confirm how they are wired and used. Earlier notes mention a 20-pin connector while the README says 16. TBD: confirm.

### HWID on the carrier

Four lines with 10 kilohm pull-downs (R2 to R5). A separate HWID_A node is a 10 kilohm and 10 kilohm divider from 3.3 V (R6, R7). TBD: its purpose.

### Daughter-board platform

Schematic hierarchy, all Implemented and draft:
Root sheet (00_top): sheet symbols and notes.
01_connector: 2 by 8 connector and ESD arrays.
02_power: 5 V TVS, bulk capacitors, analog supply ferrite filter, test points.
03_hwid: four 0 ohm straps to 3.3 V.
04_bus: DNP I2C pull-ups, interrupt routing, DNP 2 by 6 expansion header.
10_module_potentiometers: eight potentiometers on two ADS1115 converters at 0x48 and 0x49.
11_module_keypad: 4 by 4 matrix with a diode per key on an MCP23017 at 0x20.
The potentiometer variant is active on the board; the keypad sheet is excluded from the board.

PCB skeleton: 50 by 50 mm outline at the carrier origin, four layers with the carrier stack-up and rules, four M2.5 holes at a 3.5 mm inset, no footprints placed.

Known mismatches to resolve:
Size: 4 by 4 MX keys and eight potentiometers do not fit 50 by 50 mm. A 100 by 100 mm panel tile on a 50 mm grid is recommended.
Holes: the carrier holes are not symmetric; the daughter board uses ideal positions.
Connector: needs the 20-position pinout of the chaining specification.
HWID codes: the draft codes are superseded by the chaining specification.

### Interfaces

Connector signals, power and grounding rules, HWID and descriptor are defined in TRT_CHAINING_SPEC_v0.1.md. Status: Planned.

### Power

+5 V from USB: budget TBD (500 mA assumed).
+3.3 V from the carrier regulator: budget TBD, depends on the regulator part.
Analog supply from 3.3 V through a ferrite: budget TBD.

### Module catalogue

Potentiometers, HWID 0x2: two ADS1115 converters. Implemented as a schematic.
Keypad 4 by 4, HWID 0x3: MCP23017. Implemented as a schematic.
Encoders, HWID 0x4: TBD. Planned.
Sliders, HWID 0x2: analog front end TBD. Planned.
OLED over I2C, HWID 0x5: TBD. Planned.
TFT over SPI, HWID 0x6: TBD, probably a module with its own microcontroller. Planned.
LED ring, HWID 0x7: I2C LED driver or microcontroller, with a 5 V level shifter. Planned.

### Manufacturing and test

Fabrication at JLCPCB with four layers: Planned.
Assembly: surface-mount on one side, through-hole controls by hand: Planned.
Bill of materials: TBD.
Test points on the daughter board: 3.3 V, 5 V, analog supply and GND. Implemented in the schematic.
Test fixture: HWID code 0xE is reserved. Planned.

### Carrier layout checklist

Compact motor loop. Power traces of 1.2 to 1.5 mm. One decoupling capacitor per VDD pin. Ferrite near pins 18 and 19. Guard ring around the crystal. A 90 ohm differential USB pair over an unbroken ground plane. Status: Planned.

## Future Placeholders

Bill of materials: TBD.
Thermal analysis: TBD.
EMC and ESD design notes: TBD.
Enclosure interface drawing: TBD.
Magnetic connector selection: TBD.
Fabrication and assembly notes: TBD.

## Appendices

Appendix A: schematic sheet index. Placeholder.
Appendix B: pin tables. Placeholder.
Appendix C: layout rules. Placeholder.
