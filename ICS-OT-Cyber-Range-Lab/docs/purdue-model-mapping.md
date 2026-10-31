# Purdue Model Asset Matrix

| Purdue Level | Functional Layer | Asset(s) | Static IP | Role |
|---|---|---|---|---|
| — | Management Bridge | ESXi Hypervisor Host | `10.0.0.150` | Bare-metal compute engine for the whole lab |
| — | Central Management | vCenter Server Appliance (VCSA) | `10.0.0.200` | Orchestration, clustering, migration tracking |
| 5 | Enterprise Stack | OT_KALI | `192.168.169.35` | Penetration auditing / network validation node |
| 5 | Enterprise Stack | OT_SIEM (Security Onion-style) | `10.0.0.181` | IDS/telemetry, fed via port mirroring |
| 5 | Enterprise Stack | OTvCenterServer | `10.0.0.200` | Hypervisor platform orchestration |
| 4 | Site Business Systems | Site Business Workstations | lab-assigned | ERP / reporting / config logging |
| 3 | Operations & Control | OT_ENG_WORKSTATION | `192.168.169.30` | PLC logic authoring & deployment |
| 2 | Local Control | OT_SCADA | `192.168.169.20` | HMI + site telemetry aggregation |
| 1 | Process Systems | OT_PLC (virtual) | `192.168.169.70` | Virtualized PLC, local automation loops |
| 1 | Process Systems | OT_PHYSICAL_PLC — 1794-AENTR (Rockwell/AB) | `192.168.169.60` | Physical EtherNet/IP Flex I/O adapter |
| 1 | Process Systems | OT_PHYSICAL_PLC — Schneider Electric M221 | `192.168.169.60`* | Physical Modicon M221 PLC, Modbus TCP control logic |
| 0 | Physical I/O | Field Sensors & Actuators | — | Raw analog/digital field instrumentation |

\* `192.168.169.60` is also used for the 1794-AENTR in the [1794-AENTR project](../pentest-projects/1794-AENTR-dos-test/). The two physical devices are connected and tested one at a time, reusing the same address — they are not live on the bench simultaneously. If you later run both concurrently, assign the M221 a separate address (e.g. `192.168.169.61`) and update this table.

## Subnetting

- **Management/Security plane:** `10.0.0.0/24`
- **ICS/OT operational plane:** `192.168.0.0/16` (lab instance: `192.168.169.0/24`)

## Segmentation rules

- Level 5/4 ↔ Level 3/2: stateful inspection + industrial-protocol DPI required.
- Level 5/4 → Level 1/0: direct connections explicitly denied.
- All cross-level flows are mirrored to the Level 5 SIEM for monitoring.
- Hypervisor/vCenter management endpoints are reachable only via a dedicated, MFA-gated jump path.
