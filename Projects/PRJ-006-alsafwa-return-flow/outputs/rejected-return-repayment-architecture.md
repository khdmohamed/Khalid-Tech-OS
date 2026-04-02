# Re-Payment Flow for Rejected Returns — Architecture

**Project:** PRJ-006 — Alsafwa Return Flow
**Date:** 2026-04-02
**Scenario:** Customer return rejected → Customer pays SAR 22 again → Item shipped back

---

## Problem Statement

When a warehouse team rejects a return, the customer needs to:
1. See the rejection reason in My Orders
2. Pay SAR 22 return shipping fee again
3. Have the item shipped back to their address

**Challenge:** Shopify checkout cannot be "re-entered" for an existing order. Need alternative payment mechanism.

---

## Solution: Draft Order API + Payment Link

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REJECTED RETURN FLOW                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. Warehouse rejects return in Reverto                             │
│     └─ Enter rejection remarks                                      │
│     └─ Status: REJECTED                                            │
│                                                                     │
│  2. Reverto triggers webhook (or Shopify Flow automation)          │
│     └─ Payload: return_id, order_id, customer_email, rejection_remark │
│                                                                     │
│  3. Backend creates Draft Order via Shopify API                    │
│     └─ Line item: "Return Shipping Fee" - SAR 22                   │
│     └─ Customer: original customer email                            │
│     └─ Tags: "return-reorder-{original_order_id}"                  │
│                                                                     │
│  4. Draft Order API returns invoice_url (payment link)             │
│     └─ Send via email to customer                                  │
│     └─ Store in order metafield for display in My Orders           │
│                                                                     │
│  5. Customer clicks link → Pays SAR 22                             │
│     └─ Draft Order becomes Order                                   │
│     └─ Order tagged: "return-fee-paid"                             │
│                                                                     │
│  6. Warehouse ships item back to customer                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Shopify Draft Order API Implementation

### Create Draft Order

**Endpoint:** `POST /admin/api/2024-01/draft_orders.json`

```json
{
  "draft_order": {
    "line_items": [
      {
        "title": "Return Shipping Fee",
        "price": 22.00,
        "quantity": 1,
        "taxable": false,
        "requires_shipping": true
      }
    ],
    "customer": {
      "id": 1234567890
    },
    "billing_address": {
      "first_name": "Customer",
      "last_name": "Name",
      "address1": "Original Address",
      "city": "Jeddah",
      "country": "SA"
    },
    "shipping_address": {
      "first_name": "Customer",
      "last_name": "Name",
      "address1": "Original Address",
      "city": "Jeddah",
      "country": "SA"
    },
    "note": "Return shipping fee for rejected return - Original Order: #1001",
    "tags": "return-reorder-1001, return-fee-pending",
    "payment_terms": "To be paid via invoice",
    "apply_discount": false
  }
}
```

### Response (Contains Payment Link)

```json
{
  "draft_order": {
    "id": 987654321,
    "invoice_url": "https://checkout.shopify.com/xyz/abc123",
    "invoice_sent_at": null,
    "status": "open"
  }
}
```

---

## Theme Integration: Show "Pay Again" Button

### Detect Rejected Return Status

Reverto stores return status. Check via:
- Order tags: `return-status:rejected`
- Order metafield: `return.rejected = true`
- Reverto customer data (if exposed)

### Update `main-order.liquid`

```liquid
{% comment %} Check for rejected return requiring payment {% endcomment %}
{% assign return_rejected = false %}
{% assign return_payment_url = '' %}

{% comment %} Check order tags for return status {% endcomment %}
{% for tag in order.tags %}
  {% if tag contains 'return-rejected' %}
    {% assign return_rejected = true %}
  {% endif %}
  {% if tag contains 'return-payment-url' %}
    {% assign return_payment_url = tag | split: ':' | last %}
  {% endif %}
{% endfor %}

{% comment %} Or check metafields {% endcomment %}
{% if order.metafields.return.status == 'rejected' %}
  {% assign return_rejected = true %}
  {% assign return_payment_url = order.metafields.return.payment_url %}
{% endif %}

{% comment %} Display payment section if return rejected {% endcomment %}
{% if return_rejected %}
  <div class="return-rejected-notice" style="margin: 2rem 0; padding: 1.5rem; background: #fff3cd; border: 1px solid #ffc107; border-radius: 8px;">
    <h4 style="color: #856404; margin-top: 0;">Return Rejected</h4>
    <p style="color: #856404;">
      {% if order.metafields.return.rejection_reason %}
        Reason: {{ order.metafields.return.rejection_reason }}
      {% endif %}
    </p>

    {% if order.metafields.return.payment_required == 'true' %}
      <p style="color: #856404;">Please pay SAR 22 to have your item shipped back.</p>

      {% if return_payment_url != blank %}
        <a href="{{ return_payment_url }}" class="action-button btn-warning" target="_blank">
          Pay Return Shipping Fee (SAR 22)
        </a>
      {% else %}
        <p style="color: #856404;">Payment link will be sent to your email.</p>
      {% endif %}
    {% endif %}
  </div>
{% endif %}
```

