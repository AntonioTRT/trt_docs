# Test Plan

Document ID: TRT-TPL-001
Version: 0.1
Date: 2026-10-30
Status: Draft
Owner: TBD

## Purpose

This document defines how the system is verified: levels, environments, cases, criteria and procedures. It is a reusable template for any project, filled in for TRT. Results are recorded in TEST_RESULTS.md and summarised in TEST_REPORT.md.

## Scope

Protocol and firmware, the host tool, the carrier and daughter hardware, and chaining.

## Definitions

Level 1: unit and simulator tests. Level 2: host tests. Level 3: platform tests on a real board. Level 4: hardware tests with instruments. Level 5: system tests, host plus boards plus modules.
Status markings: Implemented, Validated, Not Validated, Planned, TBD. For a test case, Implemented means the case exists, Validated means it ran and passed, Not Validated means it has not run or failed, and a case that cannot run says why.

## References

REQUIREMENTS.md, ARCHITECTURE.md, HWDD.md, SWDD.md, TRT_CHAINING_SPEC_v0.1.md, TRT-core stm32-com20-validation.

## Revision History

Revision 0.1, 2026-10-30, TRT project: baseline.

## Approval

Author, reviewer, approver: names, signatures and dates TBD.

## Main Content

### Strategy

Validate in order: simulator, Arduino Uno, STM32 and ESP32, hardware boards, then chaining. Automated tests first, manual hardware tests next. A requirement is Validated only after a recorded passing result.

Level 1 tools: the CMake test target trt_tests (C++).
Level 2 tools: pytest in TRT-cli.
Level 3 tools: the CLI against a real serial port.
Level 4 tools: multimeter, oscilloscope, logic analyzer (TBD).
Level 5 tools: the CLI and a bench with several boards.

### Environments

Host: Windows 11, Python 3.13, Typer, pyserial, pytest. Available.
C++ toolchain, CMake and a compiler: missing on the current machine.
STM32 toolchain, cmake, ninja, arm-none-eabi-gcc and STM32CubeProgrammer: missing on the current machine.
PlatformIO: needs truststore on the corporate network; partly working.
Boards: NUCLEO-L073RZ on COM20 with an older image, Arduino Uno, ESP32 (TBD).
Instruments: TBD.

### Entry and exit criteria

Entry: the build passes, the environment is available and the procedure is approved.
Exit: all Must-priority cases pass, failures have a recorded decision and results are filed.

### Test cases: protocol and firmware

TC-PRO-001: PING returns ACK. Requirement REQ-PRO-002. Levels 1 and 3. Not Validated; level 1 is blocked by the missing compiler.
TC-PRO-002: FW_VERSION, BOARD_VERSION, INFO, STATUS, CAPABILITIES and BUILD_ID return DATA with correct sizes. Requirement REQ-PRO-002. Levels 1 and 3. Level 3 partly executed on COM20 with an older image; level 1 blocked.
TC-PRO-003: unknown and unsupported commands return NACK with the correct error. Requirement REQ-PRO-003. Not Validated.
TC-PRO-004: UPTIME, RUNTIME and ACTIVE_PROGRAM return DATA, or NACK when unsupported. Requirement REQ-PRO-004. Not Validated.
TC-PRO-005: HEARTBEAT arrives about every second with FLAGS 0x02. Requirement REQ-PRO-005. Level 3. Not Validated.
TC-PRO-006: a corrupt SYNC, LENGTH or VERSION is rejected. Requirement REQ-PRO-001. Level 1. Not Validated.
TC-PRO-007: CRC16 vector: the ASCII text 123456789 gives 0x29B1, and the PING frame for board 0x00C9 and sequence 1 gives 0x4EC4. Requirement REQ-PRO-006. Level 1. Planned.
TC-PRO-008: the 12-entry handler table fits Uno RAM and flash. Requirement REQ-NFR-003. Level 3. Not Validated.
TC-FW-001: each STM32 application builds. Requirement REQ-FW-003. Blocked by the missing toolchain.
TC-FW-002: each application reports its name through ACTIVE_PROGRAM. Requirement REQ-FW-004. Blocked.
TC-FW-003: the board runs with no host connected. Requirement REQ-PRO-007. Not Validated.

### Test cases: host

TC-CLI-001: the full pytest suite. Level 2. Executed: 105 pass and 5 fail, the failures being older ones in tests/test_cli.py.
TC-CLI-002: discovery finds a real board. Requirement REQ-CLI-001. Executed on COM20 with an older image.
TC-CLI-003: query commands on a real board. Requirement REQ-CLI-002. Executed on COM20: id, name, revision, version, build-id and transport work; type, runtime, uptime and active-program report unsupported.
TC-CLI-004: trt listen shows the heartbeat. Requirement REQ-CLI-003. Not Validated.
TC-CLI-005: trt build stm32, trt flash stm32 and trt list-programs stm32 with the real toolchain. Requirement REQ-CLI-004. Blocked; the 21 unit tests pass without hardware.

### Test cases: hardware

TC-HW-001: carrier power-up, rails and current. Planned; the board is not fabricated.
TC-HW-002: carrier USB enumeration and SWD connection. Planned.
TC-HW-003: HWID read for all 16 codes with straps. Planned.
TC-HW-004: I2C pull-ups switch with PC15. Planned.
TC-HW-005: daughter board power, HWID and descriptor. Planned.
TC-HW-006: potentiometer module, converter readings across the full travel. Planned.
TC-HW-007: keypad module, every key and no ghosting. Planned.
TC-HW-008: magnetic connector, contact resistance and bounce. Planned.

### Test cases: chaining

TC-CHN-001: role detection for master, slave, standalone and dual master. Planned.
TC-CHN-002: a frame is forwarded through one slave at 100 kHz. Planned.
TC-CHN-003: descriptor and HWID agree; a mismatch gives a fault. Planned.
TC-CHN-004: path ID routing at depth 1, 2 and 3. Planned.
TC-CHN-005: a bad CRC is dropped and retried. Planned.
TC-CHN-006: hot-plug with 100 insertions and removals. Planned.
TC-CHN-007: power budget refusal. Planned.
TC-CHN-008: stuck-bus recovery. Planned.

### Procedure: STM32 board validation on COM20

Step 1: build with trt build stm32 program_blink.
Step 2: flash with trt flash stm32.
Step 3: run trt discover and trt boards and expect the board with ID 201.
Step 4: run trt board info, version, build-id, uptime, runtime, type and active-program, and expect program_blink for the last.
Step 5: run trt stm32 test.
Step 6: run trt listen for 10 seconds and expect about 10 heartbeats.
Step 7: record every output in TEST_RESULTS.md.
Prerequisite: the toolchain is installed.

### Defect handling

Record the defect ID, the test case, the severity and the build. Fix, retest and record the change in CHANGE_DESIGN_REVISION.md. Severity scale: TBD.

### Roles

Test lead, hardware tester and firmware tester: TBD.

## Future Placeholders

Automated hardware-in-the-loop tests: TBD.
Regression suite and continuous integration: TBD.
EMC and ESD tests: TBD.
Performance and endurance tests: TBD.

## Appendices

Appendix A: test case templates. Placeholder.
Appendix B: instrument list and calibration. Placeholder.
Appendix C: bench setup photographs. Placeholder.
