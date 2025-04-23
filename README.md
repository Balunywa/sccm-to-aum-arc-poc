# SCCM to Azure Update Manager (AUM) with Azure Arc - Rapid POC

This repository provides a **rapid Proof of Concept (POC)** for customers currently using **System Center Configuration Manager (SCCM)** and evaluating or transitioning to **Azure Update Manager (AUM)**, with **Azure Arc** as the management bridge.

> The goal: Reproduce real-world scenarios, validate coexistence of SCCM and AUM, and highlight what's possible today vs. what's still maturing (e.g., SQL patching, AG-awareness).

---

##  Use Case

Many customers have heavily invested in SCCM over the years, often with deep customizations. Azure Update Manager offers a cloud-native alternative, but the journey from SCCM to AUM is **not a lift-and-shift**. This repo helps you explore:

- AUM coexistence with SCCM in a hybrid state?
- What are the current gaps or friction points?
---

##  What's Inside

This POC covers:

1. **Side-by-side setup** of SCCM and AUM on a domain-joined server
2. **Private Endpoint configuration** to keep all AUM traffic off the public internet
3. **Azure Arc onboarding** for hybrid VMs (Windows + optional Linux)
4. **Custom script deployment** via Arc & Azure Functions
5. **Update orchestration from AUM**, validating sync behavior
6. **SQL patching insights** (what works today, what doesn’t)
7. **POC output templates** (README, diagrams, and reusable JSON & Bash scripts)

---

##  Scenarios Covered

| Scenario | Description |
|----------|-------------|
| SCCM + AUM coexistence | Validate side-by-side patching logic |
| Arc-enabled VM updates | Use Arc to deploy updates via AUM |
| SQL AG-aware patching |  |
| Custom scripts via Arc & Azure Functions | Script-based config/patch automation |
| Private Networking | AUM traffic via Private Endpoints only |

---

## Notes

- This is not a production-ready deployment — it's a **POC** environment.
- You **do not need to remove SCCM** to use AUM or Arc.
- WIN SA + Arc value can still be unlocked without relying on AUM as the sole update engine.

---

## Contributing

Found an edge case? Reproduced a bug? Submit an issue or pull request and let’s improve this together.

---

## Next Steps

- [ ] Complete lab replication steps
- [ ] Validate update compliance across SCCM + AUM
- [ ] Finalize Arc onboarding docs
- [ ] Publish script bundles for automation



---

