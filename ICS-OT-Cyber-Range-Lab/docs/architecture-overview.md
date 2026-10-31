# Architecture Overview

## Hypervisor & storage

- **Hypervisor:** VMware ESXi, either bare-metal on-prem or nested/Azure Dedicated Host for a cloud-hosted variant.
- **Primary storage:** a dedicated datastore (`OTCYBERDATAStorage`), mounted via iSCSI/NFS for the hybrid variant, VMFS6 locally for the bare-metal build.
- **Management control plane:** static management IP on a dedicated management vSwitch/vmnic, reachable only through the designated jump path (no direct public exposure).

## Dual-network model

Two explicitly separated, non-overlapping routing planes keep enterprise/IT traffic away from production OT traffic:

| Plane | Range | Purpose |
|---|---|---|
| Management / Security | `10.0.0.0/24` | Hypervisor mgmt, vCenter, SIEM, cloud bridge/migration tooling |
| ICS/OT Operational | `192.168.0.0/16` (lab subnet used: `192.168.169.0/24`) | All Purdue Level 0–3 traffic; no default internet egress |

The OT plane is intentionally "locked down" — no outbound NAT, no dynamic DHCP egress — to eliminate unauthorized lateral movement or exfiltration paths out of the production segment.

## Perimeter & inspection controls

1. **Stateful/NGFW inspection** between Level 5/4 and Level 3/2, with deep packet inspection on industrial protocols (ModbusTCP, EtherNet/IP). Direct connections from Level 5/4 down to Level 0/1 are explicitly dropped.
2. **Port mirroring / virtual TAP** feeding a continuous copy of cross-level traffic into the SIEM/IDS node for near-real-time anomaly detection.
3. **Management access isolation** — the hypervisor/vCenter dashboards are reachable only through a designated, MFA-protected jump path; no direct multi-tenant or public routing to those endpoints.

## Virtual lab / cyber range topology

The `VMnet1` (host-only) virtual switch hosts the fully isolated simulation range used for attack/defense exercises, separate from the "real" hybrid infrastructure described above. It mirrors the same Purdue-level structure (L0 Field I/O → L5 Enterprise) so that techniques practiced in the range transfer directly to the production-style build. See `images/network-topology.png` for the full diagram, including the attacker node, monitored traffic table, and per-level alerting used during exercises.

### Physical process hardware

Two real Level 1 field controllers sit on the OT plane alongside the virtualized PLC, giving the range cross-protocol coverage instead of a single-vendor view:

- **Rockwell/Allen-Bradley 1794-AENTR** — Flex I/O EtherNet/IP (CIP) adapter. See the [1794-AENTR recon & load test project](../pentest-projects/1794-AENTR-dos-test/README.md).
- **Schneider Electric M221** — Modicon M221 PLC, controlled over Modbus TCP, used for Modbus-specific protocol exercises (as a counterpart to the EtherNet/IP work on the 1794-AENTR).

## Optional: hybrid cloud bridge (Azure Migrate)

For the hybrid variant, an on-prem staging node runs the Azure Migrate appliance to perform agentless discovery/replication of the ESXi inventory into Azure. This is strictly a **migration/inventory tool** — it does not expose the OT plane to the internet; its own management interface sits on the `10.0.0.0/24` plane behind the same perimeter controls. See [`../lab-setup/03-azure-migrate-bridge.md`](../lab-setup/03-azure-migrate-bridge.md).
