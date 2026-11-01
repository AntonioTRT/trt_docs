# TRT-CLI Test Plan

Document ID: TRT-CLI-TPL-001
Version: 0.1
Date: 2026-10-30
Status: Draft
Owner: TBD

## Purpose

This document defines how the TRT-CLI host tool (the `trt` command, Python, Typer and pyserial) is tested: what is tested, at which level, with which tools, and which test cases apply. It refines the host section of TEST_PLAN.md (TC-CLI-001 to TC-CLI-005) into a complete set of cases. Results are recorded in TEST_RESULTS.md and summarised in TEST_REPORT.md.

## Scope

In scope:
- Command parsing, help text, argument and option validation, exit codes and output of every `trt` command group.
- Board discovery, the board cache, the board query commands, listen and heartbeat handling.
- The transport layer (mock transport and serial transport), the frame reader and the protocol encoder and decoder as used by the CLI.
- The STM32 build, flash, list-programs and handshake-test commands.
- The simulated command groups (GPIO, PWM, ADC, DAC, I2C, SPI, LCD, LED, debug) as far as they are implemented.
- Packaging, installation and the update check.

Out of scope:
- Firmware behaviour on the board (see TEST_PLAN.md, TC-PRO and TC-FW cases). The CLI is only tested against a firmware through its observable replies.
- Hardware tests of the carrier and daughter boards.
- Chaining (planned; CLI cases for it are listed as placeholders).

## Definitions

Level A: unit tests of one function or class, with no I/O. Level B: command tests that run the CLI in-process with the Typer test runner and the mock transport. Level C: tests with a fake serial device or a protocol simulator. Level D: tests with a real board on a serial port. Level E: tests of build and flash with the real toolchain.
Exit code convention: 0 success, 1 failure (transaction, transport or tool error), 2 the board reported the query as unsupported, and the usual Typer code 2 for usage errors. The two meanings of 2 are a known ambiguity, see TC-CLI-WRN-001.
Status markings: Implemented, Validated, Not Validated, Planned, TBD. For a case, Implemented means an automated test exists, Validated means it ran and passed with a recorded result, Not Validated means it has not run or has failed, Planned means no test exists yet.

## References

TEST_PLAN.md, TEST_RESULTS.md, TEST_REPORT.md, REQUIREMENTS.md (REQ-CLI-001 to REQ-CLI-004), SWDD.md, ARCHITECTURE.md, TRT-core docs/protocol-v0.1-wire-format.md and docs/error-model.md, TRT-cli README.md, QUICKSTART.md and docs/.

## Revision History

Revision 0.1, 2026-10-30, TRT project: first TRT-CLI test plan.

## Approval

Author, reviewer, approver: names, signatures and dates TBD.

## Main Content

### Current baseline

TRT-CLI is at commit f7182f7. The pytest suite has 110 tests in tests/test_cli.py, tests/test_board_queries.py and tests/test_stm32_cli.py. A run on 2026-10-30 gave 105 passed and 5 failed; the five failures are in tests/test_cli.py and are older than the recent work (DEF-001, not investigated). The first target of this plan is to understand and close those five failures.

The commands in the current command tree, as read from the source:
- Top level: boards, discover (rescan), listen, version, update check, help reference.
- `trt board`: info, status, reset, reboot, capabilities, listen, modules, and the query commands id, name, type, revision, version, build-id, runtime, active-program, uptime, transport.
- `trt board` sub-groups: gpio (list, read, write), pwm (list, start, stop, set), adc (list, read), dac (list, read, set), i2c (scan, read, write), spi (transfer, config), debug (logs, monitor, shell).
- `trt lcd` and `trt led` command groups.
- `trt protocol`: info and version.
- `trt build stm32`, `trt flash stm32`, `trt list-programs stm32`, `trt stm32 test`.

Whether each simulated group talks to a real board is TBD per command; the cases below mark the expected behaviour and must be adjusted when a command is confirmed as simulation only.

### Test strategy

