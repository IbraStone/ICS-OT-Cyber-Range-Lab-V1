# Phase 4 — Network Zoning & Hardening

## Segment the subnets

- **IT/management plane:** `10.0.0.0/24` — hypervisor, vCenter, SIEM, migration tooling, outbound NAT/DHCP as needed for patching.
- **ICS/OT plane:** `192.168.0.0/16` (lab instance `192.168.169.0/24`) — locked down, no default internet egress, no outbound NAT.

## Firewall / inspection rules

1. Route all Level 5/4 ↔ Level 3/2 traffic through a stateful/NGFW inspection point with deep packet inspection on ModbusTCP/EtherNet-IP.
2. Explicitly **deny** any packet from Level 5/4 attempting to reach Level 1/0 directly.
3. Mirror cross-level traffic (TAP or SPAN) into the SIEM/IDS node for continuous monitoring.
4. Restrict hypervisor/vCenter management dashboard access to a dedicated, MFA-protected jump host — no direct routing from general user segments.

## Operational verification checklist

- [ ] ISO/media hashes match vendor signatures
- [ ] Management IP hardcoded with zero DHCP lease drift
- [ ] Datastore shows healthy/green status
- [ ] vCenter reachable with active SSO
- [ ] (If hybrid) Migrate appliance registered and reporting inventory
- [ ] OT subnet has zero default internet egress
- [ ] Cross-level traffic visible in SIEM
- [ ] Management dashboards unreachable except via jump host
