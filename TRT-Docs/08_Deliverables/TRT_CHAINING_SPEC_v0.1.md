# TRT Chaining Specification v0.1

| Field | Value |
|---|---|
| Document ID | TRT-CHN-001 |
| Version | 0.1 (draft for approval) |
| Date | 2026-10-30 |
| Status | **Proposed**: decisions marked *Frozen* take effect when the approval table in section 0.2 is signed |
| Applies to | TRT Carrier Board (STM32G474), TRT Daughter Board platform, TRT-Core runtime, TRT Protocol V0.1 (frame layout unchanged) |
| Owner | TBD |

## 0. Document control

### 0.1 Revision history

| Rev | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-10-30 | TRT project | First complete draft |
| | | | |

### 0.2 Approval

| Role | Name | Signature | Date |
|---|---|---|---|
| Hardware lead | | | |
| Firmware lead | | | |
| Project owner | | | |

### 0.3 Status vocabulary

| Word | Meaning |
|---|---|
| **Frozen** | Decided by this specification. Changing it needs a change record (see `08_DELIVERABLES/CHANGE_DESIGN_REVISION.md`). |
| **Open** | Needs an owner decision before PCB layout. A recommendation is given. |
| **Reserved** | Name or range set aside; no behaviour defined yet. |

Nothing in this specification is implemented in firmware or hardware yet. Section 19 lists what must change.

## 1. Purpose and scope

Fix the architecture decisions that affect both the Carrier Board and the Daughter Board platform, so PCB layout can start without rework.

In scope: roles, topology, connector signals, power, ground, identification, discovery, I2C, addressing, routing, link layer, CRC, hot-plug, errors, MCU modules, expansion.

Out of scope: PCB layout and footprints, module-specific register maps (section 17 only sets the rule), enclosure design, magnetic connector part selection, firmware implementation.

The TRT Protocol V0.1 frame layout (`protocol-v0.1-wire-format.md`) is not changed. This specification only allocates values inside existing fields (BOARD_ID) and reserves new command, event and error IDs.

## 2. Terms

| Term | Definition |
|---|---|
| Carrier | STM32G474 board running TRT-Core |
| Daughter / Module | A function board attached to a carrier port |
| Node | A carrier in a chain |
| Master | The node connected to the host over USB. It terminates the host protocol. |
| Slave | A node reached through an upstream port, with no host connection |
| Standalone | A carrier with no host and no upstream node |
| DP | Downstream Port: the carrier side of a TRT Port |
| UP | Upstream Port: the module side of a TRT Port |
| Segment | The I2C link between one DP and the one occupant of the UP it connects to |
| Path ID | A BOARD_ID that encodes the route to a node |

## 3. Decision summary

| ID | Decision | Status |
|---|---|---|
| D-01 | Roles are Master, Slave, Standalone. Role follows USB attachment. | Frozen |
| D-02 | One connector definition, **TRT Port**, with two sides: DP (carrier) and UP (module or slave carrier). A slave carrier looks like a self-describing module to its upstream. | Frozen |
| D-03 | Topology is a tree rooted at the master. Each link is a separate point-to-point I2C **segment**. No shared multi-node bus. | Frozen |
| D-04 | One occupant per DP. Maximum chain depth 3 for v0.1. | Frozen |
| D-05 | Single I2C master per segment (the DP side). No multi-master. | Frozen |
| D-06 | I2C speed 100 kHz for v0.1. 400 kHz only after validation. | Frozen |
| D-07 | Fixed segment-local I2C addresses: node link 0x10, descriptor 0x50. No global address assignment. | Frozen |
| D-08 | Routing by BOARD_ID. Path IDs 0x1000-0x1FFF encode the route. Static IDs 0x0001-0x0FFF stay as they are today. | Frozen |
| D-09 | HWID is 4 bits and identifies the module **family**. 0x0 = no module, 0xF = fault. Detail comes from the descriptor. | Frozen |
| D-10 | Module descriptor: 32 bytes at I2C address 0x50, CRC16 protected. | Frozen |
| D-11 | Chain link frames carry a real CRC16 (CCITT-FALSE). The host link gains it through a FLAGS bit. | Frozen |
| D-12 | 3.3 V is local to each carrier. Only 5 V crosses a link. The DP switches 5 V per port. | Frozen |
| D-13 | Connector logical pinout is **TRT Port v1, 20 positions**. Pins 1-16 are unchanged from the current carrier J2. | Frozen (logical); physical part Open |
| D-14 | MCU-based modules are allowed from the start. They use HWID 0x1 and the same descriptor and register rules. | Frozen |
| D-15 | Hot-plug is supported on every port, using the state machine in section 15. | Frozen |
| D-16 | Number of DPs per carrier, and the physical arrangement of UP and DP connectors on the carrier. | **Open** (section 20) |

