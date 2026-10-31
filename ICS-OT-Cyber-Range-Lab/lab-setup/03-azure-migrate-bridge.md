# Phase 3 — Optional Azure Ingestion Bridge

This phase is only needed if you want a hybrid (on-prem + cloud-hosted) variant of the lab, using Azure Migrate for agentless discovery/replication of the ESXi inventory. It is **not required** for the pure on-prem cyber range.

## Create the Azure Migrate project

1. In the Azure Portal, search **Azure Migrate** → **Discover, assess and migrate** → **Create project**.
2. Resource group: e.g. `RG-Azure-OT-Migration-Hybrid`.
3. Project name: e.g. `Azure-OT-Migration-Project`.
4. Geography: your nearest region.
5. Under **Discover**, select **Yes, with VMware vSphere**, generate a project key, and save it securely — it's needed to register the on-prem appliance.

## Stand up the on-prem staging/bridge node

Minimum spec: Windows Server 2022/2025, 8 vCPU, 32 GB RAM, 80 GB disk, NIC on the `10.0.0.0/24` management plane with outbound internet access.

1. Download and extract `AzureMigrateInstaller.zip` to `C:\AzureMigrateInstaller`.
2. From an elevated PowerShell prompt:
   ```powershell
   cd C:\AzureMigrateInstaller
   .\AzureMigrateInstaller.ps1
   ```
3. Select the VMware environment assessment option and your Azure cloud environment (public/government).
4. Once setup completes, browse to `https://localhost:44368` to open the Appliance Configuration Manager and confirm all prerequisite checks (time sync, Azure connectivity, updates) are green.

## Register & discover

1. Paste the project key from step 1 into **Register Azure Migrate**, complete the device-code sign-in flow.
2. Install the VMware VDDK (8.0/9.0) into `C:\Program Files\VMware\VMware Virtual Disk Development Kit`.
3. Add vCenter credentials (`administrator@vsphere.local`) and a discovery source pointing at `10.0.0.200:443`.
4. **Start discovery** and confirm inventory counters increment in the Azure Portal over the following 15–30 minutes.

> Note: the bridge node's own management interface stays on the `10.0.0.0/24` plane behind the same perimeter controls as the rest of the management stack — it is a discovery/replication tool, not a path for OT-plane traffic to reach the internet.
