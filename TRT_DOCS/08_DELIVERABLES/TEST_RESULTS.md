# Test Results

Document ID: TRT-TRS-001
Version: 0.1
Date: 2026-10-30
Status: Living log. Add an entry per test run; never delete entries.
Owner: TBD

## Purpose

This document records the results of executed tests, one entry per run, with the build, the board and the evidence. It is a reusable template. The formal summary for a release is TEST_REPORT.md.

## Scope

All tests defined in TEST_PLAN.md.

## Definitions

Result values: PASS, FAIL, PARTIAL, BLOCKED (could not run because of a missing tool or item), NOT RUN.
Status markings: Implemented, Validated, Not Validated, Planned, TBD.

## References

TEST_PLAN.md, REQUIREMENTS.md, TEST_REPORT.md.

## Revision History

Revision 0.1, 2026-10-30, TRT project: baseline with the results known from the project record.

## Approval

Tester, reviewer, approver: names, signatures and dates TBD.

## Main Content

### How to add an entry

Each entry has the date, the test ID, the build or version, the board and port, the result, notes and a reference to the evidence. Attach logs and reference them. Dates known only to the month are written as month only until evidence is attached.

### Summary of results

Host tests: 110 run, 105 pass and 5 fail. The failures are older than the recent changes. Validated on the host.
C++ simulator and protocol tests: none run in the current environment, because there is no compiler. Not Validated.
Platform tests on the COM20 board: one session with an older image. Partly validated.
Hardware tests: none; the boards are not fabricated.
Chaining tests: none; chaining is not implemented.

### Result entries

Entry 1. Date 2026-09-08. Test: Arduino Uno milestone. Build: Uno firmware before the SystemService refactor. Board: Arduino Uno. Result: PASS, per the project record. The board answered through TRT-CLI. Retest is needed after the refactor. Evidence: project record.

Entry 2. Date 2026-10. Test TC-CLI-002. Board: NUCLEO-L073RZ on COM20. Result: PASS. The board was discovered, with board ID 2 and build 000000. Evidence: terminal session.

Entry 3. Date 2026-10. Test TC-CLI-003. Board: COM20. Result: PARTIAL. The queries id, name, revision, version, build-id and transport work. The queries type, runtime, uptime and active-program report unsupported with exit code 2, because the board runs an older image. Evidence: terminal session.

Entry 4. Date 2026-10. Test TC-PRO-002, level 3. Board: COM20, older image. Result: PASS. The six version 0.1 commands were answered. Evidence: terminal session.

Entry 5. Date 2026-10. Test TC-PRO-005. Board: COM20. Result: BLOCKED. The flashed image sends no heartbeat; a new build is needed.

Entry 6. Date 2026-10-30. Test TC-CLI-001. Build: TRT-cli at commit f7182f7. Host only. Result: PARTIAL. 105 passed and 5 failed. The failures are in tests/test_cli.py and also occur with the recent changes removed. Evidence: pytest output.

Entry 7. Date 2026-10. Tests TC-FW-001 and TC-CLI-005. Board: COM20. Result: BLOCKED. There is no cmake, arm-none-eabi-gcc, ninja or STM32_Programmer_CLI, and the PlatformIO platform download fails.

### Outstanding results

Not run because there is no C++ compiler: TC-PRO-001 to TC-PRO-004, TC-PRO-006 and TC-PRO-008 at level 1.
Not run because the STM32 toolchain is missing: TC-PRO-005, TC-FW-001 to TC-FW-003, TC-CLI-004 and TC-CLI-005.
Not run because the boards are not fabricated: TC-HW-001 to TC-HW-008.
Not run because chaining is not implemented: TC-CHN-001 to TC-CHN-008.

### Defects

DEF-001, 2026-10, tests/test_cli.py: five failing tests, older than the recent work. Severity TBD. Open, not investigated.
DEF-002, 2026-10, documentation: the carrier README says AMS1117-3.3 while the power sheet shows AP2112K-3.3. Severity TBD. Open.
DEF-003, 2026-10, documentation: older notes mention a 20-pin connector while the README says 16. Severity TBD. Open.

### Conclusion

Host side: Validated, except five older test failures. Firmware changes after the shared refactor: Not Validated. Hardware: Not Validated.

## Future Placeholders

Entries for the rebuilt STM32 firmware: TBD.
Entries for hardware bring-up: TBD.
Entries for chaining: TBD.
Evidence folder convention: TBD.

## Appendices

Appendix A: raw terminal logs. Placeholder.
Appendix B: pytest output. Placeholder.
Appendix C: instrument captures. Placeholder.