---

## Return Status Badges (Add to main-order.liquid)

```liquid
{% comment %} Return Status Badge {% endcomment %}
{% if order.metafields.return.status %}
  <span class="order-status-badge status-{{ order.metafields.return.status }}">
    {% case order.metafields.return.status %}
      {% when 'requested' %}
        Return Requested
      {% when 'approved' %}
        Return Approved
      {% when 'rejected' %}
        Return Rejected
      {% when 'completed' %}
        Return Completed
    {% endcase %}
  </span>
{% endif %}
```

CSS to add:
```css
.status-requested { background: #d1ecf1; color: #0c5460; }
.status-approved { background: #d4edda; color: #155724; }
.status-rejected { background: #f8d7da; color: #721c24; }
.status-completed { background: #e2e3e5; color: #383d41; }
```

---

## Backend/Webhook Handler Options

### Option 1: Reverto Webhook (Recommended)

Configure Reverto to send webhook on return status change:
```
POST -> https://your-backend.com/webhooks/reverto/return-updated
```

Handler logic:
1. Parse return status
2. If `rejected`: create Draft Order via Shopify API
3. Store `invoice_url` in order metafield
4. Send email to customer with payment link

### Option 2: Shopify Flow (No Code)

Use Shopify Flow app:
```
Trigger: Order tagged with "return-rejected"
Action: Create Draft Order with SAR 22 fee
Action: Send email with invoice URL
```

### Option 3: Manual Process

Warehouse team manually:
1. Tags order with `return-rejected`
2. Creates draft order via Shopify Admin
3. Sends invoice email to customer

---

## Required Configuration

### 1. Create "Return Shipping Fee" Product

| Setting | Value |
|---|---|
| Product Title | Return Shipping Fee |
| Product Type | Service |
| Price | SAR 22.00 |
| Taxable | No |
| Requires Shipping | Yes (for tracking) |
| Inventory | Not tracked |

### 2. Order Metafields (Shopify Admin → Settings → Custom Data)

| Namespace | Key | Type | Description |
|---|---|---|---|---|
| return | status | Single line text | requested, approved, rejected, completed |
| return | payment_url | Single line text | Draft order invoice URL |
| return | payment_required | Boolean | Customer needs to pay again |
| return | rejection_reason | Multi line text | Warehouse remarks |
| return | return_id | Single line text | Reverto return reference |

### 3. Reverto Webhook Configuration

Check Reverto app settings for:
- Return status change webhooks
- Custom email templates
- Order tagging on status change

---

## Summary Checklist

| Item | Status |
|---|---|
| Create "Return Shipping Fee" product in Shopify | ⬜ Pending |
| Configure order metafields for return tracking | ⬜ Pending |
| Update `main-order.liquid` with rejection display | ⬜ Pending |
| Add return status badges CSS | ⬜ Pending |
| Configure Reverto webhook or Shopify Flow | ⬜ Pending |
| Create backend handler for Draft Order API | ⬜ Pending |
| Test rejected return → payment flow | ⬜ Pending |

---

## Alternative: Payment Link Product

If Draft Order API is not feasible, create approach:

1. Create hidden product "Return Shipping Fee - SAR 22"
2. Add to cart programmatically with redirect
3. Customer completes checkout

```liquid
<a href="/cart/add?id={{ return_fee_variant_id }}&quantity=1&return={{ order.id }}">
  Pay Return Shipping Fee
</a>
```

**Limitation:** Requires customer to go through full checkout flow (less ideal).