## 4. Architecture overview

```text
 Host PC (TRT-CLI)
      | USB (TRT frames over serial)
 +----------------+  DP1  segment (I2C, point to point)   +----------------+
 | Carrier A      |-------------------------------------->| Carrier B      |  Slave
 | MASTER         |  UP                                   | UP    DP1      |
 | id 0x00C9      |                                       | id 0x1100      |
 +----------------+                                       +----------------+
      | DP2  segment                                           | DP1 segment
      v                                                        v
 +----------------+                                       +----------------+
 | Daughter board |                                       | Daughter board |
 | (passive,      |                                       | (MCU based,    |
 | HWID family)   |                                       | HWID 0x1)      |
 +----------------+                                       +----------------+
```

Every arrow is a TRT Port. The carrier on the left of an arrow is the master of that segment. The thing on the right is a passive module, an MCU module, or a slave carrier.

## 5. Roles and responsibilities

### 5.1 Role selection (D-01)

| State | Condition | Behaviour |
|---|---|---|
| Master | A USB host session is active | Terminates the host protocol. Enumerates its DPs. |
| Slave | No USB session, valid 5 V on UP, and I2C activity from upstream | Answers on UP. Masters its own DPs. Never talks to a host. |
| Standalone | Neither | Runs its application. Masters its own DPs. Emits no host traffic. |

Role is evaluated at boot and on USB attach or detach. A carrier that has a USB session **and** sees an upstream master on its UP reports `ERROR_DUAL_MASTER`, keeps its UP target disabled, and continues as an isolated master. Neither carrier is damaged.

### 5.2 Carrier responsibilities

1. Own the DP side of every segment: I2C master, pull-ups, HWID pull-downs, 5 V switch, INT_N input.
2. Detect, power, identify and enumerate whatever is on each DP (section 11).
3. Route frames by BOARD_ID (section 13) and forward them across segments.
4. Expose attached modules as TRT-Core modules and capabilities.
5. Enforce the power budget (section 8).
6. Report topology changes to the host (master only).
7. As a slave: serve the UP side (section 6.2) and look like a self-describing module upstream.

### 5.3 Daughter board responsibilities

1. Present a valid HWID and, where it has memory, a descriptor.
2. Tolerate being plugged in while the carrier is powered. No power sequencing may be assumed.
3. Keep all signals high impedance until its own supply is good.
4. Stay inside its declared 5 V and 3.3 V current.
5. Own no pull-ups on SDA, SCL or INT_N (the DP supplies them), unless it is an MCU module that is also a DP master of its own segment.
6. Carry ESD protection on every connector signal.

## 6. Topology and ports

### 6.1 Rules (D-03, D-04, D-05)

1. A carrier has one UP (optional, only used in slave role) and one or more DPs.
2. A DP connects to exactly one UP. A DP never shares a bus with another DP.
3. Each segment has a single I2C master: the DP side.
4. Maximum depth is 3 (master = depth 0, up to three slave levels below it).
5. Each segment has its own capacitance budget, so depth does not accumulate bus load.

Why segments instead of one shared bus: no address collisions between modules of the same family (two keypads both use their fixed addresses), capacitance and hot-plug glitches stay local, and every carrier keeps its own module control.

### 6.2 Side behaviour

| Item | DP (carrier side) | UP (module or slave side) |
|---|---|---|
| I2C | Master | Target at 0x10 (link) and 0x50 (descriptor) |
| SDA/SCL pull-ups | Yes (2.2 kOhm to 3.3 V, switchable) | None |
| HWID_0..3 | 10 kOhm pull-down each, read as input | Strap to its **own** 3.3 V |
| INT_N | Input with pull-up | Open-drain output |
| +5 V | Switched, current limited, output | Input (diode-protected against back-feed) |
| +3.3 V | Output (always on) | Not used as a supply (section 8) |
| MOD_RST_N | Output, open-drain | Input |

