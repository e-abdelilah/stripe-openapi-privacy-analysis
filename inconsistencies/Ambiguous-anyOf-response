# Ambiguous `anyOf` response — no discriminator on customer GET

**Type:** `SCHEMA_AMBIGUITY / MISSING_REQUIRED`
**Endpoint:** `GET /v1/customers/{customer}`
**Severity:** Medium
**GDPR relevance:** Low — structural, but causes incorrect schema validation

---

## What spec3.json says

The response schema for `GET /v1/customers/{customer}` is:

```json
{
  "anyOf": [
    { "$ref": "#/components/schemas/customer" },
    { "$ref": "#/components/schemas/deleted_customer" }
  ]
}
```

These two schemas are **different objects** with different required fields:

| Schema | `required` fields |
|--------|-------------------|
| `customer` | `created`, `id`, `livemode`, `object` |
| `deleted_customer` | `deleted`, `id`, `object` |

The `deleted_customer` schema defines:
```json
"deleted": {
  "description": "Always true for a deleted object",
  "enum": [true],
  "type": "boolean"
}
```

There is **no `discriminator`** property in the `anyOf` to tell clients or validators which variant to expect.

---

## What the API returns

For an active customer, the API returns a customer object **without** the `deleted` field:

```json
{
  "id": "cus_XXXXXX",
  "object": "customer",
  "created": 1717000000,
  "livemode": false,
  "email": "alice.test@example-peeps.com",
  "name": "Alice Durand"
}
```

The field `deleted` is absent. The response matches the `customer` branch of the `anyOf` — but without a discriminator, a validator must try both branches and cannot automatically determine which one applies.

---

## The gap

Without a `discriminator`, clients cannot:
- Know which schema variant to validate against at parse time
- Determine from the spec alone whether `deleted` will be present
- Distinguish active-customer payloads from deleted-customer payloads without reading `deleted` itself (circular dependency)

A `discriminator` would resolve this:
```json
"discriminator": {
  "propertyName": "deleted",
  "mapping": {
    "true": "#/components/schemas/deleted_customer"
  }
}
```

---

## How to test in Postman

**Auth:** Basic Auth — Username: `YOUR_STRIPE_TEST_KEY` — Password: *(empty)*

### Step 1 — Create a customer

```
POST https://api.stripe.com/v1/customers
```

Body (`x-www-form-urlencoded`):

| Key | Value |
|-----|-------|
| email | alice.test@example-peeps.com |
| name | Alice Durand |
| phone | +33612345678 |
| address[line1] | 10 Rue de Rivoli |
| address[city] | Paris |
| address[postal_code] | 75001 |
| address[country] | FR |

Save `id` → `{CUSTOMER_ID}`

### Step 2 — Retrieve the active customer

```
GET https://api.stripe.com/v1/customers/{CUSTOMER_ID}
```

**Check:** `deleted` is absent. The response matches only the `customer` branch of the `anyOf`.

### Step 3 — Delete the customer

```
DELETE https://api.stripe.com/v1/customers/{CUSTOMER_ID}
```

**Check:** Response now contains `"deleted": true`. This is the `deleted_customer` branch.

### Step 4 — Retrieve the deleted customer

```
GET https://api.stripe.com/v1/customers/{CUSTOMER_ID}
```

**Check:** Response contains `"deleted": true`. Compare the two GET responses — same endpoint, two structurally different objects, no discriminator in the spec.

---

> **Finding:** `GET /v1/customers/{customer}` uses `anyOf [customer, deleted_customer]` with no `discriminator`. A validator cannot determine which variant to use without inspecting the `deleted` field, which only exists in one branch. The OpenAPI 3.0 specification provides `discriminator` exactly for this case; its absence here is a spec preciseness gap.
