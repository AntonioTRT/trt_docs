# System Architecture

Document ID: TRT-ARC-001
Version: 0.1
Date: 2026-10-30
Status: Draft
Owner: TBD

## Purpose

This document describes the structure of the system, the decisions behind it and the state of each part. Detailed hardware and software design are in HWDD.md and SWDD.md. Chaining decisions are in TRT_CHAINING_SPEC_v0.1.md. The structure is reusable for other projects.

## Scope

The whole TRT system: host tool, protocol, firmware runtime, carrier board, daughter-board platform and chaining.

## Definitions

TRT, TRT-Core, TRT-CLI, carrier, daughter board, HWID: as in REQUIREMENTS.md.
Runtime: the board-specific layer that supplies transport, clock and profile to TRT-Core.
Segment: a point-to-point I2C link between a downstream port and the one thing connected to it.

Status markings: Implemented, Validated, Not Validated, Planned, TBD.

## References

REQUIREMENTS.md, HWDD.md, SWDD.md, TEST_PLAN.md, RISK_REGISTER.md, TRT_CHAINING_SPEC_v0.1.md.
TRT-core documentation: protocol-v0.1-wire-format, command-identifiers, error-model, event-system, platform-status.

## Revision History

Revision 0.1, 2026-10-30, TRT project: baseline from the repository state.

## Approval

Author, reviewer, approver: names, signatures and dates TBD.

## Main Content

### System context

The PC runs TRT-CLI. It talks over USB serial using the TRT protocol to a board runtime. The runtime hosts the shared TRT-Core. TRT-Core drives the hardware: a carrier board and daughter modules.

Repositories: TRT-core (firmware), TRT-cli (host), TRT_HW (hardware, KiCad), trt_docs (documentation).

### Architectural principles

One TRT-Core and one protocol for every platform; platforms differ only by compile-time flags and a thin runtime layer. Status: Implemented.
The board is autonomous: the main loop never blocks and the board runs with no host. Status: Implemented in design; Not Validated on hardware.
Commands ask for intent and do not drive hardware directly; a state machine keeps behaviour deterministic. Status: Implemented.
Capabilities are declared by the board; the host does not filter by board. Status: Implemented.
Validation order: simulator, Arduino, then ESP32 and STM32. Status: In progress.
The host is layered: commands, services, protocol, transport. Status: Implemented.

### TRT-Core

Components: frame parser and encoder, dispatcher with command registry, BoardContext (board information, capabilities, uptime clock), SystemService (all system commands, registered once), HeartbeatService, capability registry, ModuleManager and hardware interfaces (gpio, adc, dac, pwm, i2c, spi, uart, lcd, neopixel, relay, storage).
Status: Implemented. The C++ was not compiled in the current environment, so it is Not Validated.
Memory model: STL and dynamic memory on ESP32, STM32 and the simulator; static memory on Arduino Uno with a 50-byte payload buffer and a 12-entry handler table.

### Protocol version 0.1

The wire format is frozen. A frame has SYNC 0xAA55, VERSION 0x01, FLAGS, BOARD_ID, SEQ_ID, COMMAND, LENGTH, PAYLOAD and CRC16, with 14 bytes of overhead, big-endian.
Commands: 0x0001 PING, 0x0002 FW_VERSION, 0x0003 BOARD_VERSION, 0x0004 INFO, 0x0005 STATUS, 0x0006 CAPABILITIES, 0x0007 BUILD_ID.
Extensions: 0x0008 UPTIME, 0x0009 RUNTIME, 0x000A ACTIVE_PROGRAM, and INFO field 0x06 (family).
Responses: 0x8001 ACK, 0x8002 NACK, 0x8003 DATA.
Events: 0x9001 HEARTBEAT with FLAGS 0x02.
Errors: 0x0001 to 0x0006.
CRC: the field is present and holds 0x0000; the algorithm is Planned.

### Platforms

Arduino Uno, board ID 101: Implemented. Validated at an earlier milestone (2026-09-08), before the shared refactor; now Not Validated.
ESP32, board ID 301: Implemented. Not Validated; no hardware run recorded.
STM32 with CubeIDE and multiple applications, board ID 201: Implemented. Not Validated; never built.
STM32 with PlatformIO (legacy), board ID 2: Implemented. The board on COM20 answers six commands with an older image.
Simulator: Implemented. Not Validated in the current environment.
RP2040: Planned.

### Hardware architecture

Carrier board: STM32G474RET6, USB-C power, 3.3 V regulator, SWD, I2C1 with switchable pull-ups, SPI1, three GPIO, 4-bit HWID and a 16-pin connector. Status: Implemented as schematics; not fabricated.
Daughter-board platform: a common core (connector, power, HWID, bus) plus variant sheets. First variants: potentiometers and a mechanical keypad. Status: Implemented as a KiCad skeleton; Not Validated.

### Chaining architecture (summary)

The USB-attached carrier is the master. Other carriers are slaves. Each link is a point-to-point I2C segment with one master. A slave looks like a self-describing module to its upstream. Frames are routed by BOARD_ID, with path IDs for slave nodes. Status: Planned. Full detail is in TRT_CHAINING_SPEC_v0.1.md.

### Host architecture

Layers: commands (Typer), services (discovery, board query, build, flash, programs), protocol (v01) and transport (serial). Discovery uses two probes: PING plus INFO, then BOARD_VERSION, FW_VERSION, CAPABILITIES and BUILD_ID for older boards. Status: Implemented; Validated on the host and against the board on COM20.

### Build and deployment

Arduino and ESP32: tools/upload.py builds an isolated PlatformIO project and uploads it. Implemented; ESP32 Not Validated.
STM32 applications: trt build stm32 (CMake and arm-none-eabi-gcc), trt flash stm32 (STM32CubeProgrammer). Implemented; Not Validated.
Legacy STM32: pio run and trt flash stm32 --platformio. Implemented.
Simulator and tests: CMake. Implemented; not run in the current environment.

### Decision log

Decisions 1 to 11 are inherited from the earlier documentation: autonomous board, intent-based commands, frozen frame, no peripheral commands in version 0.1, board-declared capabilities, validation order, one TRT-Core with flags, CubeIDE builds with TRT-Core as a library, isolated upload projects, layered CLI, and option bytes for PB8 and BOOT0. Dates: TBD.
Decision 12: SystemService registers all system commands once (October 2026).
Decision 13: one STM32 project with selectable applications (October 2026).
Decision 14: daughter boards reuse the carrier 16-pin interface and 4-bit HWID (October 2026).
Decision 15: chaining by I2C segments with path IDs (October 2026, proposed).

### Quality attributes

Portability: Implemented. Determinism: Implemented. Testability: partly, see TEST_PLAN.md. Maintainability: known debt, the byte-to-frame state machine is copied in four runtimes. Safety and security: TBD.

### Known gaps

C++ never compiled in the current environment. Heartbeat never observed. CRC16 not computed. No module drivers. No chaining. Mating geometry open.

## Future Placeholders

Performance targets: TBD.
Security model: TBD.
Firmware update strategy: TBD.
Module catalogue: TBD.
Enclosure and mechanical interface: TBD.

## Appendices

Appendix A: block diagrams. Placeholder.
Appendix B: interface definitions. Placeholder.
Appendix C: decision records. Placeholder.