## 7. Connector: TRT Port v1 (D-13)

Logical pinout. Pins 1-16 are the current carrier J2 (carrier README section 3.1) and do not move. Physical connector (2.00 mm header and socket for prototypes, magnetic contact connector later) is Open.

| Pin | Signal | Role |
|---|---|---|
| 1 | SPI_CS | SPI chip select (display modules) |
| 2 | GND | Ground |
| 3 | SPI_MOSI | SPI |
| 4 | +5V | Switched 5 V from DP |
| 5 | SPI_MISO | SPI |
| 6 | TRT_GPIO_1 | General purpose |
| 7 | SPI_SCK | SPI |
| 8 | +3V3 | 3.3 V from DP (passive modules only) |
| 9 | HWID_0 | Family code bit 0 |
| 10 | TRT_GPIO_2 | General purpose |
| 11 | HWID_1 | Family code bit 1 |
| 12 | SDA | I2C |
| 13 | HWID_2 | Family code bit 2 |
| 14 | SCL | I2C |
| 15 | HWID_3 | Family code bit 3 |
| 16 | TRT_GPIO_3 | General purpose |
| 17 | INT_N | Module interrupt, open-drain, active low (**new**) |
| 18 | GND | Second ground (**new**) |
| 19 | MOD_RST_N | Module reset / boot request (**new**) |
| 20 | +5V | Second 5 V pin (**new**) |

Rules:
1. GND is on pins 2 and 18. Do not route fast signals (SPI) with no ground neighbour.
2. SPI is claimed by at most one module per DP. Display modules use GPIO for DC, reset and backlight.
3. All signals are 3.3 V logic. 5 V is for loads only.
4. 2x10 is the recommended size. A 2x12 with more ground pins is the fallback if validation shows ground bounce (Open, section 20).

## 8. Power and grounding

### 8.1 Power (D-12)

1. USB 5 V is the only external source. The master supplies the chain.
2. Each carrier makes its own 3.3 V from 5 V. 3.3 V never crosses a link as a supply, so rails cannot fight. A slave carrier may sense the UP 3.3 V pin through a series resistor but does not connect it to its own rail.
3. Each DP has a 5 V load switch with current limit and soft start. 5 V is enabled only after the HWID is valid (section 15).
4. A slave carrier takes 5 V from UP through a diode, OR-ed with its own VBUS, so it never back-feeds the upstream DP.
5. Budget: every node declares its maximum current in the descriptor. The master sums the chain before enabling a port and refuses with `ERROR_POWER_BUDGET` if the total would exceed the budget. Default source budget is the USB default of **500 mA** (assumption A-04).
6. Per-port default limit when no descriptor exists: **150 mA**, until each family declares its own (Open O-07).

### 8.2 Grounding

1. One common ground reference through the whole chain. No isolation in v0.1.
2. At every port, ground is carried on at least two pins (2 and 18), interleaved with the fast signals.
3. Each board uses a solid ground plane. Analog sections (pots, ADC) use a filtered supply (ferrite from 3.3 V), **not** a split ground.
4. The USB connector shield connects to ground through the arrangement on the carrier (1 MOhm in parallel with a 4.7 nF capacitor is the usual pattern; detail belongs in the Hardware Design Description).

## 9. HWID (D-09)

The HWID is read on the four HWID pins. The DP has 10 kOhm pull-downs, so an absent module reads 0x0. The UP straps the bits it wants set to its own 3.3 V.

| Code | Meaning |
|---|---|
| 0x0 | No module |
| 0x1 | **Self-describing**: read the descriptor (all MCU modules and slave carriers) |
| 0x2 | Analog family (potentiometers, sliders) |
| 0x3 | Switch matrix family (keypads) |
| 0x4 | Encoder family |
| 0x5 | I2C display (OLED) |
| 0x6 | SPI display (TFT) |
| 0x7 | Addressable LED (rings, strips) |
| 0x8-0xB | Reserved for future families |
| 0xC-0xD | Experimental / custom (never in production) |
| 0xE | Factory test fixture |
| 0xF | Fault or invalid (all lines high) |

