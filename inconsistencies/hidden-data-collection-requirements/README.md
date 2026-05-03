# Undocumented Personal Data Collection for Payment Methods

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

### 1. **Hidden Compliance Risk (GDPR Articles 12-14)**
- **Article 12 (Transparency via notice):** Users not informed about data collection
- **Article 13 (Transparency requirement):** No transparency documentation in API
- **Article 6 (Lawful basis):** Consent not explicitly documented
- **Fine risk:** €20M or 4% global annual revenue

### 2. **Special Category Data Not Flagged (GDPR Article 9)**
- Date of birth triggers Article 9 (special category)
- Requires explicit consent + additional safeguards
- Spec shows no warning for developers
- Developers may not implement required consent mechanisms

### 3. **Location Tracking Not Disclosed**
- IP address collection creates privacy concerns
- Used for fraud detection (legitimate interest) but not disclosed
- Combined with name/DOB = precise person identification
- GDPR Article 6(1)(f) requires balancing test documentation

### 4. **Developer Liability Gap**
- Developers integrate using spec (simplified view)
- Get surprising API errors requiring sensitive data
- May comply by collecting data without consent
- Company faces GDPR liability through developer integrations

### 5. **Data Minimization Violation (GDPR Article 5)**
- DOB collects excess information (day + month + year not all needed for age)
- User agent collection seems unnecessary
- Full address collected vs. country-only requirement
- Violates "data minimization" principle


## Classification

- **Severity:** CRITICAL (€20M fine risk)
- **GDPR Articles:** 6, 9, 12, 13, 14 (Multi-article violation)
- **Category:** Transparency & Consent Documentation Gap
- **Affected Capabilities:** alma_payments, klarna, card_payments, transfers, afterpay_clearpay
- **Fine Risk:** €20M or 4% global annual revenue
- **Special Category Data:** YES (DOB)

---

## References

- **GDPR Article 6:** Lawfulness - https://gdpr-info.eu/art-6-gdpr/
- **GDPR Article 9:** Special category data - https://gdpr-info.eu/art-9-gdpr/
- **GDPR Article 12-14:** Data subject transparency - https://gdpr-info.eu/art-12-gdpr/
- **GDPR Article 5:** Data minimization - https://gdpr-info.eu/art-5-gdpr/
