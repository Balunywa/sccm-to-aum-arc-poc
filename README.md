# SCCM to Azure Update Manager (AUM) with Azure Arc - Rapid POC

This repository provides a **rapid Proof of Concept (POC)** for customers currently using **System Center Configuration Manager (SCCM)** and evaluating or transitioning to **Azure Update Manager (AUM)**, with **Azure Arc** as the management bridge.

This POC is designed to help enterprise IT teams assess the feasibility of operating SCCM and AUM side-by-side. It provides guidance for validating coexistence scenarios, identifying integration challenges, and evaluating the business value of using cloud-native tools like AUM while leveraging existing SCCM investments.

---

## Use Case

Many enterprise environments rely on SCCM for endpoint management and software updates. As organizations adopt cloud-native services, AUM offers a scalable alternative for update management through Azure Arc-enabled servers.

This repository enables customers to:

- Evaluate SCCM and AUM coexistence in a hybrid state
- Identify friction points in dual-update management
- Understand update control delegation between SCCM and AUM

---

## POC Flow

This POC is intended for a small-scale validation in a lab or development environment.

1. **Install SCCM using the "Typical" Setup**  
   Use the Configuration Manager setup wizard with default settings to quickly deploy a primary site using SQL Express.

2. **Arc-enable Domain-Joined VMs**  
   Use the Azure Arc agent to onboard a small number of Windows Server VMs to Azure.

3. **Enable Azure Update Manager (AUM)**  
   Assign the Arc-enabled servers to an update deployment schedule from the Azure portal.

4. **Test Update Compliance and Behavior**  
   Observe how SCCM and AUM interact with update management, including whether both tools try to enforce compliance, and if scheduling conflicts arise.

5. **Document Observations**  
   Track key behavioral differences, management conflicts, and areas where clarification or control configuration is needed.

---

## Key Integration Challenges to Observe

- Conflicts between SCCM WSUS scan cycles and AUM patch orchestration
- Discrepancies in update compliance reporting
- Coordination issues when the same VM is managed by both ConfigMgr and Azure Arc
- Patch schedule collisions between SCCM and AUM
- Assignment overlaps in Update Deployment Rings and SCCM Collections

---

## What's Inside

This repository includes:

- Side-by-side setup of SCCM and AUM on domain-joined Windows servers
- Private Endpoint configuration for securing AUM connectivity
- Azure Arc onboarding instructions for hybrid VM management
- Sample scripts for testing update orchestration through Arc
- Validation logic to track patch compliance and dual-management behavior
- POC templates (README, visual aids, JSON policies, automation scripts)

---

## Scenarios Covered

| Scenario                        | Description                                             |
|--------------------------------|---------------------------------------------------------|
| SCCM + AUM coexistence         | Evaluate dual-management patching logic                 |
| Arc-enabled VM updates         | Use Azure Arc to deliver updates via AUM                |
| SQL AG-aware patching          | Identify current limitations for clustered SQL          |
| Script-based patch automation  | Automate config changes and validation checks           |
| Private networking for AUM     | Restrict AUM traffic to private endpoints only          |

---

## Extended Scope Considerations

This POC specifically aims to evaluate patching Windows servers and SQL Server instances, with the following goals:

- Evaluate whether SCCM can be replaced or operate side-by-side with AUM
- Determine how SQL patching behaves under Arc-enabled management
- Inventory reporting of SQL Server environments
- Assess Azure Arc's ability to integrate SQL BPA (Best Practices Analyzer) with Failover Clusters
- Explore single-pane-of-glass management for SQL workloads
- Determine if patching via Arc is viable, while disabling it in SCCM
- Evaluate current SCCM capabilities:
  - CM Pivot queries
  - Patch scheduling
  - Software repository and app catalog
  - Device inventory

### Key Inputs Required

- Number of servers in scope
- Versions of Windows Server and SQL Server in scope
- Whether servers have internet access or need private-only connectivity
- Intention to continue using internal WSUS for update source (if preferred)
- Target to complete the POC setup by May to align with monthly patch cycle

---

## Notes

- This POC is not designed for production environments.
- SCCM does not need to be removed to onboard to Azure Arc or AUM.
- Windows Server Software Assurance (WIN SA) and Azure Arc benefits can be realized independently of AUM adoption.

---

## Contributing

We welcome feedback, issue reports, and pull requests. If you identify edge cases or conflicts in the dual-management workflow, please contribute improvements.

---

