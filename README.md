# SCCM to Azure Update Manager (AUM) with Azure Arc: Comprehensive Proof of Concept (POC)

## Executive Summary

This document provides a comprehensive framework for evaluating the migration and coexistence of System Center Configuration Manager (SCCM) with Azure Update Manager (AUM) in hybrid Windows Server and SQL Server environments. The framework is designed to help organizations assess, plan, and execute a controlled transition from traditional on-premises management to cloud-native solutions.

## Objectives

The primary goals of this Proof of Concept (POC) are to:

- Validate Azure-native server management capabilities through hands-on testing
- Assess feature parity between SCCM and Azure management services across key operational scenarios
- Develop a strategic transition roadmap aligned with business requirements
- Minimize operational disruption during migration through careful planning and testing
- Evaluate cost implications and operational efficiency gains
- Document specific technical requirements and dependencies

## Strategic Overview

### Current Landscape

- **SCCM Status**: Supported and viable for existing infrastructure
- **Azure Direction**: Strategic shift towards cloud-native management services with continuous feature improvements
- **Transition Approach**: Parallel operation and gradual workload migration with risk mitigation
- **Hybrid Considerations**: Support for hybrid environments using with connnectivity through a proxy, S2S VPN, Express Route and Arc enabled private link

### Capability Comparison

| SCCM Capability                       | Azure Equivalent                                | Migration Complexity | Key Considerations |
|---------------------------------------|------------------------------------------------|---------------------|-------------------|
| Patch Management                     | Azure Update Manager                            | Low                 | Native integration with Azure security tools |
| Compliance Management                | Azure Machine Configuration                     | Medium              | Policy-driven approach with enhanced reporting |
| Software Inventory                   | Azure Change Tracking & Inventory               | Medium              | Real-time tracking capabilities |
| Hardware Inventory                   | Azure Resource Graph                            | Low                 | Graph-based querying for fleet insights |
| Application Deployment               | Azure VM Apps, Custom Script Extension          | High                | May require workflow redesign |
| OS Deployment                        | Azure Automation + Scripts                      | High                | Custom automation required |
| Endpoint Protection                  | Microsoft Defender for Cloud                    | Low                 | Enhanced threat protection |

## Key Challenges and Limitations

### Azure Management Service Constraints

- Limited guest application update support requiring supplementary solutions
- No native patch rollback mechanism; requires snapshot-based approach
- Partial sequential patch ring implementation with planned improvements
- Custom scripting required for complex deployment scenarios
- Network considerations for hybrid connectivity
- Compliance reporting differences requiring mapping

## Proof of Concept Environment

### Infrastructure Requirements

- SCCM Lab Environment
  - SQL Dev Edition backend for testing
  - Configured WSUS infrastructure
  - Existing patch management collections
  - Test systems representing production workloads

- Azure Prerequisites
  - Azure Arc enabled servers
  - Subscription with appropriate permissions
  - Azure Update Manager configured
  - Network connectivity validation
  - Required RBAC roles assigned

### Onboarding Strategy

```powershell
# Azure Connected Machine Agent Deployment
msiexec /i AzureConnectedMachineAgentSetup.msi /qn ^
  RESOURCE_GROUP="ArcServers" ^
  LOCATION="eastus" ^
  TENANT_ID=<TenantID> ^
  SUBSCRIPTION_ID=<SubID>
```

## Validation Tests and Scenarios

