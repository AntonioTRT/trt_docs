# Software Design Description

Document ID: TRT-SWD-001
Version: 0.1
Date: 2026-10-30
Status: Draft
Owner: TBD

## Purpose

This document describes the software design: components, interfaces, build and verification. It is a reusable template for software projects, filled in for TRT.

## Scope

Firmware (TRT-Core and board runtimes) and the host tool (TRT-CLI). Hardware is in HWDD.md.

## Definitions

TRT-Core, runtime, TRT-CLI: as in ARCHITECTURE.md.
SystemService: the shared service that implements the system commands.
HeartbeatService: the service that emits periodic events.
Status markings: Implemented, Validated, Not Validated, Planned, TBD.

## References

ARCHITECTURE.md, REQUIREMENTS.md, TEST_PLAN.md, TRT_CHAINING_SPEC_v0.1.md.
TRT-core documentation and TRT-cli documentation (cli-reference).

## Revision History

Revision 0.1, 2026-10-30, TRT project: baseline from the repository state.

## Approval

Author, reviewer, approver: names, signatures and dates TBD.

## Main Content

### Software status

Frame parser and encoder: Implemented; Not Validated in the current environment (simulator tests existed earlier).
Dispatcher and command registry (12 handlers on Uno): Implemented; Not Validated.
SystemService with PING, FW_VERSION, BOARD_VERSION, INFO, STATUS, CAPABILITIES, BUILD_ID, UPTIME, RUNTIME and ACTIVE_PROGRAM: Implemented; Not Validated, never compiled.
HeartbeatService: Implemented; Not Validated, never seen on hardware.
Arduino runtime: Implemented; validated at an earlier milestone, before the refactor, so now Not Validated.
ESP32 runtime: Implemented; Not Validated.
STM32 CubeIDE runtime and application launcher: Implemented; Not Validated.
STM32 PlatformIO runtime (legacy): Implemented; the COM20 board runs an older image that answers six commands.
Simulator: Implemented; Not Validated in the current environment.
Module drivers: Planned; the folders are empty.
Chaining (link layer, routing, discovery): Planned.
TRT-CLI: Implemented. 110 tests, 105 pass, and five failures are older than the recent changes. Validated on the host.

### TRT-Core design

Protocol (core/protocol): frame, parser, encoder and identifiers.
Dispatcher (core/dispatcher): command registry, a function-pointer table on Uno.
Board context (core/board): board information (name, revision, firmware version, build ID, family, active program), capabilities and an uptime clock.
Services (core/services): SystemService and HeartbeatService.
Capabilities (core/capabilities): capability identifiers and registry.
Modules (modules): module interface and ModuleManager. The family folders relay, lcd, neopixel, dac, analog_io and digital_io are empty.
Interfaces (interfaces): gpio, adc, dac, pwm, i2c (a write and read interface only), spi, uart, lcd, neopixel, relay and storage.
Simulator (simulator): a software board used by protocol tests.
Feature flags: TRT_HAS_STL, TRT_HAS_DYNAMIC_MEMORY, TRT_HAS_MODULES and TRT_MAX_FRAME_SIZE.

### Board runtimes

Each runtime supplies a UART transport, a millisecond clock, the board profile, capability registration, and the entry points trt_runtime_setup and trt_runtime_poll.
Arduino Uno, ID 101: static memory with a 50-byte payload buffer.
ESP32, ID 301: the active program is esp32_discovery.
STM32 CubeIDE, ID 201: SysTick, HSI16 and USART2 on PA2 and PA3.
STM32 PlatformIO, ID 2: GPIO, DAC and LED capabilities; STATUS reports WARNING on overflow.
Known debt: the byte-to-frame state machine is copied in four runtimes, and boards/arduino/mega is an unused older copy.

### Commands

Identifiers and replies are in ARCHITECTURE.md. Unsupported commands return NACK with ERROR_UNSUPPORTED_COMMAND. UPTIME needs a clock and ACTIVE_PROGRAM needs a declared program name; otherwise they report unsupported.

### STM32 application framework

One CubeIDE CMake project holds all applications and one TRT-Core. main.c is a launcher. app_launcher.c sets the program name, calls trt_runtime_setup and program_init, and in the loop calls trt_runtime_poll and program_loop. The application is selected by ACTIVE_PROGRAM in app_selector.h or by -DACTIVE_PROGRAM=PROGRAM_X. Applications: program_blink, program_pulse_train and program_idle. Time comes from app_time_us. Status: Implemented; Not Validated, never built.

### Build and flash

Artifacts: artifacts/PROGRAM/PROGRAM.elf and artifacts/latest.json.
STM32 needs cmake, arm-none-eabi-gcc and STM32_Programmer_CLI. Arduino and ESP32 need PlatformIO and truststore on the corporate network.

### Host design

Commands: discover, boards, board, listen, stm32, build, flash and list-programs.
Services: discovery, board service (resolve_serial_connection), BoardQueryService, build and flash helpers.
Protocol: protocol/v01.py, including INFO decoding and the family field.
Transport: SerialTransport.
Board queries: version, build-id, revision, name, type, id, runtime, active-program, uptime and transport. The transport query is answered by the host. Unsupported replies exit with code 2. The board status command reads the discovery cache, not the live board.

### Planned software

Chain link layer, TRT frames over I2C with registers at address 0x10. Planned.
Routing by path ID and forwarding. Planned.
Port state machine for hot-plug. Planned.
Descriptor reader and HWID decoder. Planned.
Family drivers for analog, matrix, encoder, display and LED modules. Planned.
CRC16 CCITT-FALSE in parser and encoder, plus a CRC_VALID flag. Planned.
New commands 0x0020 to 0x0022, events 0x9002 and 0x9003, errors 0x0010 to 0x0016. Planned.
CLI topology view. Planned.
Removal of the runtime duplication. Planned, optional.

### Configuration management

Repositories on GitHub under AntonioTRT. build_registry.csv records builds. Branching, release and versioning rules: TBD. Coding standard and static analysis: TBD.

## Future Placeholders

Continuous integration: TBD.
Logging and diagnostics: TBD.
Firmware update: TBD.
Security: TBD.
Memory budget on Arduino Uno: TBD.

## Appendices

Appendix A: class and module diagrams. Placeholder.
Appendix B: command reference. Placeholder.
Appendix C: build instructions. Placeholder.