Rules:
1. HWID identifies the **family** and thereby selects the carrier driver. Variant, revision and serial come from the descriptor.
2. The carrier accepts a code only after 5 consecutive identical reads, 10 ms apart.
3. Analog (ternary) encoding of the HWID lines would give more codes without a hardware change on the carrier (the pins are ADC capable). It is **not** adopted in v0.1 (Open O-09).

## 10. Module descriptor (D-10)

32 bytes at I2C address **0x50**, offset 0. An EEPROM on passive modules, emulated by an MCU on active modules and slave carriers. Multi-byte fields are big-endian.

| Offset | Size | Field | Notes |
|---|---|---|---|
| 0 | 2 | Magic | 0x54 0x4D ('TM') |
| 2 | 1 | Descriptor version | 0x01 |
| 3 | 1 | Family | Must equal the HWID, else FAULT |
| 4 | 2 | Variant | Family-specific (for example 4 or 8 pots) |
| 6 | 1 | HW revision major | |
| 7 | 1 | HW revision minor | |
| 8 | 8 | Serial | Unique |
| 16 | 1 | Primary I2C address | Module's own register or IC address (0x10 for slave carriers) |
| 17 | 1 | Flags | bit0 MCU module, bit1 needs 5 V, bit2 uses INT_N, bit3 uses SPI |
| 18 | 2 | Max I2C speed (kHz) | |
| 20 | 2 | Max 3.3 V current (mA) | |
| 22 | 2 | Max 5 V current (mA) | |
| 24 | 6 | Reserved | Zero |
| 30 | 2 | CRC16 | CCITT-FALSE over bytes 0-29 |

A module without a descriptor is allowed only for a passive family, and then the family default values apply. A descriptor with a bad CRC puts the port into FAULT.

## 11. Discovery and enumeration

1. DP idle: read HWID. 0x0 means nothing is attached.
2. HWID stable and not 0x0 or 0xF: go to POWER_UP (section 15).
3. Read the descriptor (3 attempts). Compare family against the HWID.
4. Passive family: load the family driver and expose its capabilities.
5. Self-describing (0x1): read the descriptor. If the descriptor says slave carrier, send `CHAIN_ASSIGN_ID` (section 13), then request its topology, and repeat for its DPs.
6. Master only: emit `TOPOLOGY_CHANGED` to the host.

Enumeration runs at boot, on every insertion event, and on `CHAIN_TOPOLOGY` request. A node is never assumed to exist; presence is re-checked by the heartbeat (a slave's heartbeat is forwarded to the host with the slave's path ID) and by polling.

## 12. I2C (D-05, D-06, D-07)

| Item | Decision |
|---|---|
| Mode | Single master per segment |
| Speed | 100 kHz (v0.1). 400 kHz needs validation and a descriptor that allows it. |
| Pull-ups | On the DP, 2.2 kOhm, switchable (existing PC15 scheme) |
| Capacitance target | 150 pF per segment at 400 kHz, 400 pF at 100 kHz |
| Addresses | Segment-local. 0x10 = TRT node link. 0x50 = descriptor. Family ICs use their vendor addresses inside a block reserved per family (kept in the HWDD). 0x00-0x07 and 0x78-0x7F are never used. |
| Stuck bus | Master sends up to 9 clock pulses and a STOP, then FAULT if SDA is still low |
| STM32 note | Targets use dual own-address (0x10 and 0x50) |

Because every DP has one occupant, two keypads on two DPs use the same address with no conflict.

## 13. Routing and BOARD_ID (D-08)

BOARD_ID is a 2-byte field in the frame; the layout is unchanged.

| Range | Use |
|---|---|
| 0x0000 | Invalid |
| 0x0001-0x0FFF | Static board IDs (today: 2, 101, 201, 301). Used by masters and standalone boards. |
| 0x1000-0x1FFF | **Path IDs** for slave nodes |
| 0x2000-0xFFFE | Reserved |
| 0xFFFF | Reserved for broadcast |

Path ID = `0x1000 | (p1 << 8) | (p2 << 4) | p3`, where p1..p3 are DP numbers (1-15) taken at each hop, and unused nibbles are 0. Example: 0x1100 is the node on DP1 of the master, 0x1120 is on DP2 of that node.

