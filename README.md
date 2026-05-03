# Stripe API Specification vs Runtime Behavior Inconsistencies
---

# Overview

This repository documents inconsistencies between Stripe’s published API specifications / documentation and the real runtime behavior observed during testing.

The goal is to identify where the documented contract differs from actual responses, hidden requirements, privacy-impacting behavior, deletion semantics, and security assumptions.

This repository is structured :
- each inconsistency has its own dedicated folder + report
- future findings can be added incrementally

---

# Main Objectives

- Compare OpenAPI specs vs real Stripe API behavior
- Detect undocumented fields or hidden workflows
- Analyze privacy / GDPR implications
- Identify deletion vs retention inconsistencies
- Detect contract drift across endpoints
- Improve API governance awareness
- Build reusable DevSecOps validation methodology

---

# Why Stripe as Use Case

Stripe is used because it is:

- globally adopted
- heavily documented
- used in financial ecosystems
- processes sensitive identity/payment data
- integrated by thousands of developers
- suitable for trust-boundary analysis

Stripe serves as a strong real-world case study for API specification reliability.

---

# Stripe High-Level Architecture
Stripe operates as a multi-layer payment and financial infrastructure platform that connects merchants, customers, banks, and payment networks through APIs.
Its architecture is designed around modular services, event-driven workflows, secure tokenization, and compliance controls.
<br>  </br>

<img width="1300" height="700" alt="Image" src="https://github.com/user-attachments/assets/5c1b1b88-07fe-464b-9623-d6067584f676" />

<br>  </br>
# Current Findings Index

| ID | Title | Category | Severity | Status |
|----|------|----------|----------|--------|
| 001 | Hidden Personal Data Collection Behind Capabilities | Privacy / Transparency | Critical | Published |
| 002 | Customer Deletion vs Historical Data Retention | Data Lifecycle | High | Upcoming |
| 003 | Person Deletion vs Identity Exposure | Deletion Boundary | High | Upcoming |
| 004 | Hidden Runtime Required Fields | Contract Drift | Medium | Upcoming |
| 005 | API Version Behavior Drift | Governance | Medium | Upcoming |

---

# Published Finding

## 001 — Hidden Personal Data Collection Behind Capability Status

### Summary

Certain capability states may appear simple in documentation while runtime workflows can require additional onboarding or identity-related data not clearly reflected in the specification.

### Risk Themes

- transparency gap  
- hidden onboarding complexity  
- privacy implications  
- developer misunderstanding  
- contract mismatch  

### Full Report

[Hidden Data Collection Requirements](inconsistencies/hidden-data-collection-requirements/README.md)

