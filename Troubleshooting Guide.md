# SCCM + Azure Update Manager (AUM) Troubleshooting Guide

## 1. Overview

This guide walks you through the end-to-end update workflow when using on‑premises Configuration Manager (SCCM) together with Azure Update Manager (AUM) (via Arc), and shows how to troubleshoot the most common failure scenarios.

**Key components**:

- **SCCM/WSUS**: Central on‑prem approval and distribution of Windows updates.
- **Windows Update Agent (WUA)**: Built‑in Windows service that contacts WSUS or Microsoft Update.
- **Arc/Azure Update Manager (AUM)**: Cloud‑based orchestrator that invokes WUA scans and reports results in Azure Portal.
- **Group Policy (GPO)**: Directs WUA to point at your WSUS server instead of MS Update.

---

## 2. Typical Update Flow

**Sync & Approve (SCCM → WSUS)**

- SCCM triggers WSUS sync from Microsoft Update.
- An administrator approves specific KBs for target collections.

**Client Scan (WUA)**

- WUA on each VM reads GPO (`WUServer`, `WUStatusServer`, `UseWUServer=1`).
- WUA contacts WSUS, downloads metadata, then reports available updates.

**SCCM Deployment**

- SCCM deploys approved updates to clients.
- Clients install via CCM agent invoking WUA.

**AUM Assessment**

- Azure Update Manager issues a `StartScan` (WUA).
- WUA responds based on current WSUS metadata and installed state.
- AUM logs and displays results in Azure Portal.

---

## 3. Common Scenarios & Troubleshooting

### Scenario A: GPO not applied, client still talking to Microsoft Update

Symptom
Your VM’s Windows Update Agent (WUA) isn’t pointing at WSUS, so AUM only ever sees “vanilla” Microsoft Update (and often nothing). You’ll spot it when:

powershell
Copy
Edit
Get-ItemProperty `
  -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU' |
  Select-Object UseWUServer, WUServer, WUStatusServer
returns something like:

text
Copy
Edit
UseWUServer    : 0
WUServer       :
WUStatusServer :
What’s Broken

UseWUServer = 0 (or the WSUS server keys are missing) tells WUA to ignore your WSUS/SCCM SUP entirely.

Azure Update Manager (AUM) talks only to WUA, so it never sees your approved WSUS catalog.

Fix & Verify
Force-apply your WSUS GPO

batch
Copy
Edit
gpupdate /force
Restart the Windows Update service

powershell
Copy
Edit
Restart-Service wuauserv
Kick off an immediate scan

batch
Copy
Edit
wuauclt /detectnow        # legacy
# or, on newer OS’s:
UsoClient StartScan
Re-check your registry

powershell
Copy
Edit
(Get-ItemProperty `
  -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU') |
  Select-Object UseWUServer, WUServer, WUStatusServer
Now you should see something like:

text
Copy
Edit
UseWUServer    : 1
WUServer       : http://your‐wsus-server:8530
WUStatusServer : http://your‐wsus-server:8530
Rerun your AUM assessment in the Azure portal. It will now query WUA → WSUS → AUM, and pick up exactly those updates you’ve approved in SCCM/WSUS.
  ```

### Scenario B: WSUS metadata missing or approvals skipped

**Symptom**  
WSUS console shows KB not downloaded or not approved; WUA event log "No updates detected."

**Fault**  
SCCM auto-approval rules not covering this KB classification.

**Fix**  
1. In WSUS console go to **Options → Products and Classifications**  
2. Tick the right categories (e.g. "Updates", "Servicing Stack")  
3. Approve the specific KB for the correct computer group

---

### Scenario C: SCCM installs update before AUM scan

**Symptom**  
AUM portal shows no pending updates, even though WSUS has approvals.

**Fault**  
Once installed, WUA no longer lists them—AUM sees nothing to report.

**Fix**  
- Schedule AUM to scan **before** SCCM deployments  
- Or stagger your scans so AUM runs earlier, then SCCM deploys  
- As a fallback, track both SCCM and AUM reports

---

### Scenario D: Manual Windows Update on VM (outside SCCM)

**Symptom**  
Customer logs into VM, sees available updates via **Settings → Update**, but AUM has no record.

**Fault**  
Manual UI Update triggers Microsoft Update (or WSUS) directly; SCCM isn't aware.

**Fix**  
- Redirect WUA via GPO to WSUS only  
- Remove "Microsoft Update" fallback in client settings  
- Audit with:
  ```powershell
  Get-ItemProperty HKLM:\...\WindowsUpdate\AU |
    Format-List UseWUServer,WUServer,WUStatusServer
  ```

### Scenario E: Arc agent & AUM conflict with SCCM client setting

**Symptom**  
Arc-connected VM shows a green check but Azure Portal no updates, while SCCM report shows many missing patches.

**Fault**  
Both agents calling WUA; if SCCM GPO disables auto-approval of Microsoft Update, AUM sees only what WSUS has.

**Fix**  
- Recognize AUM depends entirely on WSUS approvals  
- Either place AUM-only machines in a separate OU/collection with its own WSUS GPO  
- Or align your SCCM approval policies to include everything AUM needs

---

## 4. Step-By-Step Troubleshoot Template

1. **Verify GPO settings on VM**  
   ```powershell
   Get-ItemProperty HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU |
     Select UseWUServer,WUServer,WUStatusServer
   ```

2. **Refresh policy & WUA**
   ```bat
   gpupdate /force &&
     net stop wuauserv &&
     net start wuauserv &&
     wuauclt /detectnow
   ```

3. **Inspect WUA event log**
   ```bash
   wevtutil qe Microsoft-Windows-WindowsUpdateClient/Operational /c:20 /f:text
   ```

4. **Confirm WSUS catalog**  
   In the WSUS console, ensure your KBs show Downloaded and Approved for the target group.

5. **Run AUM assessment**  
   In the Azure Portal, go to Update Manager → Assess on the target machine.

6. **Compare outputs**  
   Compare the SCCM Software Update compliance reports against the AUM portal results.

7. **Adjust sequence**  
   If SCCM deploys first, consider delaying its deployment window or running a pre-deployment AUM scan to capture visibility.

## 5. Best Practices & Recommendations

### Single approval source
Use SCCM/WSUS as your single system of record for patch approvals. Do not manually sync in AUM without WSUS.

### Scan cadence
Schedule AUM to scan before each SCCM deployment window (e.g. AUM at 3 AM, SCCM at 4 AM).

### Separate OUs
If some VMs are managed only by AUM (no SCCM), place them in a distinct OU with its own WSUS GPO.

### Monitoring
Monitor both SCCM and AUM dashboards—use SCCM for approval/deployment compliance and AUM for cloud-side visibility.

## Outcome

Following this guide, you'll pinpoint exactly which part of the chain (GPO → WUA → WSUS → SCCM → AUM) is misconfigured or out of sync, then correct it so both SCCM and Azure Update Manager show consistent, accurate patch status.