Forwarding rule (stateless): a node compares the frame's BOARD_ID with its own ID. Equal: handle. Otherwise take the next non-zero nibble after its own depth, and forward out that DP. A path ID with no matching port gets `ERROR_NODE_UNREACHABLE`.

The master assigns a path ID to each slave with `CHAIN_ASSIGN_ID`. A slave replies with its own path ID in BOARD_ID, so the host knows the origin. SEQ_ID is preserved end to end. The host reaches each master over its own serial port, so the same path ID under two masters is not ambiguous (the CLI already keys on port plus board ID).

## 14. Link layer and CRC (D-11)

### 14.1 TRT over I2C

TRT V0.1 frames travel unchanged over I2C. Frame size is at most 128 bytes (payload at most 114) in v0.1. The slave link is a small register interface at address 0x10:

| Register | Access | Content |
|---|---|---|
| 0x00 LINK_STATUS | Read | 2 bytes: pending upstream frame length (0 = none) |
| 0x01 RX_FRAME | Read | The pending frame (length from LINK_STATUS) |
| 0x02 TX_FRAME | Write | A complete frame from the master |

The slave asserts INT_N when a response or event is pending and releases it when the master has read the whole frame. If INT_N is not wired or stays low for more than 100 ms without a read, the master falls back to polling LINK_STATUS every 20 ms and reports a fault.

Timing budget (targets, to be validated): 5 ms per I2C transaction; a 128-byte frame takes about 14 ms at 100 kHz; end-to-end response timeout is 100 ms per hop.

### 14.2 CRC

| Item | Value |
|---|---|
| Algorithm | CRC-16/CCITT-FALSE: polynomial 0x1021, init 0xFFFF, no reflection, final xor 0x0000 |
| Covered bytes | VERSION through the end of PAYLOAD (SYNC and CRC excluded) |
| Placement | CRC16 field, big-endian |
| Check value | CRC of ASCII "123456789" = **0x29B1** |
| Test vector | PING, board 0x00C9, seq 1: frame `AA 55 01 00 00 C9 00 01 00 01 00 00 4E C4` |

Rules:
1. **Chain links**: CRC is mandatory. A frame with a bad CRC is dropped and counted. The sender retries up to 2 times, then reports `ERROR_LINK_CRC`.
2. **Host link**: V0.1 sends 0x0000. A new FLAGS bit **0x10 CRC_VALID** (proposed) says the CRC field is real. Hosts and boards that ignore the flag keep working. Enabling it needs a protocol patch to the wire-format document and both runtimes.
3. The STM32 hardware CRC unit can compute this polynomial.

## 15. Hot-plug (D-15)

| State | Entry | Actions | Exit |
|---|---|---|---|
| ABSENT | Start, or removal | 5 V off. HWID polled every 50 ms. | HWID not 0x0 |
| DEBOUNCE | HWID changed | 5 reads, 10 ms apart | stable and valid -> POWER_UP; else ABSENT |
| POWER_UP | Valid HWID | Budget check, enable 5 V with soft start (10 ms), wait 100 ms for module boot | done -> IDENTIFY; over budget -> FAULT |
| IDENTIFY | Powered | Read descriptor (3 tries), check family | ok -> ACTIVE; else FAULT |
| ACTIVE | Identified | Normal operation, presence monitored | removal or 3 consecutive I2C failures -> ABSENT |
| FAULT | Any error | 5 V off, I2C released, error reported | re-insertion (HWID 0x0 then valid) -> DEBOUNCE; automatic retry after 1 s up to 3 times |

Removal is detected by HWID reading 0x0 for 20 ms, or three failed transactions in a row. On removal the carrier switches 5 V off within 10 ms, removes the module and capabilities, and the master emits `TOPOLOGY_CHANGED`. Removing a slave carrier removes its whole subtree.

Module rules: the UP strap is valid as soon as its supply is on, but a module may not drive any signal until its supply is good.

## 16. Error handling

| Layer | Fault | Handling |
|---|---|---|
| Frame | Bad length, SYNC or VERSION | Dropped by the parser (existing rule) |
| Link | CRC failure | Drop, count, sender retries 2 times, then `ERROR_LINK_CRC` |
| Link | I2C NACK or timeout | Retry 2 times, then mark unreachable after 3 consecutive failures |
| Bus | SDA stuck low | 9-pulse recovery, then port FAULT |
| Port | Overcurrent, over budget | Switch off, FAULT, `ERROR_POWER_BUDGET` |
| Port | HWID and descriptor disagree | FAULT, `ERROR_MODULE_FAULT` |
| Topology | Depth over 3 | Refuse, `ERROR_TOPOLOGY_DEPTH` |
| Role | Two masters | `ERROR_DUAL_MASTER`, UP disabled |

