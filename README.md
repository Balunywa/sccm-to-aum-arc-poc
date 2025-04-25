# SCCM to Azure Update Manager (AUM) with Azure Arc – Comprehensive POC Documentation

## Overview

This repository provides a comprehensive framework for a Proof of Concept (POC) to evaluate the replacement or side-by-side operation of System Center Configuration Manager (SCCM) with Azure Update Manager (AUM), using Azure Arc to manage hybrid Windows Server and SQL Server environments.

### Goal

- Provide a structured way to test Azure-native server management
- Identify gaps and validate feature parity with SCCM
- Help IT teams decide when and how to transition from SCCM to AUM

---

## Objectives

- Evaluate SCCM and AUM coexistence in hybrid environments
- Validate end-to-end patching workflows for Windows Server and SQL Server through Azure Arc
- Identify integration and operational challenges
- Provide hands-on guidance for testing and rollout planning

---

## Strategic Considerations

### Lifecycle and Roadmap of SCCM

- SCCM is not deprecated and should continue to be used where it meets current needs
- System Center 2022 and 2025 will remain supported
- Azure-native services including AUM, Arc, and Machine Configuration are the strategic direction

### Transition Philosophy

- Do not remove SCCM immediately
- Run SCCM and AUM in parallel
- Gradually shift supported workloads to AUM

---

## SCCM vs Azure Management Services Capability Map

| SCCM Capability                       | Azure Equivalent                                |
|--------------------------------------|-------------------------------------------------|
| Patch Management                     | Azure Update Manager                            |
| Compliance / Configuration Management| Azure Machine Configuration                     |
| Asset Inventory (Software)           | Azure Change Tracking & Inventory               |
| Asset Inventory (Hardware)           | Azure Resource Graph                            |
| Application Deployment               | Azure VM Apps, Custom Script Extension          |
| OS Deployment / Upgrade              | Azure Automation + Scripts                      |
| Troubleshooting                      | Azure Automation, Run Command                   |
| Audit and Compliance                 | Azure Change Tracking, Change Analysis          |
| Endpoint Protection                  | Microsoft Defender for Cloud                    |

---

## Key Feature Gaps in Azure Management (vs SCCM)

- Guest Application Updates – Not currently supported in AUM (e.g., Chrome, Acrobat)
- Automated In-place OS Upgrades – Requires custom runbooks and pre-check scripts
- Patch Rollback – No native rollback support
- Sequential Patch Rings – Basic support available, full ring control improvements coming
- Distribution Points – No WSUS replacement; Azure Guest Patching enhancements are planned

---

## POC Environment Setup

### 1. SCCM Deployment

- Install SCCM in a lab using SQL Express
- Enable WSUS and configure patching collections

### 2. Azure Arc Onboarding (via GPO or Script)

```powershell
msiexec /i AzureConnectedMachineAgentSetup.msi /qn ^
  RESOURCE_GROUP="ArcServers" ^
  LOCATION="eastus" ^
  TENANT_ID=<TenantID> ^
  SUBSCRIPTION_ID=<SubID>

## Azure Update Manager Configuration

### Step 3: Enable Azure Update Manager

- Assign update schedules using Azure Update Manager (AUM)
- Define patching rings using Azure Tags or dynamic scoping
- Enable SQL AG-aware patching (Preview feature)
- Validate that servers appear in Azure Arc with proper configuration and installed extensions

---

## Validation Tests and Scenarios

| Test ID | Scenario                              | Expected Result                                                                 |
|---------|----------------------------------------|----------------------------------------------------------------------------------|
| TC01    | Onboard VM with SQL                   | Server appears in Azure Arc with compliance and guest configuration extensions  |
| TC02    | Patch via AUM                         | Patches apply within defined maintenance window                                 |
| TC03    | Conflict test with SCCM               | Conflicts identified; either dual scan error or patch rejection                 |
| TC04    | SQL AG-aware patching                 | Patching succeeds without AlwaysOn availability group disruption                |
| TC05    | CM Pivot vs Arc Run Command           | Comparable output returned; execution time differences documented               |
| TC06    | Inventory comparison                  | Validate parity in OS, installed software, and patch data                       |
| TC07    | AUM Patch Failure Scenario            | Simulate a failed patch to observe AUM error reporting and alerting behavior    |
| TC08    | Patch Ring Behavior                   | Patches are applied progressively across tags (e.g., Dev > Staging > Prod)      |
| TC09    | SQL Server Workload Load Test         | Confirm patch does not degrade SQL performance post-install                     |
| TC10    | Azure Monitor Integration             | Patch events and status show up in Monitor dashboards/logs                      |
| TC11    | Custom Scope Filtering with Tags      | AUM targets only tagged resources for patching                                  |
| TC12    | Guest Configuration Compliance        | Machine Configuration policies enforce registry or file state compliance        |
| TC13    | Role-Based Access Control (RBAC)      | Only designated users can manage updates at scope level                         |
| TC14    | Reporting Discrepancy Resolution      | Compare AUM and SCCM compliance reports and explain discrepancies               |
| TC15    | Private Endpoint Isolation Test       | Validate that AUM can patch servers via Private Link only                       |
| TC16    | Run Command Script Deployment         | Successfully deploy a script using Arc Run Command for pre-patch checks         |
| TC17    | Linux VM Patch Support via AUM + Arc  | Apply security patches to an onboarded Linux server and confirm success         |
| TC18    | Update Baseline Application           | Define an update baseline and verify it's applied across a resource group       |
| TC19    | Patch Rollback Simulation (Manual)    | Use VM snapshot to simulate rollback and document guidance                      |
| TC20    | SQL Cluster Visibility in Arc         | Confirm all SQL nodes and health are visible and manageable under Arc           |

---

## Extended Scope Testing

- Validate update orchestration across multiple regions
- Test AUM over Private Endpoint connections
- Run SQL Best Practice Analyzer through Azure Arc integration
- Enable Microsoft Defender for Endpoint for enhanced protection
- Test snapshot-based rollback for patch failures

---

## Reporting & Monitoring

- Use Azure Monitor Workbooks to build patch compliance dashboards
- Track and compare update states across the environment using:
  - Patch compliance state
  - Last update scan time
  - Patch classification breakdowns (e.g., Critical, Security, Feature)
- Use Log Analytics and KQL queries to correlate update, patching, and agent telemetry

### Example KQL Query

```kql
UpdateSummary
| summarize count() by Computer, Classification, UpdateState