Principle 1: most CLI behaviour is tested without hardware. The mock transport and a fake serial port make levels A to C fast and repeatable, and they run on every change.
Principle 2: the real board is used to confirm what the mocks assume. Level D results decide whether the mocks are still correct.
Principle 3: every command has at least one success case, one failure case and one help case.
Principle 4: output is checked on its content (values, units, exit code) and on machine-readable fields, not on exact formatting or colour.
Principle 5: the CLI must never hang. Every transaction has a timeout, and each timeout case is tested.
Principle 6: a defect found on hardware gets a level A to C regression test before it is closed.

### Test tools

pytest, with the Typer CliRunner for in-process command tests.
The mock transport in src/trt/transport/mock_transport.py for scripted replies.
A frame-level fake for the serial transport: a loopback or a virtual serial pair (TBD: com0com on Windows, pty on Linux and macOS).
unittest.mock or monkeypatch for subprocess calls (cmake, ninja, STM32_Programmer_CLI, PlatformIO) in build and flash tests.
Temporary directories for cache and project fixtures.
Coverage measurement with pytest-cov: planned.
Real board: NUCLEO-L073RZ on COM20; Arduino Uno and ESP32 boards TBD.

### Environments

Operating systems: Windows 11 (available). Linux and macOS: TBD, needed because the CLI claims cross-platform support and serial port naming differs.
Python: 3.13 on the development machine. The supported range is whatever pyproject.toml declares; the lowest and highest supported versions must each be tested (TBD).
Installation modes: editable install, wheel install and `python -m trt`.
Network: corporate network that blocks PlatformIO downloads; the update check must be tested both online and offline.

### Entry and exit criteria

Entry: the package installs, the pytest suite collects without errors, and the test plan is approved.
Exit for a release: all Must-priority cases pass, no open defect of high severity (the severity scale is TBD), the five known failures are resolved or have a recorded decision, and results are filed in TEST_RESULTS.md.

### Priorities

Must: protocol framing, discovery, query commands, exit codes, timeouts, build and flash safety.
Should: help text, output formats, the cache, listen.
Could: cosmetic output, shell completion, simulated command groups.

### Test cases: installation and basics

TC-CLI-INS-001 (Must, level B, Implemented): `trt --help` lists all command groups and exits with 0.
TC-CLI-INS-002 (Must, level B): every command and sub-command answers `--help` with usage text and exit code 0. Implement as a loop over the Typer command tree, so a new command without help text fails the test.
TC-CLI-INS-003 (Must, level B): `trt version` prints the version from src/trt/version.py and it agrees with pyproject.toml.
TC-CLI-INS-004 (Should, level E): a clean wheel install in a fresh virtual environment provides the `trt` entry point and `python -m trt` works.
TC-CLI-INS-005 (Should, level B): an unknown command gives a usage error, a non-zero exit code and no traceback.
TC-CLI-INS-006 (Should, level B): `trt update check` works offline: it reports that the check failed, does not crash and exits cleanly. Online behaviour is TBD.
TC-CLI-INS-007 (Could, level B): the full command reference (`trt help`) lists every registered command, so documentation cannot drift from the command tree.

### Test cases: protocol, encoder, decoder and frame reader

TC-CLI-PRO-001 (Must, level A): the encoder produces the exact bytes for a PING frame: SYNC 0xAA55, VERSION 0x01, FLAGS, BOARD_ID, SEQ_ID, COMMAND 0x0001, LENGTH 0, CRC 0x0000, big-endian, 14 bytes of overhead. Use fixed vectors.
TC-CLI-PRO-002 (Must, level A): the decoder round-trips every command and response ID: commands 0x0001 to 0x000A, responses 0x8001 to 0x8003 and HEARTBEAT 0x9001 with FLAGS 0x02.
TC-CLI-PRO-003 (Must, level A): the decoder rejects a bad SYNC, a bad VERSION and a LENGTH larger than the maximum, each with a distinct error.
TC-CLI-PRO-004 (Must, level A): the frame reader reassembles a frame delivered one byte at a time, in two halves, and in a burst of several frames.
TC-CLI-PRO-005 (Must, level A): the frame reader resynchronises after garbage bytes before SYNC and after a truncated frame.
TC-CLI-PRO-006 (Should, level A): a payload of the maximum length and a payload of zero length both work.
TC-CLI-PRO-007 (Should, level A): sequence IDs increment and wrap at their maximum, and a response with a wrong SEQ_ID is ignored or reported.
TC-CLI-PRO-008 (Should, level A): a response whose BOARD_ID differs from the request is rejected.
TC-CLI-PRO-009 (Planned, level A): CRC16 CCITT-FALSE. The ASCII text 123456789 gives 0x29B1 and the PING frame for board 0x00C9 and sequence 1 is AA55010000C90001000100004EC4. Blocked on the CRC protocol patch (open decision O-10).
TC-CLI-PRO-010 (Planned, level A): a frame with a wrong CRC is rejected once CRC_VALID is defined.
TC-CLI-PRO-011 (Should, level A): an unknown error ID in a NACK is shown as a number, not as a crash. The known error IDs 0x0001 to 0x0006 each map to a readable message.

