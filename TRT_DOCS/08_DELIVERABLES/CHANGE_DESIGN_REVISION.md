# Change and Design Revision Record

Document ID: TRT-CDR-001
Version: 0.1
Date: 2026-10-30
Status: Living log. Add an entry per change; never delete entries.
Owner: TBD

## Purpose

This document records every design change to hardware, firmware, host, protocol and documentation: the reason, the impact and who approved it. It is a reusable change-control template for any project, filled in for TRT.

## Scope

All items under configuration control: TRT-core (firmware), TRT-cli (host), TRT_HW (hardware, KiCad) and trt_docs (documentation).

## Definitions

CDR: a change and design revision entry, identified as CDR-NNN.
Frozen item: an item that needs formal approval to change. For TRT these are the protocol version 0.1 wire format, the connector pinout and the HWID scheme.
Status markings: Implemented, Validated, Not Validated, Planned, TBD.

## References

REQUIREMENTS.md, ARCHITECTURE.md, HWDD.md, SWDD.md, TEST_PLAN.md, TEST_RESULTS.md, RISK_REGISTER.md, TRT_CHAINING_SPEC_v0.1.md.

## Revision History

Revision 0.1, 2026-10-30, TRT project: baseline with the changes known to date.

## Approval

Author, reviewer, approver: names, signatures and dates TBD.

## Main Content

### Change procedure

Step 1: anyone opens a change request with an ID CDR-NNN, using the template below.
Step 2: the area owner evaluates the impact.
Step 3: the approver decides to approve, reject or defer.
Step 4: the change is implemented and the commit or revision is linked.
Step 5: the change is verified and the result is recorded in TEST_RESULTS.md.
Step 6: the request is closed.
Changes to frozen items always need formal approval.

### Change log

CDR-001, 2026-10, host and firmware. Build and flash for STM32 were separated into trt build stm32 and trt flash stm32, and trt list-programs stm32 was added. Reason: a clear workflow with several applications. Reference: TRT-cli commit 501494f. Verification: Not Validated; 21 unit tests pass, but the real toolchain was never run. Approval: TBD.

CDR-002, 2026-10, firmware. The STM32 workspace became a multi-application project: main.c launcher, app_launcher.c, program files and selection by ACTIVE_PROGRAM. Reason: reuse one project and one TRT-Core. Reference: TRT-core commit 8865ee1. Verification: Not Validated; never compiled. Approval: TBD.

CDR-003, 2026-10, firmware. Cross-platform alignment: SystemService registers PING, INFO and STATUS for every platform, the Uno handler table grew from 4 to 12 entries, and the simulator error mapping was aligned. Reason: every platform answers the same. Reference: TRT-core commit 8865ee1. Verification: Not Validated; never compiled. Approval: TBD.

CDR-004, 2026-10, protocol, firmware and host. Query commands UPTIME (0x0008), RUNTIME (0x0009) and ACTIVE_PROGRAM (0x000A), the INFO field 0x06 (family), and the trt board query commands. Reason: read-only board information. Reference: TRT-core commit b0891c4 and TRT-cli commit f7182f7. Verification: host Validated, firmware Not Validated. Approval: TBD.

CDR-005, 2026-10, documentation. A review of the query commands found no clear code duplication; a "which command to use" section was added to the CLI reference. Reference: TRT-cli commit f7182f7. Verification: Implemented. Approval: TBD.

CDR-006, 2026-10, hardware. The daughter-board KiCad project was created with a root sheet, four common sheets and two variant sheets (potentiometers and keypad), and a PCB outline skeleton. Reason: a reusable module platform. Reference: TRT_HW, uncommitted when this document was written. Verification: the schematic editor ERC reported zero violations; no electrical review, so Not Validated. Approval: TBD.

CDR-007, 2026-10-30, architecture. The chaining specification version 0.1: roles, I2C segments, the 20-position TRT Port pinout, family HWID, descriptor, routing by BOARD_ID and CRC16. Reason: freeze the decisions before layout. Reference: TRT_CHAINING_SPEC_v0.1.md. Verification: Planned; nothing is implemented. Approval: pending signature.

CDR-008, 2026-10-30, documentation. The documentation baseline in TRT_DOCS/08_DELIVERABLES. Reference: this folder. Verification: Implemented. Approval: TBD.

### Change request template

Title: CDR-NNN, short title.
Date: TBD.
Requester: TBD.
Area: hardware, firmware, host, protocol or documentation.
Affected items: TBD.
Priority: high, medium or low.
Description of the change: TBD.
Reason: TBD.
Alternatives considered: TBD.

Impact analysis:
Hardware (schematic, PCB, bill of materials): TBD.
Firmware: TBD.
Host: TBD.
Protocol and compatibility: TBD.
Tests required: TBD.
Documentation: TBD.
Cost and schedule: TBD.
New risks, to be added to RISK_REGISTER.md: TBD.

Decision: approved, rejected or deferred. Date and signature: TBD.
Implementation reference, commit or revision: TBD.
Verification result: TBD.
Closure date: TBD.

### Open decisions that will generate changes

Mating geometry, hole pattern and module size (chaining specification O-01 and O-04).
Physical magnetic connector and pin count (O-02 and O-03).
Number of downstream ports per carrier (O-12).
Carrier 3.3 V regulator: the README says AMS1117-3.3, the power sheet shows AP2112K-3.3 (O-14).
Carrier connector size, 16 or 20 pins depending on the document (O-13).
CRC_VALID flag value and the protocol patch (O-10).

## Future Placeholders

Release notes and version tags: TBD.
Configuration baseline definitions: TBD.
Change control board membership: TBD.
Links to a change tracking tool: TBD.

## Appendices

Appendix A: closed change requests. Placeholder.
Appendix B: impact analysis checklists. Placeholder.
Appendix C: baseline snapshots. Placeholder.
