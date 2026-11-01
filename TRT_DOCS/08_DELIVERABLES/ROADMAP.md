# Roadmap

Document ID: TRT-ROD-001
Version: 0.1
Date: 2026-10-30
Status: Draft
Owner: TBD

## Purpose

This document gives the plan: where the project is, the phases, the milestones, the near-term plan and the stopping points. It is a reusable template for project planning, filled in for TRT.

## Scope

All work needed to reach a working modular TRT hardware platform: STM32 carrier, USB master, chained slave mode, magnetic board-to-board connectors, I2C between boards, and daughter modules identified by HWID.

## Definitions

Milestone: a point with an objective pass condition.
Gate: a check that must pass before the next phase starts.
Status markings: Implemented, Validated, Not Validated, Planned, TBD.

## References

REQUIREMENTS.md, ARCHITECTURE.md, TEST_PLAN.md, RISK_REGISTER.md, TRT_CHAINING_SPEC_v0.1.md.

## Revision History

Revision 0.1, 2026-10-30, TRT project: baseline.

## Approval

Author, reviewer, approver: names, signatures and dates TBD.

## Main Content

### Current state

Protocol version 0.1, discovery, query commands and heartbeat: Implemented; the host side is Validated and the firmware is Not Validated.
Arduino, ESP32 and STM32 runtimes: Implemented; Not Validated since the shared refactor.
STM32 multi-application workspace and the build and flash commands: Implemented; Not Validated because the toolchain is missing.
Carrier board: schematics done, PCB in placement.
Daughter board: KiCad skeleton with two variants, layout not started.
Chaining: specification drafted, nothing implemented.
Documentation baseline: created.

### Phase history

Core and simulator: parser, encoder, dispatcher and services. Implemented.
Arduino: the Uno answered through the CLI on 2026-09-08. Validated before the refactor.
Cross-platform alignment and query commands: TRT-core commits 8865ee1 and b0891c4, TRT-cli commits 501494f and f7182f7. Implemented.
Daughter-board KiCad skeleton: in TRT_HW, uncommitted when this document was written. Implemented.
Chaining specification and documentation baseline: Implemented.

### Milestones

M0, documentation and chaining approval. Done when the approval sections are signed and open decisions O-01 to O-04 and O-12 are settled. Planned.
M1, firmware proven on the Nucleo. Done when program_blink is built and flashed, and discovery, queries and heartbeat are seen on COM20. Planned.
M2, platforms re-checked. Done when Uno and ESP32 are rebuilt, Uno memory is measured and the C++ simulator tests pass. Planned.
M3, carrier PCB ready. Done when the connector change is decided, layout is complete, design rules pass and the order is placed. Planned.
M4, two-board I2C. Done when a master and a slave carrier exchange PING and INFO over I2C and the CLI queries the slave. Planned.
M5, first module. Done when a module is detected by HWID, its descriptor is read and its events reach the CLI. Planned.
M6, magnetic connector validated. Done when contact bounce and hot-plug are tested on the real connector. Planned.
M7, module set. Done when an encoder, slider or LED ring works on the same contract. Planned.

### Thirty-day plan

Week 1: install the toolchain, compile, fix errors, flash program_blink and run the COM20 procedure. Gate M1.
Week 2: re-run the simulator tests, rebuild Uno and ESP32, settle the open decisions and approve the chaining specification. Gates M0 and M2.
Week 3: update the carrier connector and schematic if needed and start carrier layout. Gate: carrier decision recorded.
Week 4: build the I2C link layer on two Nucleo boards with jumper wires, without a new PCB. Gate: the link works at 100 kHz.

### Ninety-day plan

Days 31 to 45: frame forwarding, path ID routing, CRC16 in parser and encoder, and a CLI topology view. Gate M4.
Days 46 to 60: HWID read, descriptor, port state machine, and the carrier PCB ordered.
Days 61 to 75: first module (keypad or potentiometers): layout, order and bring-up. Gate M5.
Days 76 to 90: magnetic connector tests, hot-plug validation and the start of the next module. Gate M6.

### Stopping points

After week 1: if the firmware does not run on the Nucleo, fix that before any new feature.
Before layout: the chaining decisions are signed.
After M4: modules do not start until chained queries are stable, including forced disconnects.
After M6: if bounce or hot-plug breaks the bus, fix the hardware before more firmware.
After M5: the first module should need no change outside its driver; if it does, fix the contract first.

### Dependencies

Toolchain and instrument availability, connector part selection, fabrication lead time (TBD), and the owner decisions listed in the chaining specification.

### Not planned for now

More query commands, RP2040 support, merging the two STM32 runtimes, and removing the frame parser duplication (only if framing changes).

## Future Placeholders

Team, dates, effort and budget: TBD.
Release plan and versioning: TBD.
Manufacturing ramp: TBD.
Certification plan: TBD.

## Appendices

Appendix A: detailed schedule. Placeholder.
Appendix B: dependency map. Placeholder.
Appendix C: resource plan. Placeholder.