### Test cases: transport

TC-CLI-TRN-001 (Must, level B): with the mock transport, a request gets its scripted response.
TC-CLI-TRN-002 (Must, level C): the serial transport opens the configured port, sends the encoded bytes and closes the port, including after an error. The port is not left open.
TC-CLI-TRN-003 (Must, level C): a read timeout raises a clear timeout error within the configured time and the CLI exits with code 1.
TC-CLI-TRN-004 (Must, level C): opening a port that does not exist gives a clear error with the port name, not a traceback.
TC-CLI-TRN-005 (Should, level C): opening a port held by another program gives a clear "busy" error.
TC-CLI-TRN-006 (Should, level C): the port is closed and the CLI exits cleanly when the device is unplugged during a transaction.
TC-CLI-TRN-007 (Should, level C): the baud rate and other serial settings match the firmware defaults (value TBD, to be copied from the TRT-core transport configuration).
TC-CLI-TRN-008 (Could, level C): an explicit `--port` overrides auto-detection and the port name is accepted in the forms COM5, /dev/ttyACM0 and /dev/cu.usbmodem1.

### Test cases: discovery and the board cache

TC-CLI-DSC-001 (Must, level B): `trt discover` with one mock board lists it with the board ID, name and port.
TC-CLI-DSC-002 (Must, level B): with no serial ports, discover reports that nothing was found and exits cleanly.
TC-CLI-DSC-003 (Must, level B): with two boards on two ports, both are listed and neither hides the other.
TC-CLI-DSC-004 (Must, level B): a port that does not answer within the timeout is skipped without blocking the others, and the total scan time stays within a stated limit (limit TBD).
TC-CLI-DSC-005 (Should, level B): a port that answers with a non-TRT protocol is ignored and reported, not treated as a board.
TC-CLI-DSC-006 (Should, level B): two boards with the same BOARD_ID are reported as a conflict.
TC-CLI-DSC-007 (Should, level B): `trt boards` shows the cached boards and marks an entry as not seen if its port is gone.
TC-CLI-DSC-008 (Should, level B): the cache file is created on first use, and a corrupt or empty cache file is handled without a crash. The cache location on Windows, Linux and macOS is TBD in this plan.
TC-CLI-DSC-009 (Should, level B): the cache is updated when a board changes its port.
TC-CLI-DSC-010 (Could, level D): discovery finds the NUCLEO-L073RZ on COM20 with board ID 2. This is the existing TC-CLI-002, which passed with an older image.

### Test cases: board commands and queries

