# Stripe Payment Intents API: Specification vs Runtime Inconsistencies

## What Is a Payment Intent?

A **Stripe Payment Intent** is an object used by Stripe to manage and track a payment from the moment it is created until it is completed or fails.

In simple terms:

> It is a "payment session" that keeps track of a customer's payment and ensures it is processed correctly and securely.

It handles:

- Starting a payment
- Checking if extra authentication is needed (like 3D Secure)
- Processing the transaction
- Confirming success or failure

---

## Endpoint

```http
GET https://api.stripe.com/v1/payment_intents
POST https://api.stripe.com/v1/payment_intents
```

**OpenAPI Spec:** `spec3.yaml` (version 2026-03-25.dahlia)  
**Schema Location:** Lines 29110–29637  
**Endpoint Location:** Lines 107194–107330

---

## API Response Used for Comparison

```json
{
    "id": "pi_*******",
    "object": "payment_intent",
    "amount": 3000,
    "amount_capturable": 0,
    "amount_details": {
        "tip": {}
    },
    "amount_received": 0,
    "application": null,
    "application_fee_amount": null,
    "automatic_payment_methods": null,
    "canceled_at": null,
    "cancellation_reason": null,
    "capture_method": "automatic_async",
    "client_secret": "pi_*******",
    "confirmation_method": "automatic",
    "created": 1778458680,
    "currency": "eur",
    "customer": null,
    "customer_account": null,
    "description": null,
    "excluded_payment_method_types": null,
    "last_payment_error": null,
    "latest_charge": null,
    "livemode": false,
    "managed_payments": {
        "enabled": false
    },
    "metadata": {
        "customer_name": "Abdo",
        "order_id": "ORDER-123"
    },
    "next_action": null,
    "on_behalf_of": null,
    "payment_method": null,
    "payment_method_configuration_details": null,
    "payment_method_options": {
        "card": {
            "installments": null,
            "mandate_options": null,
            "network": null,
            "request_three_d_secure": "automatic"
        }
    },
    "payment_method_types": [
        "card"
    ],
    "processing": null,
    "receipt_email": "abdo@gmail.com",
    "review": null,
    "setup_future_usage": null,
    "shared_payment_granted_token": null,
    "shipping": null,
    "source": null,
    "statement_descriptor": null,
    "statement_descriptor_suffix": null,
    "status": "requires_payment_method",
    "transfer_data": null,
    "transfer_group": null
}
```

---

## Inconsistencies Found

Three fields returned by the API are completely absent from the `payment_intent` schema in the OpenAPI spec (lines 29135–29609). These appear in every response regardless of what is sent in the request body.

| # | Field | In Spec? | In API Response? | Type |
|---|-------|----------|------------------|------|
| 1 | `managed_payments` |  No |  Yes | Undocumented field |
| 2 | `shared_payment_granted_token` |  No |  Yes | Undocumented field |
| 3 | `source` |  No |  Yes | Undocumented legacy field |

---

### 1. `managed_payments` — Undocumented Field

**API returns:**
```json
"managed_payments": {
    "enabled": false
}
```

**Spec says:** Not documented. Searched the entire `payment_intent` schema (lines 29135–29609) and the full spec file — zero results for `managed_payments`.

**What it appears to be:** A configuration object indicating whether Stripe-managed payment processing is enabled for this PaymentIntent. Returned in every response with no documentation of its purpose, behavior, or data handling.

---

### 2. `shared_payment_granted_token` — Undocumented Field

**API returns:**
```json
"shared_payment_granted_token": null
```

**Spec says:** Not documented. Searched the entire spec file — zero results for `shared_payment_granted_token`.

**What it appears to be:** A security/delegation token for shared payment flows. Currently `null` in all observed responses. The field name suggests it could grant access to payment operations, but there is no documentation of its scope, lifetime, or revocation mechanism.

---

### 3. `source` — Undocumented Legacy Field

**API returns:**
```json
"source": null
```

**Spec says:** Not documented in the `payment_intent` schema. The spec only documents `payment_method` as the field for referencing payment instruments:

```yaml
# Line 29454
payment_method:
  anyOf:
    - maxLength: 5000
      type: string
    - $ref: '#/components/schemas/payment_method'
  description: ID of the payment method used in this PaymentIntent.
  nullable: true
```

**What it appears to be:** A legacy field from the older Sources API that Stripe still returns for backwards compatibility, even though the spec has migrated to `payment_method`. The `source` field exists in other schemas (like `charge`), but not in `payment_intent`.

---

## Reproduction

### Request (Postman)

```http
POST https://api.stripe.com/v1/payment_intents
Authorization: Bearer sk_test_YOUR_TEST_KEY
Content-Type: application/x-www-form-urlencoded

amount=3000
currency=eur
payment_method_types[]=card
receipt_email=abdo@gmail.com
metadata[order_id]=ORDER-123
metadata[customer_name]=Abdo
```

### Verify Undocumented Fields

```bash
# List all keys in the response
curl -s https://api.stripe.com/v1/payment_intents \
  -H "Authorization: Bearer sk_test_..." \
  | jq '.data[0] | keys'

# Check for undocumented fields
curl -s https://api.stripe.com/v1/payment_intents \
  -H "Authorization: Bearer sk_test_..." \
  | jq '.data[0] | {managed_payments, shared_payment_granted_token, source}'
```

### Search Spec for Missing Fields

```bash
grep -n "managed_payments" openapi/spec3.yaml
# Result: No matches

grep -n "shared_payment_granted_token" openapi/spec3.yaml
# Result: No matches

grep -n "source:" openapi/spec3.yaml | grep -A2 "payment_intent"
# Result: No matches within payment_intent schema (lines 29110-29637)
```

---

## References

* **OpenAPI Spec File:** `spec3.yaml` (version 2026-03-25.dahlia)
* **Payment Intent Schema:** Lines 29110–29637
* **Endpoint Definition:** Lines 107194–107330
* **Amount Details Schema:** Lines 28436–28520
