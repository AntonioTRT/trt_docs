# Requirements Specification

Document ID: TRT-REQ-001
Version: 0.1
Date: 2026-10-30
Status: Draft
Owner: TBD

## Purpose

This document states what the system must do and the constraints it works under. It is the reference for design, test and acceptance. It is written as a reusable template: the structure applies to any engineering project, and the content below is filled in for TRT.

## Scope

In scope for TRT: the host command-line tool, the firmware runtime and its protocol, the carrier board, the daughter-board platform, and chaining of carriers.

Out of scope: enclosure design, marketing, cloud services (TBD).

## Definitions

TRT: Tool Runtime Terminal, a framework that controls embedded hardware from a PC.
TRT-Core: the portable C++ runtime that runs inside boards.
TRT-CLI: the Python command-line tool that runs on the PC.
Carrier board: the STM32G474 board that runs TRT-Core.
Daughter board or module: a function board attached to a carrier port.
HWID: the 4-bit hardware identification code read on the connector.
Master, slave: roles of carriers in a chain (see the chaining specification).

Status markings used in every document:
Implemented: the work exists in a repository.
Validated: implemented and verified by a recorded test.
Not Validated: implemented but not verified, or verified only in part.
Planned: not started.
TBD: information or decision not yet available.

## References

TRT-core documentation: protocol-v0.1-wire-format, command-identifiers, commands-v0.1, error-model, event-system, platform-status.
TRT_HW: carrier-board README and KiCad projects, daughter-board KiCad project.
TRT_CHAINING_SPEC_v0.1.md in this folder.
ARCHITECTURE.md, HWDD.md, SWDD.md, TEST_PLAN.md in this folder.

## Revision History

Revision 0.1, 2026-10-30, TRT project: baseline created from the repository state.
Revision: TBD.

## Approval

Author: name, signature and date TBD.
Reviewer: name, signature and date TBD.
Approver: name, signature and date TBD.

## Main Content

### How requirements are written

Each requirement has an ID in the form REQ-AREA-NNN, a priority (Must, Should, Could), a status marking, and a verification method (Test, Inspection, Analysis, Demonstration). Areas: PRO protocol, FW firmware, CLI host, HW hardware, CHN chaining, NFR non-functional.

### Protocol requirements

REQ-PRO-001 (Must): Frames use SYNC 0xAA55, VERSION, FLAGS, BOARD_ID, SEQ_ID, COMMAND, LENGTH, PAYLOAD and CRC16, big-endian. Status: Validated (simulator tests and the Arduino Uno milestone of 2026-09-08, before the shared refactor). Verification: Test.

REQ-PRO-002 (Must): Boards implement PING, FW_VERSION, BOARD_VERSION, INFO, STATUS, CAPABILITIES and BUILD_ID. Status: Not Validated. The shared SystemService implements them; the changes were never compiled or run.

REQ-PRO-003 (Must): Unsupported commands return NACK with ERROR_UNSUPPORTED_COMMAND. Status: Not Validated.

REQ-PRO-004 (Should): Boards provide UPTIME, RUNTIME and ACTIVE_PROGRAM. Status: Not Validated. Implemented but never compiled.

REQ-PRO-005 (Should): A board emits HEARTBEAT (command 0x9001, FLAGS 0x02) about once per second. Status: Not Validated. Implemented on four runtimes; never observed on hardware.

REQ-PRO-006 (Must): CRC16 is computed and verified. Status: Planned. Version 0.1 sends 0x0000. The chaining specification proposes CRC-16/CCITT-FALSE.

REQ-PRO-007 (Must): The runtime never blocks its main loop and keeps working with no host. Status: Not Validated.

### Firmware requirements

REQ-FW-001 (Must): One TRT-Core source tree for all platforms, varied only by feature flags. Status: Implemented.
REQ-FW-002 (Must): Support Arduino Uno, ESP32, STM32 and the simulator. Status: Not Validated.
REQ-FW-003 (Should): STM32 applications are selected at build time and share one runtime. Status: Implemented (not built).
REQ-FW-004 (Should): Each application declares its name, reported by ACTIVE_PROGRAM. Status: Implemented.
REQ-FW-005 (Must): Module drivers for each daughter-board family. Status: Planned. The module folders exist and are empty.
REQ-FW-006 (Must): Module discovery by HWID and descriptor. Status: Planned.
REQ-FW-007 (Must): Firmware identifies itself: board ID, name, revision, build ID and family. Status: Implemented.