TC-CLI-BRD-001 (Must, level B): `trt board info <id>` shows identity and firmware information from a mocked INFO reply, including the family field 0x06.
TC-CLI-BRD-002 (Must, level B): each query command id, name, type, revision, version, build-id, runtime, active-program, uptime and transport returns the right field from the mocked reply and exits with 0.
TC-CLI-BRD-003 (Must, level B): when the board replies NACK with "unsupported" for a query, the command prints "unsupported" and exits with code 2. This is a tested behaviour for type, runtime, uptime and active-program on older firmware.
TC-CLI-BRD-004 (Must, level B): a transport failure during a query exits with code 1 and prints the cause.
TC-CLI-BRD-005 (Must, level B): a board ID in decimal (101) and in hexadecimal (0x65) are both accepted and mean the same board; a non-numeric ID gives a usage error. Note that the help text of the gpio, pwm and adc examples shows `board0`, which is not numeric and is a documentation defect to confirm (DEF-004 proposed).
TC-CLI-BRD-006 (Should, level B): `--port` is honoured, and without it the port is found through the cache or discovery. If neither works, the error says what to do.
TC-CLI-BRD-007 (Should, level B): the uptime value is shown in a readable unit and a value at the 32-bit boundary is handled.
TC-CLI-BRD-008 (Should, level B): `board status` and `board capabilities` show the data from the mocked reply, and an empty capability list is shown as empty, not as an error.
TC-CLI-BRD-009 (Should, level B): `board reset` and `board reboot` ask for no confirmation or ask for one (the intended behaviour is TBD); they never reset without a reply from a real board.
TC-CLI-BRD-010 (Should, level B): the "which command to use" table in the CLI reference (added in commit f7182f7) is consistent with the commands that exist.
TC-CLI-BRD-011 (Must, level D): the queries id, name, revision, version, build-id and transport on COM20 (existing TC-CLI-003, partial pass). Repeat with a newly built image for type, runtime, uptime and active-program.
TC-CLI-BRD-012 (Could, level B): query output with `--json` or a similar machine-readable switch, if it exists (TBD whether the option exists).

### Test cases: listen and heartbeat

TC-CLI-LST-001 (Must, level B): `trt listen` prints a heartbeat frame (command 0x9001, FLAGS 0x02) delivered by the mock.
TC-CLI-LST-002 (Must, level B): listen stops cleanly on Ctrl+C with exit code 0 and closes the port.
TC-CLI-LST-003 (Should, level B): heartbeat frames and event frames are shown differently, and an unknown asynchronous command ID is shown, not discarded.
TC-CLI-LST-004 (Should, level B): a heartbeat from a board other than the selected one is filtered out for `trt board listen <id>`.
TC-CLI-LST-005 (Should, level B): a missed heartbeat is reported after a timeout (limit TBD, expected around three missed intervals of one second).
TC-CLI-LST-006 (Should, level C): a request and a heartbeat arriving together do not corrupt each other; the request still gets its reply.
TC-CLI-LST-007 (Must, level D): on a rebuilt firmware, about 10 heartbeats arrive in 10 seconds. This is TC-CLI-004 and TC-PRO-005, currently blocked.

### Test cases: STM32 build, flash and program listing

TC-CLI-STM-001 (Must, level B, Implemented): `trt list-programs stm32` lists the files Sources/programs/program_*.c of a fixture workspace, sorted, and says so if there are none.
TC-CLI-STM-002 (Must, level B, Implemented): `trt build stm32 <program>` with an unknown program name gives a usage error that lists the available programs.
TC-CLI-STM-003 (Must, level B): `trt build stm32` with a mocked toolchain calls cmake and ninja with the expected arguments and the selected ACTIVE_PROGRAM, and exits with 0.
TC-CLI-STM-004 (Must, level B): a failing cmake or ninja exit code is passed on as the exit code of the CLI, and the output of the tool is shown.
TC-CLI-STM-005 (Must, level B): a missing toolchain (cmake, ninja, arm-none-eabi-gcc) gives an error naming the missing tool and how to install it, not a traceback.
TC-CLI-STM-006 (Must, level B): `trt flash stm32` without a previous build stops with a message that says to build first.
TC-CLI-STM-007 (Must, level B): `trt flash stm32` calls STM32_Programmer_CLI with the artifact of the last build, and a failing programmer exit code is passed on.
TC-CLI-STM-008 (Should, level B): the `--platformio` flash mode uses `--port` and the PlatformIO environment, and refuses to run without the port.
TC-CLI-STM-009 (Should, level B): `--project-dir` selects the workspace; a directory that is not a TRT STM32 workspace gives a clear error.
TC-CLI-STM-010 (Should, level B): the program name is validated against path traversal and shell metacharacters; a name such as `../x` or `a;b` is rejected and nothing is executed. Commands are run without a shell.
TC-CLI-STM-011 (Should, level B): the build and flash steps never write outside the project directory and the build directory.
TC-CLI-STM-012 (Should, level E): the real build of every program in the workspace succeeds. Blocked until the toolchain is installed (TC-FW-001 and TC-CLI-005).
TC-CLI-STM-013 (Should, level E): the full sequence from TEST_PLAN.md, build program_blink, flash, discover, query active-program (expect program_blink) and run `trt stm32 test`. Blocked.
TC-CLI-STM-014 (Should, level B): `trt stm32 test` runs the six version 0.1 commands against a mocked board and reports a pass or a failure for each; one failing step gives exit code 1.
TC-CLI-STM-015 (Should, level B): `trt stm32 test` defaults to board ID 0x0002, and `--board-id` and `--port` override it.