Existing error IDs (0x0001-0x0006) are unchanged. Reserved new IDs, returned as NACK payloads:

| ID | Name |
|---|---|
| 0x0010 | ERROR_NODE_UNREACHABLE |
| 0x0011 | ERROR_LINK_CRC |
| 0x0012 | ERROR_BUS_FAULT |
| 0x0013 | ERROR_MODULE_FAULT |
| 0x0014 | ERROR_POWER_BUDGET |
| 0x0015 | ERROR_DUAL_MASTER |
| 0x0016 | ERROR_TOPOLOGY_DEPTH |

## 17. MCU-based modules (D-14)

1. An MCU module is a self-describing module (HWID 0x1) with a descriptor at 0x50 and a **register interface** on its primary address.
2. The register interface has a common header (ID, status, interrupt flags, interrupt acknowledge), then family-specific registers. A passive module's family driver on the carrier implements the same interface in software, so carrier code above the driver does not change. The header layout is a future document (v0.2).
3. INT_N means "read my interrupt flags". Level is held until acknowledged.
4. Firmware update: programming pads on the module are used in production. In the field, an MCU module enters its bootloader when the carrier pulls MOD_RST_N low in a defined pattern (reserved; detail in v0.2).
5. A slave carrier is a special MCU module whose primary address 0x10 is the node link.

## 18. Future expansion

| Area | Reserved |
|---|---|
| Commands | 0x0020-0x003F chain and module management: 0x0020 CHAIN_ASSIGN_ID, 0x0021 CHAIN_TOPOLOGY, 0x0022 MODULE_LIST (proposed names, not yet in `command-identifiers.md`). 0x0100-0x01FF module access. |
| Events | 0x9002 TOPOLOGY_CHANGED, 0x9003 MODULE_EVENT |
| Errors | 0x0010-0x001F as in section 16 |
| HWID | 0x8-0xB for new families, 0xC-0xD experimental |
| Descriptor | Version byte, 6 reserved bytes |
| Connector | A future version may add ground pins (2x12) or use ternary HWID |
| Speed | 400 kHz after validation; a later link layer may use SPI between carriers |
| Depth | Raise above 3 by extending the path ID (needs a protocol patch) |

## 19. Impact on the boards

### 19.1 Carrier Board (current: STM32G474, J2 2x08, PB9/PB8 I2C1)

1. Connector: grow J2 from 16 to 20 positions (INT_N, GND, MOD_RST_N, +5V). Assign MCU pins.
2. Add a second I2C peripheral for the UP target role, or confirm that the existing hardware gives separate nets for UP and DP (the current PCB has a socket and a header with the same reference; verify the nets).
3. Add a 5 V load switch with current limit per DP.
4. Add diode OR-ing of VBUS and UP 5 V.
5. Add the UP strap for HWID = 0x1 (to its own 3.3 V).
6. Keep the existing 10 kOhm HWID pull-downs on every DP.
7. Provide an interrupt-capable pin for INT_N per DP, and one open-drain output for MOD_RST_N.
8. Decide the number of DPs (Open D-16).

### 19.2 Daughter Board platform (current KiCad skeleton)

1. Connector sheet becomes 20 positions with the pin table in section 7.
2. `MOD_INT_N` comes from pin 17, not a borrowed GPIO.
3. HWID codes change to the allocation in section 9. The current draft (0x1 pots, 0x2 keypad) is superseded.
4. Add the descriptor EEPROM at 0x50 on passive modules.
5. Second GND and second 5 V pins are used on the connector.
6. Mounting-hole pattern and outline follow the mating decision (Open O-01).

## 20. Open decisions

