# Phase 2 — vCenter Server Appliance (VCSA) & Storage

## Deploy VCSA

Stage 1 (OVA deployment):

| Setting | Value |
|---|---|
| Target host | `10.0.0.150` (root creds) |
| VM name | `OTvCenterServer` |
| Deployment size | Tiny |
| Datastore | `OTCYBERDATAStorage`, thin disk |
| Network | Management port group |
| IP assignment | Static — `10.0.0.200` / `255.255.255.0` / GW `10.0.0.1` |

Stage 2 (configuration):

- Time sync: with ESXi host.
- SSH: enabled (admin flexibility — disable once the lab build is finalized if not needed).
- New SSO domain: `vsphere.local`, strong SSO password.

## Create datacenter & attach host

1. Browse to `https://10.0.0.200` → **Launch vSphere Client (HTML5)**.
2. Log in as `administrator@vsphere.local`.
3. Create a new datacenter (e.g. `OT-Hybrid-Datacenter`).
4. **Add Host** → `10.0.0.150` with root credentials; accept the thumbprint; leave Lockdown Mode disabled for initial staging (re-enable per your hardening policy once the build is stable).

## Storage fabric

From the ESXi Host Client (`https://10.0.0.150`):

1. **Storage → New Datastore → Create new VMFS datastore**.
2. Name: `OTCYBERDATAStorage`.
3. File system: VMFS 6, full disk capacity.
4. Confirm the volume shows green/healthy in the recent-tasks pane before provisioning guest VMs.
