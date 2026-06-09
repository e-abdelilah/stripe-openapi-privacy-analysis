# `shared_payment_granted_token` — undocumented credential field

**Type:** `UNDOCUMENTED + PII (HIGH/CREDENTIALS)`
**Endpoints:** `POST /v1/payment_intents`, `GET /v1/payment_intents/{intent}`
**Severity:** Critical
**GDPR relevance:** High — credential-class field with no documented lifecycle or purpose

---

## What spec3.json says

Search the entire `spec3.json` for `shared_payment_granted_token`:

```
Result: 0 occurrences
```

The field has **no entry anywhere** in the spec — not in the payment_intent schema properties, not in any component schema, not in any request or response definition.

The payment_intent response schema (`#/components/schemas/payment_intent`) documents many fields but `shared_payment_granted_token` is not among them.

---

## What the API returns

Every `POST /v1/payment_intents` response includes:

```json
{
  "id": "pi_XXXXXX",
  "object": "payment_intent",
  "shared_payment_granted_token": null,
  ...
}
```

The field is also present on `GET /v1/payment_intents/{intent}`.

In the sandbox the value is always `null`. In production contexts it may become non-null — but because the field is entirely absent from the spec, its type, lifecycle, and sensitivity level are completely unknown to any client.

---

## The gap

| Dimension | Detail |
|-----------|--------|
| Spec entry | None |
| API emission | Every payment intent, always |
| Field name classification | `_token` suffix → HIGH/CREDENTIALS |
| Type when non-null | Unknown (not documented) |
| When it becomes non-null | Unknown |
| Whether it is sensitive | Unknown — must be inferred from name |

A client developer reading only the spec has no basis to:
- Know this field exists
- Know whether to treat it as sensitive
- Know whether to log, store, or display it
- Know what to do if it becomes non-null

---

## GDPR relevance

If `shared_payment_granted_token` becomes non-null, it likely constitutes a **bearer credential** — a piece of data that grants access to a resource or action. Under GDPR Article 5(1)(f), personal data must be processed with "appropriate security". A credential field that is invisible in the spec cannot be included in a data mapping, DPIA, or access-control review. Its presence in API logs or frontend storage would not be flagged by any spec-driven tooling.

---

## How to test in Postman

**Auth:** Basic Auth — Username: `YOUR_STRIPE_TEST_KEY` — Password: *(empty)*

### Step 1 — Create a payment intent

```
POST https://api.stripe.com/v1/payment_intents
```

Body (`x-www-form-urlencoded`):

| Key | Value |
|-----|-------|
| amount | 2000 |
| currency | eur |
| description | PEEPS privacy test |

Save `id` → `{PI_ID}`

**In the response:** look for `"shared_payment_granted_token": null`

### Step 2 — Retrieve the payment intent

```
GET https://api.stripe.com/v1/payment_intents/{PI_ID}
```

**In the response:** `shared_payment_granted_token` is still present on GET.

### Step 3 — Verify it is absent from the spec

Open `spec3.json` and search (Ctrl+F) for `shared_payment_granted_token` → **zero results**.

---

> **Finding:** The API emits `shared_payment_granted_token` on every payment intent. The spec has no record of this field. Because its name ends in `_token`, it is classified as HIGH/CREDENTIALS. Clients cannot know what this field represents, whether it is sensitive, or what to do when its value is non-null — the spec gives them nothing.