| Test ID | Scenario | Description | Steps | Expected Result | Actual Result |
|---------|----------|-------------|-------|----------------|---------------|
| TC01 | Onboard SQL VM | Validate Azure Arc onboarding for a SQL Server VM | 1. Prepare SQL VM<br>2. Install Azure Connected Machine Agent<br>3. Register with Azure Arc<br>4. Verify extensions | Server appears in Azure Arc with compliance and guest configuration extensions | |
| TC02 | Patch via Update Manager | Verify patches apply during maintenance window | 1. Configure maintenance window<br>2. Define patch criteria<br>3. Trigger patch deployment<br>4. Monitor progress | Patches apply within defined maintenance window | |
| TC03 | SCCM and AUM Conflict | Test potential conflicts between patch management tools | 1. Enable both SCCM and AUM scanning<br>2. Attempt simultaneous patch deployment<br>3. Document behavior | Conflicts identified; either dual scan error or patch rejection | |
| TC04 | SQL AlwaysOn Group Patch | Ensure patching does not disrupt AlwaysOn availability | 1. Validate initial AG health<br>2. Initiate patch deployment<br>3. Monitor failover behavior<br>4. Check database availability | Patching succeeds without AlwaysOn availability group disruption | |
| TC05 | CM Pivot vs Arc Run Command | Compare functionality of management tools | 1. Execute identical commands in both tools<br>2. Measure response times<br>3. Compare output formats | Comparable output returned; execution time differences documented | |
| TC06 | Inventory Comparison | Validate consistency of system information | 1. Generate SCCM inventory<br>2. Pull Azure Arc inventory<br>3. Compare data points | Validate parity in OS, installed software, and patch data | |
| TC07 | Failed Patch Handling | Test error reporting and alerting | 1. Trigger intentional patch failure<br>2. Monitor alert generation<br>3. Review error reporting | Simulated failure triggers appropriate alerts and logs error details | |
| TC08 | Staged Deployment Behavior | Verify progressive patch application | 1. Define update rings<br>2. Configure schedules<br>3. Deploy test update<br>4. Monitor progression | Patches are applied progressively across tags (e.g., Dev > Staging > Prod) | |
| TC09 | SQL Load Validation Post-Patch | Assess performance impact of patching | 1. Establish baseline performance metrics<br>2. Apply patches<br>3. Monitor key performance indicators | No significant degradation in SQL performance post-patching | |
| TC10 | Integration with Azure Monitor | Validate monitoring and logging | 1. Configure Azure Monitor<br>2. Deploy patches<br>3. Review dashboard data<br>4. Verify alert flow | Patch events and status show up in Monitor dashboards/logs | |
| TC11 | Scope Filtering with Tags | Test resource targeting | 1. Configure tag-based targeting<br>2. Apply test tags<br>3. Deploy updates<br>4. Verify scope adherence | Only tagged resources receive updates | |
| TC12 | Guest Configuration Compliance | Enforce configuration policies | 1. Define configuration policies<br>2. Apply to test systems<br>3. Attempt policy violations<br>4. Monitor enforcement | Registry and file settings enforced as per defined baselines | |
| TC13 | RBAC Enforcement | Validate access control | 1. Configure RBAC roles<br>2. Test permissions<br>3. Attempt unauthorized actions<br>4. Review audit logs | Update operations limited to users with assigned roles | |
| TC14 | Report Alignment Check | Compare reporting across tools | 1. Generate reports from SCCM and AUM<br>2. Compare data points<br>3. Identify discrepancies | Compliance data reconciled across AUM and SCCM | |
| TC15 | Private Network Isolation Test | Validate secure update delivery | 1. Configure Private Link<br>2. Verify connectivity<br>3. Deploy updates<br>4. Monitor traffic flow | Patch success over Private Endpoint with no public egress | |
| TC16 | Script Deployment via Run Command | Test pre-patch scripting | 1. Create test script<br>2. Deploy via Azure Arc Run Command<br>3. Verify execution<br>4. Check results | Pre-check script executes successfully | |
| TC17 | Linux Patch Workflow | Validate Linux server update process | 1. Onboard Linux servers<br>2. Configure patch policies<br>3. Deploy updates<br>4. Verify reboot handling | Linux servers patched and rebooted per policy | |
| TC18 | Baseline Application Validation | Test update baseline consistency | 1. Define update baseline<br>2. Apply to resource group<br>3. Verify compliance<br>4. Check for consistent application | Patch baselines applied consistently to resource groups | |
| TC19 | Manual Patch Rollback Simulation | Evaluate recovery mechanisms | 1. Create system snapshot<br>2. Apply test update<br>3. Initiate rollback<br>4. Verify system state | Snapshot or restore validated for rollback | |
| TC20 | SQL Cluster Visibility in Azure Arc | Confirm cluster management capabilities | 1. Register SQL cluster<br>2. View health state<br>3. Manage updates<br>4. Monitor nodes | All nodes and cluster health visible and manageable | |


## Recommended Next Steps

1. Validate POC results against organizational requirements and success criteria
2. Develop detailed migration playbook with rollback procedures
3. Create comprehensive training materials for IT operations teams
4. Plan phased rollout with minimal risk and defined success metrics
5. Document lessons learned and best practices
6. Establish ongoing monitoring and optimization processes

## Appendices

- Detailed test case specifications with expected outcomes
- Configuration reference architectures for various scenarios
- Risk mitigation strategies and contingency plans
- Network architecture considerations
- Security and compliance documentation
- Training and operational readiness checklists

**Version**: 1.0  


