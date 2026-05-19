
# Undocumented Personal Data Collection for Payment Methods

## Table of Contents

1. [Endpoint](#endpoint)
2. [What Spec Says](#what-spec-says)
3. [What Real API Returns](#what-real-api-returns)
4. [The Inconsistency](#the-inconsistency)
5. [Why It Matters](#why-it-matters)
6. [Classification](#classification)
7. [OpenAPI Enrichment Solution](#openapi-enrichment-solution)
   - [7.1 What is OpenAPI Enrichment?](#71-what-is-openapi-enrichment)
   - [7.2 The Five Enrichments](#72-the-five-enrichments)
     - [Enrichment 1: Expand the Description](#enrichment-1-expand-the-description)
     - [Enrichment 2: Add x-activation-requirements](#enrichment-2-add-x-activation-requirements)
     - [Enrichment 3: Add Privacy Labels](#enrichment-3-add-privacy-labels)
     - [Enrichment 4: Add Consent Requirements](#enrichment-4-add-consent-requirements)
   - [7.3 Complete Enriched Example](#73-complete-enriched-example)
   - [7.4 Summary Table](#74-summary-table)
   - [7.5 Affected Capabilities](#75-affected-capabilities)
8. [References](#references)

---

## Endpoint

```
GET /v1/accounts/{account}
POST /v1/account_capabilities (implicit)
```

---

## What Spec Says

**Payment Capabilities Schema** (from `spec3.yaml`)

```yaml
account_capabilities:
  description: ''
  properties:
    alma_payments:
      description: >-
        The status of the Alma capability of the account, or whether the
        account can directly process Alma payments.
      enum:
        - active
        - inactive
        - pending
      type: string
      
    klarna:
      description: >-
        The status of the Klarna capability of the account, or whether the
        account can directly process Klarna payments.
      enum:
        - active
        - inactive
        - pending
      type: string
      
    card_payments:
      description: >-
        The status of the card payments capability of the account.
      enum:
        - active
        - inactive
        - pending
      type: string
```

**Spec Documentation:**
- Payment capabilities presented as simple status fields
- No mention of required data fields
- No transparency warnings
- No GDPR impact assessment

---

## What Real API Returns

### Actual Data Collection Requirements

When `alma_payments` status is `"pending"`, API returns:

```json
{
  "capabilities": {
    "alma_payments": "pending",
    "klarna": "pending",
    "card_payments": "active"
  },
  "requirements": {
    "currently_due": [
      "representative.first_name",
      "representative.last_name",
      "representative.email",
      "representative.dob.day",
      "representative.dob.month",
      "representative.dob.year",
      "representative.phone",
      "representative.address.line1",
      "representative.address.postal_code",
      "representative.address.city",
      "representative.address.country",
      "tos_acceptance.ip",
      "tos_acceptance.date",
      "tos_acceptance.user_agent"
    ]
  }
}
```

### Hidden Data Collections

| Field | Data Type | GDPR Category | Sensitivity |
|-------|-----------|---------------|-------------|
| `representative.first_name` | Personal identifier | Name | HIGH |
| `representative.last_name` | Personal identifier | Name | HIGH |
| `representative.email` | Contact data | Email | MEDIUM |
| `representative.dob.day/month/year` | Age/DOB | Special category | **CRITICAL** |
| `representative.phone` | Contact data | Phone | MEDIUM |
| `representative.address.*` | Location data | Address | MEDIUM |
| `tos_acceptance.ip` | Location data | IP Address | MEDIUM |
| `tos_acceptance.user_agent` | Device fingerprint | Browser/OS data | LOW |

---

## The Inconsistency

| Aspect | Spec Says | API Actually Does |
|--------|-----------|-------------------|
| **Field complexity** | Simple enum (active/inactive/pending) | Requires 14+ data fields |
| **Transparency** | No data requirements mentioned | Extensive PII collection hidden |
| **Legal basis** | Not stated | Implied consent without documentation |
| **Data sensitivity** | No risk warning | Special category data (DOB) collected |
| **Consent requirement** | Silent | Actually requires explicit consent |
| **Purpose declaration** | No purpose given | KYC/fraud prevention purpose unclear |
| **GDPR Article 12** | No privacy notice | Should provide transparency info |

---

## Why It Matters

### 1. Hidden Compliance Risk (GDPR Articles 12-14)
- **Article 12 (Transparency via notice):** Users not informed about data collection
- **Article 13 (Transparency requirement):** No transparency documentation in API
- **Article 6 (Lawful basis):** Consent not explicitly documented
- **Fine risk:** €20M or 4% global annual revenue

### 2. Special Category Data Not Flagged (GDPR Article 9)
- Date of birth triggers Article 9 (special category)
- Requires explicit consent + additional safeguards
- Spec shows no warning for developers
- Developers may not implement required consent mechanisms

### 3. Location Tracking Not Disclosed
- IP address collection creates privacy concerns
- Used for fraud detection (legitimate interest) but not disclosed
- Combined with name/DOB = precise person identification
- GDPR Article 6(1)(f) requires balancing test documentation

### 4. Developer Liability Gap
- Developers integrate using spec (simplified view)
- Get surprising API errors requiring sensitive data
- May comply by collecting data without consent
- Company faces GDPR liability through developer integrations

### 5. Data Minimization Violation (GDPR Article 5)
- DOB collects excess information (day + month + year not all needed for age)
- User agent collection seems unnecessary
- Full address collected vs. country-only requirement
- Violates "data minimization" principle

---

## Classification

- **Severity:** CRITICAL (€20M fine risk)
- **GDPR Articles:** 6, 9, 12, 13, 14 (Multi-article violation)
- **Category:** Transparency & Consent Documentation Gap
- **Affected Capabilities:** alma_payments, klarna, card_payments, transfers, afterpay_clearpay
- **Fine Risk:** €20M or 4% global annual revenue
- **Special Category Data:** YES (DOB)

---

## OpenAPI Enrichment Solution

### 7.1 What is OpenAPI Enrichment?

**OpenAPI Enrichment** means adding missing information to an OpenAPI spec without breaking it. OpenAPI allows custom fields starting with `x-` (called extensions). These are ignored by standard validators but can be read by custom tools.

```
ORIGINAL SPEC (incomplete):              ENRICHED SPEC (complete):
┌─────────────────────────┐              ┌─────────────────────────────────┐
│ alma_payments:          │              │ alma_payments:                  │
│   type: string          │              │   type: string                  │
│   enum:                 │   + ADD      │   enum: [active,inactive,       │
│     - active            │  FIELDS      │          pending]               │
│     - inactive          │              │                                 │
│     - pending           │              │   # ENRICHMENT 1                │
└─────────────────────────┘              │   description: "pending means  │
                                         │     collect user DOB"           │
                                         │                                 │
                                         │   # ENRICHMENT 2                │
                                         │   x-activation-requirements:    │
                                         │     - first_name                │
                                         │     - dob.day                   │
                                         │                                 │
                                         │   # ENRICHMENT 3                │
                                         │   x-pii: true                   │
                                         │   x-gdpr-sensitivity: critical  │
                                         │                                 │
                                         │   # ENRICHMENT 4                │
                                         │   x-observed-vs-spec: {...}     │
                                         │                                 │
                                         │   # ENRICHMENT 5                │
                                         │   x-consent-requirements: {...} │
                                         └─────────────────────────────────┘
```

**The `x-` prefix rule:**

| Field Type | Example | Validator Behavior |
|------------|---------|-------------------|
| Standard | `type: string` | Must follow spec rules |
| Standard | `enum: [...]` | Must follow spec rules |
| Custom | `x-pii: true` | Ignored (never causes error) |
| Custom | `x-gdpr-sensitivity: critical` | Ignored (never causes error) |

**Why this matters:** You can add privacy annotations without breaking the API contract.

---

### 7.2 The Five Enrichments

Each enrichment solves one specific problem discovered during the audit.

---

#### Enrichment 1: Expand the Description

**Problem:** The original description tells developers nothing useful about what `pending` actually means.

**Original:**
```yaml
description: "The status of the Alma capability of the account, or whether the account can directly process Alma payments."
```

**What developers think:** "Pending means Stripe is processing. No user action needed."

**Enriched:**
```yaml
description: >-
  The status of the Alma capability of the account. 'active' means the
  capability is enabled. 'inactive' means it has been disabled. 'pending'
  means activation is in progress and Stripe is awaiting required
  verification data — including personal data about the account
  representative (name, date of birth, address, phone) — before this
  capability can be activated.
```

**What developers understand now:** "Pending means my user must submit personal data including date of birth."

**Where it goes:** Replaces the existing `description` field.

---

#### Enrichment 2: Add x-activation-requirements

**Problem:** The spec doesn't document what data Stripe actually requests when a capability is pending.

**Original:** Nothing.

**Enriched:**
```yaml
x-activation-requirements:
  description: Fields Stripe requires before this capability becomes active
  fields:
    - path: representative.first_name
      pii: true
      gdpr_category: personal_identifier
    - path: representative.last_name
      pii: true
      gdpr_category: personal_identifier
    - path: representative.dob.day
      pii: true
      gdpr_category: special_category
      gdpr_article: "9"
    - path: representative.dob.month
      pii: true
      gdpr_category: special_category
      gdpr_article: "9"
    - path: representative.dob.year
      pii: true
      gdpr_category: special_category
      gdpr_article: "9"
    - path: representative.email
      pii: true
      gdpr_category: contact_data
    - path: representative.phone
      pii: true
      gdpr_category: contact_data
    - path: representative.address.line1
      pii: true
      gdpr_category: location_data
    - path: representative.address.postal_code
      pii: true
      gdpr_category: location_data
    - path: representative.address.city
      pii: true
      gdpr_category: location_data
    - path: representative.address.country
      pii: false
      gdpr_category: location_data
    - path: tos_acceptance.ip
      pii: true
      gdpr_category: location_data
    - path: tos_acceptance.user_agent
      pii: true
      gdpr_category: device_fingerprint
    - path: tos_acceptance.date
      pii: false
      gdpr_category: audit_data
```

**What this enables:** Developers can see exactly what 14 fields they need to collect. Tools can auto-generate data collection forms.

**Where it goes:** New `x-activation-requirements` field under the capability.

---

#### Enrichment 3: Add Privacy Labels

**Problem:** The spec gives no warning that this capability triggers personal data collection, including special-category data under GDPR Article 9.

**Original:** Nothing.

**Enriched:**
```yaml
x-pii: true
x-gdpr-sensitivity: critical
x-gdpr-articles:
  - article: "6"
    reason: Lawful basis required for processing representative personal data
  - article: "9"
    reason: Date of birth is special category data requiring explicit consent
  - article: "12"
    reason: Transparency obligation — data subjects must be informed before collection
  - article: "13"
    reason: Developer must provide privacy notice when collecting this data
  - article: "5"
    reason: Data minimization — verify which DOB fields are truly necessary
```

**What the labels mean:**

| Label | Value | Meaning |
|-------|-------|---------|
| `x-pii` | `true` | This field triggers collection of Personally Identifiable Information |
| `x-gdpr-sensitivity` | `critical` | Includes Article 9 special category data (DOB) |
| `x-gdpr-sensitivity` | `high` | Direct identifiers (name, email) |
| `x-gdpr-sensitivity` | `medium` | Indirect identifiers (IP address) |
| `x-gdpr-sensitivity` | `low` | Technical data (timestamps) |

**What this enables:** Developers see a warning label before using the field. Legal teams know which GDPR articles are triggered.

**Where it goes:** New fields directly under the capability.

```

**What this enables:** Every discrepancy is documented with evidence. You can programmatically extract all findings from the spec. Your internship report has verifiable data.

**Where it goes:** New `x-observed-vs-spec` field under the capability.

---

#### Enrichment 4: Add Consent Requirements

**Problem:** The spec gives no guidance on what developers must legally implement before using this API.

**Original:** Nothing.

**Enriched:**
```yaml
x-consent-requirements:
  required: true
  special-category-data: true
  legal-basis-options:
    - basis: explicit-consent
      article: "9(2)(a)"
      implementation: >-
        Explicit consent must be obtained from the representative before
        collecting DOB. A pre-ticked box or implied consent is not
        sufficient under Article 9.
    - basis: legal-obligation
      article: "6(1)(c)"
      implementation: >-
        If KYC is required by AML regulation, this may serve as legal basis
        for DOB collection but must still be documented.
  privacy-notice-required: true
  data-retention-policy-required: true
  dpa-with-stripe-required: true
  note: >-
    Developers must have a Data Processing Agreement with Stripe before
    routing special-category data through their platform.
```

**What this enables:** Developers know exactly what legal requirements they must satisfy. Legal teams have a clear checklist.

**Where it goes:** New `x-consent-requirements` field under the capability.

---

### 7.3 Complete Enriched Example

Here is the fully enriched `alma_payments` field with all 5 enrichments:

```yaml
alma_payments:
  # ORIGINAL FIELDS (unchanged - API contract)
  type: string
  enum:
    - active
    - inactive
    - pending
  
  # ENRICHMENT 1: Expanded description
  description: >-
    The status of the Alma capability of the account. 'active' means the
    capability is enabled. 'inactive' means it has been disabled. 'pending'
    means activation is in progress and Stripe is awaiting required
    verification data — including personal data about the account
    representative (name, date of birth, address, phone) — before this
    capability can be activated. See x-activation-requirements for the
    full list of fields Stripe will request.
  
  # ENRICHMENT 2: Privacy labels
  x-pii: true
  x-gdpr-sensitivity: critical
  x-gdpr-articles:
    - article: "6"
      reason: Lawful basis required for processing representative personal data
    - article: "9"
      reason: Date of birth is special category data requiring explicit consent
    - article: "12"
      reason: Transparency obligation — data subjects must be informed before collection
    - article: "13"
      reason: Developer must provide privacy notice when collecting this data
    - article: "5"
      reason: Data minimization — verify which DOB fields are truly necessary
  
  # ENRICHMENT 3: Required data fields
  x-activation-requirements:
    description: Fields Stripe requires before this capability becomes active
    fields:
      - path: representative.first_name
        pii: true
        gdpr_category: personal_identifier
      - path: representative.last_name
        pii: true
        gdpr_category: personal_identifier
      - path: representative.dob.day
        pii: true
        gdpr_category: special_category
        gdpr_article: "9"
      - path: representative.dob.month
        pii: true
        gdpr_category: special_category
        gdpr_article: "9"
      - path: representative.dob.year
        pii: true
        gdpr_category: special_category
        gdpr_article: "9"
      - path: representative.email
        pii: true
        gdpr_category: contact_data
      - path: representative.phone
        pii: true
        gdpr_category: contact_data
      - path: representative.address.line1
        pii: true
        gdpr_category: location_data
      - path: representative.address.postal_code
        pii: true
        gdpr_category: location_data
      - path: representative.address.city
        pii: true
        gdpr_category: location_data
      - path: representative.address.country
        pii: false
        gdpr_category: location_data
      - path: tos_acceptance.ip
        pii: true
        gdpr_category: location_data
      - path: tos_acceptance.user_agent
        pii: true
        gdpr_category: device_fingerprint
      - path: tos_acceptance.date
        pii: false
        gdpr_category: audit_data
    
  # ENRICHMENT 4: Legal requirements
  x-consent-requirements:
    required: true
    special-category-data: true
    legal-basis-options:
      - basis: explicit-consent
        article: "9(2)(a)"
        implementation: >-
          Explicit consent must be obtained from the representative before
          collecting DOB. A pre-ticked box or implied consent is not
          sufficient under Article 9.
      - basis: legal-obligation
        article: "6(1)(c)"
        implementation: >-
          If KYC is required by AML regulation, this may serve as legal basis
          for DOB collection but must still be documented.
    privacy-notice-required: true
    data-retention-policy-required: true
    dpa-with-stripe-required: true
    note: >-
      Developers must have a Data Processing Agreement with Stripe before
      routing special-category data through their platform.
```

---

### 7.4 Summary Table

| # | Enrichment | Field Name | Question It Answers | Type of Info |
|---|------------|------------|---------------------|--------------|
| 1 | Expand description | `description` | What does this field actually mean? | Human-readable text |
| 2 | Activation requirements | `x-activation-requirements` | What exact data does Stripe request? | Structured data list |
| 3 | Privacy labels | `x-pii`, `x-gdpr-sensitivity`, `x-gdpr-articles` | Is this safe to use without legal review? | Warning labels |
| 4 | Consent requirements | `x-consent-requirements` | What must I do to use this legally? | Compliance checklist |

---

### 7.5 Affected Capabilities

These enrichments apply to any capability where `pending` status triggers data collection:

- `alma_payments`
- `klarna`
- `card_payments`
- `transfers`
- `afterpay_clearpay`

The same pattern applies to all payment method capabilities that require KYC verification before activation.

---

## References

- **GDPR Article 5:** Data minimization - https://gdpr-info.eu/art-5-gdpr/
- **GDPR Article 6:** Lawfulness - https://gdpr-info.eu/art-6-gdpr/
- **GDPR Article 9:** Special category data - https://gdpr-info.eu/art-9-gdpr/
- **GDPR Article 12-14:** Data subject transparency - https://gdpr-info.eu/art-12-gdpr/
- **OpenAPI Custom Extensions:** https://swagger.io/docs/specification/openapi-extensions/
```
