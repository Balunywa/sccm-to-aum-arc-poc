SCCM to Azure Update Manager (AUM) with Azure Arc – Comprehensive POC Documentation

Overview

This document provides a comprehensive framework for a Proof of Concept (POC) to evaluate the replacement or side-by-side operation of System Center Configuration Manager (SCCM) with Azure Update Manager (AUM), using Azure Arc as the foundational bridge for managing hybrid Windows Server and SQL Server environments.

The goal of this POC is to:

Provide a structured way to test Azure-native server management

Identify gaps and validate feature parity with SCCM

Help IT teams decide when and how to transition from SCCM to AUM

Objectives

Evaluate SCCM and Azure Update Manager coexistence in hybrid environments

Validate end-to-end patching workflows for Windows Server and SQL Server through Azure Arc

Identify integration challenges between SCCM and AUM

Provide a hands-on lab approach to feature testing and assessment

Strategic Considerations

Lifecycle and Roadmap of SCCM

SCCM is not deprecated, but it is in sustainment mode with limited future innovation.

System Center 2022 and 2025 will continue to support server management.

For forward-looking architectures, Azure Update Manager, Azure Arc, and Azure Machine Configuration are recommended.

Transition Philosophy

Customers should not remove SCCM immediately.

Start with parallel testing and progressively transition workloads to AUM.

Retain SCCM for use cases not yet supported by AUM (e.g., software metering).

SCCM vs Azure Management Services Capability Map

SCCM Capability

Azure Equivalent

Patch Management

Azure Update Manager

Compliance / Configuration Management

Azure Machine Configuration

Asset Inventory (Software)

Azure Change Tracking & Inventory

Asset Inventory (Hardware)

Azure Resource Graph

Application Deployment

Azure VM Apps, Custom Script Extension

OS Deployment / Upgrade

Azure Automation + Scripts

Troubleshooting

Azure Automation, Run Command

Audit and Compliance

Azure Change Tracking, Change Analysis

Endpoint Protection

Microsoft Defender for Cloud

Key Feature Gaps in Azure Management (Compared to SCCM)

Guest Application Updates: Currently unsupported in AUM. Proposal: integrate with Winget or Shared VM App Gallery.

Automated In-place OS Upgrades: Requires custom scripts; no native wizard or rollback.

Patch Rollback: Not natively supported; requires manual intervention.

Sequential Patch Rings: Limited control for ring-based patch rollout; improvement planned.

Distribution Points: No WSUS equivalent. Planned: Azure Guest Patching with local repositories.

POC Environment Setup

1. SCCM Deployment

Deploy SCCM with SQL Express in a test lab

Enable WSUS and create baseline patching collections

2. Azure Arc Onboarding (via Script or GPO)

msiexec /i AzureConnectedMachineAgentSetup.msi /qn ^
  RESOURCE_GROUP="ArcServers" ^
  LOCATION="eastus" ^
  TENANT_ID=<TenantID> ^
  SUBSCRIPTION_ID=<SubID>

3. Enable Azure Update Manager

Assign update schedules

Use tag-based or dynamic scoping

Enable SQL AG-aware patching (preview)

4. Validation Tests and Scenarios

Test ID

Scenario

Expected Result

TC01

Onboard VM with SQL

Server onboarded in Arc with compliance extensions

TC02

Patch via AUM

Updates applied successfully within window

TC03

Conflict test with SCCM

Document duplicate updates or scan issues

TC04

SQL AG-aware patching

Validate successful sequential patching

TC05

Run CM Pivot vs Arc Run Command

Compare execution time and data returned

TC06

Asset comparison

Analyze inventory report variance

Extended Scope Testing

Validate AUM update orchestration across multiple regions

Evaluate private endpoint configurations for AUM traffic

Run SQL BPA via Azure Arc integration

Test AUM in conjunction with Defender for Endpoint

Simulate rollback scenarios and document gaps

Reporting & Monitoring

Use Azure Monitor to build patch compliance dashboards

Compare AUM and SCCM reports for:

Patch state

Last scanned time

Compliance trend

Use Log Analytics queries to correlate events

