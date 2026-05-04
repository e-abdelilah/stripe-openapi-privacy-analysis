# Stripe API Specification vs Runtime Behavior Inconsistencies

[![Research Status](https://img.shields.io/badge/status-active-brightgreen)]()
[![Stripe API Version](https://img.shields.io/badge/Stripe%20API-2025--02--24.acacia-blue)]()
[![License](https://img.shields.io/badge/license-MIT-lightgrey)]()

---

## Table of Contents

- [Overview](#overview)
- [Research Objectives](#research-objectives)
- [Why Stripe as a Use Case](#why-stripe-as-a-use-case)
- [Stripe High-Level Architecture](#stripe-high-level-architecture)
- [Research Methodology](#research-methodology)
- [Test Environment & Tooling](#test-environment--tooling)
- [Current Findings Index](#current-findings-index)
- [Published Finding: #001](#published-finding-001)
- [Limitations & Scope](#limitations--scope)
- [Future Work](#future-work)
- [References & Resources](#references--resources)


---

## Overview

This repository documents **inconsistencies** between [Stripe](https://stripe.com)’s published API specifications (OpenAPI) and the **real runtime behavior** observed during controlled testing.

The goal is to identify where the documented contract differs from actual responses, hidden requirements, privacy-impacting behavior, deletion semantics, and security assumptions.  

All findings are listed below in this single document.

---

## Research Objectives

- Compare OpenAPI specs against real Stripe API behavior (sandbox environment)
- Detect undocumented fields, hidden workflows, or unexpected required inputs
- Analyze privacy / GDPR implications of data collection, retention, and deletion
- Identify deletion vs retention inconsistencies across resources (Customer, Person, etc.)
- Detect contract drift across different API versions or endpoint families
- Improve API governance awareness and build a reproducible DevSecOps validation methodology

---

## Why Stripe as a Use Case

Stripe is chosen because it:

- is **globally adopted** by millions of businesses
- provides **extensive** and **public** OpenAPI documentation
- operates in **financial ecosystems** with strict compliance needs
- processes **sensitive identity and payment data**
- is **integrated by thousands of developers** who rely on spec accuracy
- serves as an excellent **real-world trust-boundary** case study

---

## Stripe High-Level Architecture

Stripe operates as a multi-layer payment and financial infrastructure platform that connects merchants, customers, banks, and payment networks through APIs.  
Its architecture is designed around modular services, event-driven workflows, secure tokenization, and compliance controls.

<img width="800" height="400" alt="Stripe Architecture Diagram" src="https://github.com/user-attachments/assets/5c1b1b88-07fe-464b-9623-d6067584f676" />

---

## Research Methodology

The research follows a **documentation-first, then runtime validation** approach:

1. **Spec extraction** – Use Stripe’s official OpenAPI specification.
2. **Test case design** – Derive expected request/response schemas.
3. **Runtime execution** – Execute requests against Stripe sandbox using test data, edge cases, and different API versions.
4. **Diff analysis** – Compare runtime responses against spec expectations.
5. **Severity scoring** – Classify each inconsistency (Critical, High, Medium, Low).

---

## Test Environment & Tooling

| Component | Details |
|-----------|---------|
| **Stripe Environment** | Sandbox (test mode) – no live charges or real PII |
| **API Keys** | Restricted test-mode key with `read_write` scope |
| **Test Payment Methods** | `tok_visa`, `tok_visa_debit`, `tok_mastercard` |
| **Test Identity Data** | Synthetic SSN (`000-00-0000`), US address generator |
| **Tools Used** | Postman, Swagger UI, Swagger Editor, Stripe Sandbox |
| **Standards & Frameworks** | GDPR, OWASP, privacy and security best practices |

> **Note:** No live production data or real customer information is used. All tests comply with Stripe’s sandbox terms of service.

---

## Current Findings Index

| ID | Title | Category | Severity | Status | Report Link |
|----|-------|----------|----------|--------|--------------|
| 001 | Hidden Personal Data Collection Behind Capabilities | Privacy / Transparency | **Critical** | Published | [📄 View](./inconsistencies/hidden-data-collection-requirements/README.md) |
| 002 | Customer Deletion vs Historical Data Retention | Data Lifecycle | **High** | Upcoming | – |
| 003 | Person Deletion vs Identity Exposure | Deletion Boundary | **High** | Upcoming | – |
| 004 | Hidden Runtime Required Fields | Contract Drift | Medium | Upcoming | – |
| 005 | API Version Behavior Drift | Governance | Medium | Upcoming | – |

*Severity definitions:*
- **Critical** – Privacy violation, security misconfiguration, or clear GDPR non‑compliance.
- **High** – Data retained after deletion, misleading documentation that may cause compliance issues.
- **Medium** – Undocumented fields or optional/required mismatches that break client expectations.
- **Low** – Cosmetic differences, rare edge cases, or outdated examples.

---

## Published Finding: #001

### 001 — Hidden Personal Data Collection Behind Capability Status

**Summary**  
Certain capability states may appear simple in documentation while runtime workflows can require additional onboarding or identity-related data not clearly reflected in the specification.

**Risk Themes**  
- transparency gap  
- hidden onboarding complexity  
- privacy implications  
- developer misunderstanding  
- contract mismatch  

**Full Report**  
[Hidden Data Collection Requirements](./inconsistencies/hidden-data-collection-requirements/README.md)

---

## Limitations & Scope

- **Sandbox only** – live mode may differ.
- **Snapshot testing** – findings tied to specific API versions.
- **Not exhaustive** – focuses on high-impact resources (Customers, Accounts, etc.).
- **No fuzzing** – this is specification conformance analysis, not security fuzzing.

---

## Future Work

- OpenAPI runtime validation: detects inconsistencies between Stripe OpenAPI spec and actual API behavior in sandbox tests.
- Contract drift detection: identifies changes in API behavior across versions and flags deviations from expected contracts.
- Privacy-aware OpenAPI enrichment: extends API specs with metadata for sensitive fields (PII tagging, data classification, retention hints).
- GDPR mapping layer: links API fields and behaviors to GDPR principles (deletion validity, data minimization, purpose limitation).
- Sensitive data exposure detection: identifies unexpected PII leakage in runtime responses even if not defined in OpenAPI.
- DevSecOps integration: embeds validation checks into CI pipelines for continuous API compliance and security enforcement.
- API governance monitoring: tracks long-term API consistency, privacy risks, and contract evolution across releases.

---

## References & Resources

- [Stripe OpenAPI official GitHub repository](https://github.com/stripe/openapi/tree/master/openapi)
- [Stripe API Documentation](https://stripe.com/docs/api)  
- [Stripe OpenAPI Specification (GitHub)](https://github.com/stripe/openapi)  
- [OpenAPI Specification (Official)](https://www.openapis.org/)  
- [GDPR (Official EU Law)](https://gdpr-info.eu/)  
- [Stripe Test Cards](https://stripe.com/docs/testing)

---