### Host requirements

REQ-CLI-001 (Must): Discover boards on serial ports. Status: Validated against the board on COM20.
REQ-CLI-002 (Must): Read-only board queries: version, build-id, revision, name, type, id, runtime, active-program, uptime, transport. Status: Validated on the host; some report unsupported on old firmware.
REQ-CLI-003 (Should): Listen for events. Status: Not Validated; heartbeat never seen on a board.
REQ-CLI-004 (Should): Build and flash STM32 applications separately, and list programs. Status: Not Validated; unit tests pass, the real toolchain was never run.
REQ-CLI-005 (Should): Layered code: commands, services, protocol, transport. Status: Implemented.
REQ-CLI-006 (Should): Run on Windows, Linux and macOS with Python 3.13 or later. Status: Not Validated; only Windows used.
REQ-CLI-007 (Should): Show chain topology and modules. Status: Planned.

### Hardware requirements

REQ-HW-001 (Must): Carrier with STM32G474RET6, USB-C 5 V input, 3.3 V regulator, SWD and a 16-pin board-to-board connector. Status: Implemented as schematics; the PCB is in placement; not fabricated.
REQ-HW-002 (Must): Carrier exposes I2C, SPI, three GPIO and a 4-bit HWID. Status: Implemented.
REQ-HW-003 (Must): One daughter-board outline, connector position and hole pattern for every module. Status: Not Validated; mating geometry is TBD.
REQ-HW-004 (Should): Daughter variants for potentiometers (4 to 8) and a mechanical keypad (4 by 4). Status: Implemented as schematics; electrically unreviewed.
REQ-HW-005 (Could): Encoders, sliders, OLED, TFT and LED rings. Status: Planned.
REQ-HW-006 (Should): Fabrication and assembly at JLCPCB with standard SMD parts. Status: Planned.
REQ-HW-007 (Must): Power budget per carrier, port and module. Status: Planned. The chaining specification assumes 500 mA.

### Chaining requirements

REQ-CHN-001 (Must): The USB-attached carrier is master; chained carriers are slaves. Status: Planned.
REQ-CHN-002 (Must): I2C segments, one master per segment, depth up to 3. Status: Planned.
REQ-CHN-003 (Must): HWID identifies the module family and a descriptor gives details. Status: Planned.
REQ-CHN-004 (Must): Frames are routed by BOARD_ID across the chain. Status: Planned.
REQ-CHN-005 (Must): CRC16 on every chain link. Status: Planned.
REQ-CHN-006 (Must): Hot-plug of modules and slave carriers. Status: Planned.
REQ-CHN-007 (Should): Power budget enforced by the master. Status: Planned.

### Non-functional requirements

REQ-NFR-001: Response time to a command. Status: TBD.
REQ-NFR-002: Heartbeat period of about one second. Status: Not Validated.
REQ-NFR-003: Arduino Uno memory budget with 12 handlers. Status: TBD; not measured since the handler table grew.
REQ-NFR-004: USB bandwidth and power per module. Status: TBD.
REQ-NFR-005: ESD and EMC targets. Status: TBD.
REQ-NFR-006: Operating temperature and environment. Status: TBD.

### Constraints

Toolchains: PlatformIO for Arduino and ESP32, STM32CubeIDE and CMake for STM32, CMake for the simulator, Python 3.13 or later for the CLI. The corporate network blocks PlatformIO downloads unless truststore is used.
Standards and regulations: TBD.

### Acceptance criteria

A new board is accepted when it starts, answers the mandatory commands, returns ERROR_UNSUPPORTED_COMMAND for the rest, and keeps running with no host. Further criteria: TBD.

### Traceability

Each requirement is to be traced to a design section in ARCHITECTURE.md, HWDD.md or SWDD.md and to a test case in TEST_PLAN.md. Trace links: TBD.

## Future Placeholders

Stakeholder list: TBD.
Performance targets: TBD.
Module catalogue requirements: TBD.
Environmental and regulatory requirements: TBD.
Manufacturing volume and cost targets: TBD.

## Appendices

Appendix A: requirement ID scheme. Placeholder.
Appendix B: traceability matrix. Placeholder.
Appendix C: glossary extensions. Placeholder.
