# Test Report

Document ID: TRT-TRP-001
Version: 0.1
Date: 2026-10-30
Status: Draft. A partial report: most tests could not be run yet.
Owner: TBD

## Purpose

This document is the formal summary of a test campaign: what was tested, in which environment, the results, the problems found and the conclusions. It is a reusable template for any project, filled in for TRT. Detailed entries are in TEST_RESULTS.md.

## Scope

Protocol, host and platform tests that have been run. Electrical hardware tests and chaining tests are out of scope for this report because they could not be run yet.

## Definitions

Result values: PASS, FAIL, PARTIAL, BLOCKED, NOT RUN.
Status markings: Implemented, Validated, Not Validated, Planned, TBD.

## References

TEST_PLAN.md, TEST_RESULTS.md, REQUIREMENTS.md, RISK_REGISTER.md.

## Revision History

Revision 0.1, 2026-10-30, TRT project: first report, with the results known to date.

## Approval

Test lead, reviewer, approver: names, signatures and dates TBD.

## Main Content

### Executive summary

The host-side tests of TRT-CLI were run, and one discovery and query session was run against a NUCLEO-L073RZ board on COM20 that runs an older firmware image. The modified firmware could not be compiled or tested, because the C++ and STM32 toolchains are not installed on the development machine. The carrier and daughter boards are not fabricated.

Conclusion: the host side is Validated, except five older test failures. The modified firmware is Not Validated. The hardware is Not Validated.

### Items under test

TRT-cli at commit f7182f7.
TRT-core at commit b0891c4. The firmware with these changes has not been compiled.
Board tested: NUCLEO-L073RZ on COM20, board ID 2, build 000000, an older image.
TRT_HW: carrier and daughter schematics only; no boards built.

### Test environment

PC with Windows 11, Python 3.13, Typer, pyserial and pytest: available.
C++ compiler and CMake: not available.
STM32 toolchain (cmake, ninja, arm-none-eabi-gcc, STM32CubeProgrammer): not available.
PlatformIO: the ststm32 platform download fails on the corporate network.
Instruments: TBD.

### Results

TC-CLI-001, full pytest suite: PARTIAL. 105 passed and 5 failed. The five failures are in tests/test_cli.py and also occur without the recent changes.
TC-CLI-002, discovery of a real board: PASS. The COM20 board was found with an older image.
TC-CLI-003, queries on a real board: PARTIAL. The queries id, name, revision, version, build-id and transport work. The queries type, runtime, uptime and active-program report unsupported with exit code 2, because the firmware is older.
TC-PRO-002, the six version 0.1 commands at level 3: PASS, with the older image.
TC-PRO-005, heartbeat: BLOCKED. The flashed image sends no heartbeat; new firmware must be built and flashed.
TC-FW-001 and TC-CLI-005, build and flash of STM32 applications: BLOCKED. The toolchain is missing.
TC-PRO-001 to TC-PRO-004, TC-PRO-006 and TC-PRO-008, protocol tests in C++: NOT RUN. There is no compiler.
TC-HW-001 to TC-HW-008, hardware tests: NOT RUN. The boards are not fabricated.
TC-CHN-001 to TC-CHN-008, chaining tests: NOT RUN. Chaining is not implemented.

### Nonconformities and observations

DEF-001: five tests in tests/test_cli.py fail and the cause is not investigated. Severity TBD. Open.
DEF-002: the carrier README says AMS1117-3.3, but the power sheet shows AP2112K-3.3. Severity TBD. Open.
DEF-003: older notes mention a 20-pin connector, but the README says 16. Severity TBD. Open.
OBS-001: the COM20 image sends no heartbeat and does not answer the newer queries. A new build resolves this.

### Requirements coverage

REQ-PRO-001: Validated, historically, with the simulator and the Arduino Uno on 2026-09-08.
REQ-PRO-002 to REQ-PRO-005 and REQ-PRO-007: Not Validated.
REQ-PRO-006, CRC16: Planned.
REQ-CLI-001 and REQ-CLI-002: Validated.
REQ-CLI-003 and REQ-CLI-004: Not Validated.
REQ-HW-001 to REQ-HW-007: Not Validated or Planned.
REQ-CHN-001 to REQ-CHN-007: Planned.

### Exit criteria check

The exit criterion in TEST_PLAN.md is that all Must-priority cases pass. This criterion is not met.

### Conclusions

The host side works and was tested against a real board.
Firmware changes made after the shared refactor are not compiled or tested.
There is no hardware data, so nothing can be claimed about the carrier, the daughter board or chaining.

### Recommendations

Install the toolchains and compile every platform (milestones M1 and M2 in ROADMAP.md).
Flash program_blink on the COM20 board and repeat the validation procedure.
Investigate the five failing tests in tests/test_cli.py.
Resolve DEF-002 and DEF-003 against the schematics.
Repeat this report with the new results.

## Future Placeholders

Results for rebuilt STM32 firmware: TBD.
Results for Arduino and ESP32 rebuilds: TBD.
Hardware bring-up results: TBD.
Chaining results: TBD.
Sign-off for release: TBD.

## Appendices

Appendix A: terminal logs from the COM20 session. To be attached.
Appendix B: pytest output. To be attached.
Appendix C: instrument captures. Placeholder.