### Test cases: simulated and other command groups

These groups (gpio, pwm, adc, dac, i2c, spi, lcd, led, debug, modules) are only tested as far as they are implemented. If a group is a stub, the test records that it is a stub.

TC-CLI-GRP-001 (Should, level B): each command in these groups accepts valid arguments and exits with 0 with the mock transport.
TC-CLI-GRP-002 (Should, level B): out-of-range values are rejected before any frame is sent: led brightness outside 0 to 100, led colour channels outside 0 to 255, PWM duty outside 0 to 100, GPIO value other than 0 or 1, and DAC values beyond the resolution (limit TBD).
TC-CLI-GRP-003 (Should, level B): missing arguments give a usage error with the example text.
TC-CLI-GRP-004 (Should, level B): the I2C commands accept addresses in decimal and hexadecimal, reject addresses above 0x7F, and the scan result lists addresses in a stable format.
TC-CLI-GRP-005 (Could, level B): debug shell and monitor start and stop cleanly with the mock transport.
TC-CLI-GRP-006 (Could, level D): the same groups against a real board, once the firmware supports them (TBD).

### Test cases: robustness and non-functional

TC-CLI-NFR-001 (Must, level B): no command can hang: every command is run with a transport that never replies, and returns within a time limit with exit code 1.
TC-CLI-NFR-002 (Should, level B): no command prints a Python traceback for an expected error (bad input, missing port, timeout, unsupported). A traceback is a defect.
TC-CLI-NFR-003 (Should, level B): output remains readable when colour is disabled, when output is redirected to a file, and on a narrow terminal. The `NO_COLOR` convention is honoured (TBD whether supported).
TC-CLI-NFR-004 (Should, level B): non-ASCII characters in output do not crash on a Windows console with a legacy code page.
TC-CLI-NFR-005 (Should, level B): startup time of `trt --help` stays under a limit (limit TBD, for example one second) so that lazy imports are not accidentally lost.
TC-CLI-NFR-006 (Should, level A): static checks pass: type checks and lint (tool selection TBD).
TC-CLI-NFR-007 (Could, level B): concurrent use of two `trt` processes against one cache file does not corrupt the cache.
TC-CLI-NFR-008 (Could, level B): exit codes are documented and tested for each command class, which also resolves the ambiguity of code 2 (see TC-CLI-WRN-001).
TC-CLI-NFR-009 (Should, level B): the CLI does not write or send anything to the network except in the update check, which is tested with a mocked network.

### Test cases: portability

TC-CLI-POR-001 (Should, level B): path handling for the project directory and the cache works with spaces and non-ASCII characters on Windows.
TC-CLI-POR-002 (Should, level B): serial port names are validated per platform (COMn, /dev/ttyACM*, /dev/ttyUSB*, /dev/cu.*).
TC-CLI-POR-003 (Should): the entire automated suite is run on Windows, Linux and macOS. Linux and macOS: TBD.
TC-CLI-POR-004 (Could): the suite runs on the lowest and the highest supported Python versions.

### Test cases: chaining placeholders

These cases wait for the chaining implementation.