| ID | Decision | Recommendation |
|---|---|---|
| O-01 | Mating geometry: which carrier connector mates, whether the daughter flips, stack height, hole pattern (the carrier's real holes are not symmetric) | Decide from the carrier PCB before any daughter layout |
| O-02 | Magnetic connector part for 20 positions, contact current rating | Prototype with 2.00 mm headers; select the magnetic part in parallel |
| O-03 | 2x10 or 2x12 | 2x10, validate ground bounce first |
| O-04 | Reference module size (50 mm cell, 100x100 mm panel tile) | Adopt the 50 mm cell and 100x100 mm tile |
| O-05 | Maximum chain depth above 3 | Keep 3 until measured |
| O-06 | Frame size limit on the chain link | 128 bytes until measured |
| O-07 | Per-family default currents (3.3 V and 5 V) | Fill in at the first family design review |
| O-08 | USB current budget (500 mA vs more) | Treat as 500 mA until a source is chosen |
| O-09 | Ternary HWID | Not for v0.1 |
| O-10 | CRC_VALID flag value 0x10 and the host-link rollout | Approve with the protocol patch |
| O-11 | Firmware update path for modules | Define in v0.2 |
| O-12 | Number of DPs per carrier (D-16) | Two (one for a module, one for a downstream node) |
| O-13 | Carrier docs disagree on connector size (earlier notes mention 20 pins, README 16) | Confirm against the schematic |
| O-14 | LDO part on the carrier (README says AMS1117-3.3, the power sheet shows AP2112K-3.3) | Confirm against the schematic and size the 3.3 V budget |

## 21. Risks

| ID | Risk | Mitigation |
|---|---|---|
| R-01 | Magnetic contacts bounce or open under vibration | Debounce, retries, descriptor re-check, bench test with the real connector |
| R-02 | Single or few ground pins cause ground bounce on SPI and I2C | Two grounds plus validation, 2x12 fallback |
| R-03 | 5 V on one or two pins exceeds contact rating | Per-port current limit, rating check at part selection |
| R-04 | USB budget is exceeded by LED modules | Budget check, `ERROR_POWER_BUDGET` |
| R-05 | I2C slave on STM32 stretches the clock or misses addresses | Test at 100 kHz first, dual-address configuration test |
| R-06 | Hot-plug glitches disturb the DP bus | Segmented design, high-impedance modules, bus recovery |
| R-07 | Two masters connected | Role detection, `ERROR_DUAL_MASTER`, UP isolation |
| R-08 | Path ID limits depth | Depth 3 is enough for the stated goal, extension path reserved |
| R-09 | CRC rollout on the host link breaks old tools | Flag-based, old tools ignore it |
| R-10 | Carrier PCB already advanced; connector change costs a respin | Decide now (still at placement) |
| R-11 | Nothing here is implemented or validated | Validate per section 22 and the TEST_PLAN |

## 22. Validation checkpoints

| # | Checkpoint | Evidence |
|---|---|---|
| V-1 | Two carriers on a bench, jumper wires, 100 kHz link, PING forwarded through a slave | Logic analyzer trace, CLI output |
| V-2 | Descriptor read and HWID agreement on a passive module | Test record |
| V-3 | Hot-plug sequence with the real connector, 100 insertions | Count of correct detections |
| V-4 | Power budget refusal | Test record |
| V-5 | CRC test vector and corrupted frame rejection | Unit test |
| V-6 | Three-level chain | CLI topology output |

## 23. Assumptions

| ID | Assumption |
|---|---|
| A-01 | The STM32G474 has enough I2C peripherals for one target and one master per carrier. |
| A-02 | One module (or one slave carrier) occupies each DP. |
| A-03 | The host always connects to a master over one USB serial port. |
| A-04 | USB default power (500 mA at 5 V) is the available budget. |
| A-05 | 100 kHz is adequate for the control traffic in v0.1. |
| A-06 | The carrier PCB is still in layout, so the connector can change. |
| A-07 | TRT V0.1 frames stay valid over I2C with no change. |
| A-08 | All modules run from 5 V and 3.3 V only. |
| A-09 | The existing HWID pull-downs on the carrier stay as designed. |
| A-10 | BOARD_ID values 0x0001-0x0FFF are enough for static boards. |

## 24. Related documents

`protocol-v0.1-wire-format.md`, `command-identifiers.md`, `error-model.md` (TRT-core `docs/`); carrier README and the KiCad projects in `TRT_HW/hardware/`; `ARCHITECTURE.md`, `HWDD.md`, `SWDD.md` in this tree.
