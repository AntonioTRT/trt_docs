# Risk Register

Document ID: TRT-RSK-001
Version: 0.1
Date: 2026-10-30
Status: Living document. Review at every milestone.
Owner: TBD

## Purpose

This document lists project risks with their rating, mitigation and status. It is a reusable template for any project, filled in for TRT.

## Scope

Technical, environmental, documentation, compliance and project risks for TRT.

## Definitions

Likelihood and impact are each rated from 1 (low) to 5 (high). Score is likelihood times impact. A score of 15 or more is high and needs an owner and a dated action. A score from 8 to 14 is medium. Below 8 is low.
Risk status: Open, Mitigating, Accepted, Closed.
Status markings: Implemented, Validated, Not Validated, Planned, TBD.

## References

ROADMAP.md, TEST_PLAN.md, TRT_CHAINING_SPEC_v0.1.md, CHANGE_DESIGN_REVISION.md.

## Revision History

Revision 0.1, 2026-10-30, TRT project: initial register from the reviews to date.

## Approval

Author, reviewer, approver: names, signatures and dates TBD.

## Main Content

### How to add a risk

Give an ID in the form RSK-NNN, a description, the area, likelihood, impact, score, mitigation, owner and status. Record changes in the review log.

### Register

RSK-001. Firmware changes were never compiled or run, so build errors may be hidden. Area: firmware. Likelihood 4, impact 4, score 16. Mitigation: install toolchains, compile all platforms and run the simulator tests (milestones M1 and M2). Owner TBD. Open.

RSK-002. No toolchain on the development machine blocks build, flash and validation. Area: environment. Likelihood 5, impact 4, score 20. Mitigation: install CMake, arm-none-eabi-gcc, ninja and STM32CubeProgrammer, or use another machine. Owner TBD. Open.

RSK-003. Heartbeat has never been observed on hardware. Area: firmware. Likelihood 3, impact 3, score 9. Mitigation: rebuild, flash and run trt listen. Owner TBD. Open.

RSK-004. Uno memory may be exceeded after the handler table grew to 12 entries. Area: firmware. Likelihood 3, impact 3, score 9. Mitigation: re-measure after the build. Owner TBD. Open.

RSK-005. CRC16 is not computed, so corrupted frames go undetected. Area: protocol. Likelihood 3, impact 4, score 12. Mitigation: implement CCITT-FALSE, mandatory on chain links. Owner TBD. Open.

RSK-006. The four copies of the frame state machine drift apart. Area: firmware. Likelihood 3, impact 3, score 9. Mitigation: fix in all copies or merge when framing changes. Owner TBD. Accepted.

RSK-007. Chaining changes the carrier connector after PCB progress. Area: hardware. Likelihood 4, impact 5, score 20. Mitigation: decide now, while the carrier is still in placement. Owner TBD. Open.

RSK-008. Magnetic contacts bounce or carry too little current. Area: hardware. Likelihood 3, impact 4, score 12. Mitigation: debounce, retries, and early tests with the real part (milestone M6). Owner TBD. Open.

RSK-009. Few ground pins cause ground bounce. Area: hardware. Likelihood 3, impact 3, score 9. Mitigation: two grounds on the 20-position port, with a 2 by 12 fallback. Owner TBD. Open.

RSK-010. USB power budget is exceeded by LED modules. Area: hardware. Likelihood 3, impact 3, score 9. Mitigation: descriptor current, budget check and a per-port limit. Owner TBD. Open.

RSK-011. The 50 by 50 mm outline does not fit the planned modules. Area: hardware. Likelihood 5, impact 3, score 15. Mitigation: adopt the 100 by 100 mm panel tile on a 50 mm grid. Owner TBD. Open.

RSK-012. Daughter holes do not match the carrier's real hole pattern. Area: hardware. Likelihood 4, impact 3, score 12. Mitigation: fix the mating geometry before layout. Owner TBD. Open.

RSK-013. Two sources of truth for the carrier: README and schematic disagree on the regulator part and the connector size. Area: documentation. Likelihood 4, impact 3, score 12. Mitigation: confirm against the schematic and correct the README. Owner TBD. Open.

RSK-014. PB8 shared with BOOT0 starts the board in the bootloader. Area: hardware and firmware. Likelihood 3, impact 3, score 9. Mitigation: set the option bytes at first boot. Owner TBD. Open.

RSK-015. Hot-plug glitches disturb a bus. Area: hardware and firmware. Likelihood 3, impact 4, score 12. Mitigation: segmented I2C, high-impedance modules and bus recovery. Owner TBD. Open.

RSK-016. Two masters are connected. Area: firmware. Likelihood 2, impact 3, score 6. Mitigation: role detection and ERROR_DUAL_MASTER. Owner TBD. Open.

RSK-017. The corporate network blocks PlatformIO downloads. Area: environment. Likelihood 4, impact 2, score 8. Mitigation: truststore in upload.py, and an offline mirror (TBD). Owner TBD. Mitigating.

RSK-018. Five failing tests in TRT-cli are not understood. Area: host. Likelihood 3, impact 2, score 6. Mitigation: investigate whether the tests or the code are wrong. Owner TBD. Open.

RSK-019. Older documents (future-protocol and a version 1 frame) conflict with version 0.1. Area: documentation. Likelihood 3, impact 2, score 6. Mitigation: mark them design-only and treat the wire-format document as current. Owner TBD. Open.

RSK-020. There is no regulatory or EMC plan. Area: compliance. Likelihood 3, impact 4, score 12. Mitigation: define the applicable standards (TBD). Owner TBD. Open.

RSK-021. Knowledge sits with few people and there is no continuous integration. Area: project. Likelihood 3, impact 3, score 9. Mitigation: add continuous integration for pytest and the C++ tests. Owner TBD. Open.

### Review log

2026-10-30: initial register created by the TRT project.

## Future Placeholders

Risk burn-down chart: TBD.
Contingency plans for high risks: TBD.
Supplier and supply-chain risks: TBD.
Safety risk analysis: TBD.

## Appendices

Appendix A: risk scoring examples. Placeholder.
Appendix B: closed risks. Placeholder.
Appendix C: lessons learned. Placeholder.