TC-CLI-CHN-001 (Planned): a topology view shows the master carrier, its slaves and their modules with path IDs.
TC-CLI-CHN-002 (Planned): a command to a board at path depth 1, 2 and 3 is routed and the reply returns.
TC-CLI-CHN-003 (Planned): a hot-plugged slave appears and disappears in the topology view without a restart.
TC-CLI-CHN-004 (Planned): the new error IDs 0x0010 to 0x0016 are shown as readable messages.
TC-CLI-CHN-005 (Planned): the module descriptor (32 bytes) is read and shown, including the HWID and family.

### Known issues to test around

TC-CLI-WRN-001: exit code 2 means "unsupported" in the board query commands and "usage error" in Typer. A script cannot tell them apart. Decide on a distinct code, then test it. Proposed defect DEF-004.
TC-CLI-WRN-002: the five failing tests in tests/test_cli.py (DEF-001). First step: run them one by one, decide whether the test or the code is wrong, then fix and record the change in CHANGE_DESIGN_REVISION.md.
TC-CLI-WRN-003: help examples use `board0` where the argument is numeric (see TC-CLI-BRD-005).

### Procedure: running the automated suite

Step 1: create a clean virtual environment and install TRT-cli with its test dependencies (for example `pip install -e .` plus pytest).
Step 2: run `pytest -q` in the TRT-cli directory.
Step 3: record the number of tests passed, failed and skipped, the commit hash and the Python version in TEST_RESULTS.md.
Step 4: for each failure, open a defect with the test name and the first lines of the traceback.
Step 5: run the suite again after each fix; the pass count must not decrease.

### Procedure: running the level D checks on a real board

Step 1: connect the board and note the port (COM20 for the NUCLEO-L073RZ).
Step 2: run `trt discover` and `trt boards`; expect the board.
Step 3: run every query command of `trt board`; expect values and exit code 0, or "unsupported" and exit code 2 for old firmware.
Step 4: run `trt stm32 test`.
Step 5: run `trt listen` for 10 seconds.
Step 6: unplug the board during `trt listen` and confirm the CLI exits cleanly.
Step 7: record every output in TEST_RESULTS.md.

### Coverage targets

Command coverage: every command is exercised by at least one test (target 100 percent, measured by walking the command tree).
Line coverage of src/trt: target TBD; suggested starting target 80 percent for protocol, transport, services and commands, measured with pytest-cov.
Requirement coverage: REQ-CLI-001 to REQ-CLI-004 each map to at least one Must case; the mapping is: REQ-CLI-001 to the DSC cases, REQ-CLI-002 to the BRD cases, REQ-CLI-003 to the LST cases, REQ-CLI-004 to the STM cases.

### Order of implementation

Priority 1: investigate the five failing tests, then add INS-002, NFR-001 and NFR-002, which find the most defects for the least effort.
Priority 2: the PRO and TRN cases, which protect every other command.
Priority 3: the STM cases with mocked tools, to cover the build and flash safety.
Priority 4: the level D checks, as soon as the toolchain is installed and the firmware is rebuilt.
Priority 5: portability on Linux and macOS and the chaining placeholders.

### Status summary

Implemented as automated tests, per the existing 110 tests: part of the BRD, STM and INS cases (the exact mapping between the existing test names and the case IDs here is TBD and is the first documentation task).
Validated on hardware: DSC-010 and part of BRD-011, with an older image.
Not Validated: everything else, including LST-007 and STM-012 and STM-013, which are blocked by the missing toolchain.
Planned: PRO-009, PRO-010 and all CHN cases.

## Future Placeholders

Continuous integration for pytest on three operating systems: TBD.
Hardware-in-the-loop runs against a board attached to a test server: TBD.
Fuzz tests for the frame reader and the decoder: TBD.
Performance tests for discovery with many ports: TBD.
Usability review of messages and help text: TBD.
Test data: recorded frame captures from real boards for replay: TBD.

## Appendices

Appendix A: mapping from existing pytest test names to case IDs. Placeholder.
Appendix B: frame test vectors. Placeholder.
Appendix C: mock transport scripting guide. Placeholder.
Appendix D: exit code table, written as text lines. Placeholder.
