# `source` field returned but absent from spec

**Type:** `UNDOCUMENTED`
**Endpoints:** `POST /v1/payment_intents`, `GET /v1/payment_intents/{intent}`
**Severity:** High
**GDPR relevance:** Medium — `source` can contain card/bank account data when non-null

---

## What spec3.json says

Search `spec3.json` for `"source"` inside the `payment_intent` schema properties:

```
Result: 0 occurrences in payment_intent schema
```

The `payment_intent` component schema (`#/components/schemas/payment_intent`) does not list `source` as a property.

Stripe does document a `source` object in other contexts (the deprecated Sources API), but it is absent from the payment intent response schema entirely.

---

## What the API returns

Every payment intent response includes:

```json
{
  "id": "pi_XXXXXX",
  "object": "payment_intent",
  "source": null,
  ...
}
```

The field is present on both create and retrieve. In the sandbox the value is `null`. When a legacy Source (card, bank account) is attached to a payment intent, this field contains the full Source object — which includes card details, bank account numbers, and billing information.

---

## The gap

Stripe deprecated the Sources API in favour of PaymentMethods. When they removed Sources from their recommended flows, they also removed `source` from the OpenAPI spec — but the API still emits the field on every payment intent response.

| Dimension | Detail |
|-----------|--------|
| Spec entry | None |
| API emission | Every payment intent, always |
| Value when null | Harmless but undocumented |
| Value when non-null | Contains card/bank data (PII) |
| Stripe rationale | Legacy field from deprecated Sources API |

A client that validates responses against the spec will encounter an unexpected field. A security tool scanning the spec for fields that could expose card data will miss `source` entirely.

---

## GDPR relevance

When non-null, `source` can contain:
- Card last4, brand, expiry
- Bank account last4, routing number
- Billing name and address

All of these are financial PII. Because the field is absent from the spec, it cannot appear in any spec-driven data mapping or privacy audit.

---

## How to test in Postman

**Auth:** Basic Auth — Username: `YOUR_STRIPE_TEST_KEY` — Password: *(empty)*

### Request — Create a payment intent

```
POST https://api.stripe.com/v1/payment_intents
```

Body (`x-www-form-urlencoded`):

| Key | Value |
|-----|-------|
| amount | 2000 |
| currency | eur |
| description | PEEPS source field test |

**In the response:** look for `"source": null`

### Retrieve the payment intent

```
GET https://api.stripe.com/v1/payment_intents/{PI_ID}
```

**In the response:** `source` is still present.

### Verify absence from spec

Open `spec3.json`, search for `"source"` within the payment_intent properties block → **not found**.

---

> **Finding:** `source` is emitted on every payment intent response but has no entry in the spec. The field is a legacy holdover from the deprecated Sources API. When non-null it can contain card and bank account data. A spec-driven security scan would miss this field entirely.
