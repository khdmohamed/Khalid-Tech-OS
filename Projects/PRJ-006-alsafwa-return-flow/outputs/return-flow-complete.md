# Return Flow Architecture — Complete (Corrected)

**Project:** PRJ-006 — Alsafwa Return Flow
**Date:** 2026-04-02

---

## Complete Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PHASE 1: RETURN REQUEST                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. Customer: My Orders → Request Return                                    │
│  2. Customer selects items + reason                                         │
│  3. Modal: "SAR 22 return shipping fee will be charged"                     │
│  4. Customer agrees → Checkout → Pays SAR 22                                │
│  5. Return request created (status: REQUESTED)                              │
│                                                                              │
│  → SAR 22 covers: Shipping FROM customer TO warehouse                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         PHASE 2: WAREHOUSE RECEIPT                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. Warehouse receives returned item                                       │
│  2. Warehouse inspects item                                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         PHASE 3: WAREHOUSE DECISION                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  OPTION A: APPROVED                                                 │    │
│  ├─────────────────────────────────────────────────────────────────────┤    │
│  │  1. Warehouse marks: Approved                                      │    │
│  │  2. Customer receives: Approval email + visible in My Orders        │    │
│  │  3. Refund processed: (Order amount - SAR 22 shipping fee)         │    │
│  │  4. Warehouse keeps/disposes item (depending on reason)             │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  OPTION B: REJECTED                                                 │    │
│  ├─────────────────────────────────────────────────────────────────────┤    │
│  │  1. Warehouse marks: Rejected + enters remarks                     │    │
│  │  2. Customer sees in My Orders:                                     │    │
│  │     - Rejection status (red badge)                                  │    │
│  │     - Rejection reason/remarks                                      │    │
│  │     - "Do you want this item back?" + [Pay SAR 22] button          │    │
│  │  3. Customer decision:                                              │    │
│  │                                                                      │    │
│  │     ┌──────────────────────────────────────────────────────────┐   │    │
│  │     │ CHOICE A: No, I don't want it back                        │   │    │
│  │     ├──────────────────────────────────────────────────────────┤   │    │
│  │     │ - Warehouse disposes item                                │   │    │
│  │     │ - No further action                                      │   │    │
│  │     └──────────────────────────────────────────────────────────┘   │    │
│  │                                                                      │    │
│  │     ┌──────────────────────────────────────────────────────────┐   │    │
│  │     │ CHOICE B: Yes, ship it back to me                        │   │    │
│  │     ├──────────────────────────────────────────────────────────┤   │    │
│  │     │ - Click [Pay SAR 22] button                             │   │    │
│  │     │ - Redirect to checkout → Pay SAR 22                     │   │    │
│  │     │ - Warehouse ships item back to customer address         │   │    │
│  │     └──────────────────────────────────────────────────────────┘   │    │
│  │                                                                      │    │
│  │  → SAR 22 covers: Shipping FROM warehouse TO customer                │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Fee Structure Summary

| Scenario | Fee | Who Pays | When |
|---|---|---|---|
| Return request | SAR 22 | Customer | At return request (before warehouse receives) |
| Return approved | SAR 22 | Customer | Already paid — deducted from refund |
| Return rejected + item not wanted | SAR 0 | No one | Already paid — not refunded |
| Return rejected + item wanted back | SAR 22 | Customer | After seeing rejection (second payment) |

---

## Theme Implementation: Rejected Return Display

**File:** `sections/main-order.liquid`

```liquid
{% comment %} Check for rejected return via order metafield or tag {% endcomment %}
{% assign return_rejected = false %}
{% assign rejection_reason = '' %}
{% assign return_payment_url = '' %}

{% comment %} Check metafields {% endcomment %}
{% if order.metafields.return.status == 'rejected' %}
  {% assign return_rejected = true %}
  {% assign rejection_reason = order.metafields.return.rejection_reason %}
  {% assign return_payment_url = order.metafields.return.payment_url %}
{% endif %}

{% comment %} Or check tags {% endcomment %}
{% for tag in order.tags %}
  {% if tag contains 'return-rejected' %}
    {% assign return_rejected = true %}
  {% endif %}
{% endfor %}

{% comment %} Display rejected return section {% endcomment %}
{% if return_rejected %}
  <div class="return-rejected-notice"
       style="margin: 2rem 0; padding: 1.5rem; background: #f8d7da;
              border: 1px solid #f5c6cb; border-radius: 8px;">

    <h4 style="color: #721c24; margin-top: 0; display: flex; align-items: center; gap: 10px;">
      <span class="order-status-badge status-rejected">Return Rejected</span>
      {% if request.locale.iso_code == 'ar' %}
        تم رفض الإرجاع
      {% endif %}
    </h4>

    {% if rejection_reason != blank %}
      <p style="color: #721c24; font-weight: 500;">
        {% if request.locale.iso_code == 'ar' %}
          سبب الرفف:
        {% else %}
          Reason:
        {% endif %}
        {{ rejection_reason }}
      </p>
    {% endif %}

    <p style="color: #721c24;">
      {% if request.locale.iso_code == 'ar' %}
        هل تريد إعادة الشحن إليك؟ يرجى الدفع مقابل تكلفة الشحن.
      {% else %}
        Do you want this item shipped back to you? Please pay the shipping cost.
      {% endif %}
    </p>

    {% comment %} Pay button for return shipping {% endcomment %}
    <button onclick="payReturnShipping('{{ order.id }}')"
            class="action-button btn-warning"
            style="margin-top: 1rem;">
      {% if request.locale.iso_code == 'ar' %}
        ادفع 22 ريال للشحن
      {% else %}
        Pay SAR 22 for Return Shipping
      {% endif %}
    </button>

    <p style="color: #721c24; font-size: 0.9em; margin-top: 1rem;">
      {% if request.locale.iso_code == 'ar' %}
        سيتم شحن الطرد إلى عنوانك بعد الدفع.
      {% else %}
        Your item will be shipped to your address after payment.
      {% endif %}
    </p>
  </div>
{% endif %}

<script>
function payReturnShipping(orderId) {
  {% comment %}
    Option 1: Add return shipping fee to cart and redirect to checkout
    Option 2: Create draft order and redirect to invoice URL
  {% endcomment %}

  var returnFeeProductId = {{ return_shipping_fee_variant_id }};

  fetch('/cart/add.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      items: [{
        id: returnFeeProductId,
        quantity: 1
      }],
      attributes: {
        'Return Shipping For Order': '{{ order.order_number }}',
        'Return Shipping Type': 'Rejected Return - Ship Back'
      }
    })
  })
  .then(response => response.json())
  .then(data => {
    sessionStorage.setItem('returnShippingForOrder', '{{ order.id }}');
    window.location.href = '/checkout';
  })
  .catch(error => {
    console.error('Error:', error);
    alert('Failed to add shipping fee. Please try again.');
  });
}

{% comment %} After payment, update order status {% endcomment %}
if (window.location.pathname === '/thank_you') {
  var returnShippingOrder = sessionStorage.getItem('returnShippingForOrder');
  if (returnShippingOrder) {
    {% comment %}
      Payment successful - notify backend or update metafield
      Backend should: order.metafields.return.ship_back_paid = true
    {% endcomment %}
    sessionStorage.removeItem('returnShippingForOrder');

    {% comment %}
      Optionally redirect to order page to see updated status
    {% endcomment %}
    // window.location.href = '/account/orders/' + returnShippingOrder;
  }
}
</script>
```

---

## Return Status Badges

```liquid
{% comment %} Add to order title area {% endcomment %}
{% if order.metafields.return.status %}
  <span class="order-status-badge status-{{ order.metafields.return.status }}">
    {% case order.metafields.return.status %}
      {% when 'requested' %}
        {{ 'customer.return.requested' | t | default: 'Return Requested' }}
      {% when 'approved' %}
        {{ 'customer.return.approved' | t | default: 'Return Approved' }}
      {% when 'rejected' %}
        {{ 'customer.return.rejected' | t | default: 'Return Rejected' }}
      {% when 'shipped_back' %}
        {{ 'customer.return.shipped_back' | t | default: 'Shipped Back' }}
    {% endcase %}
  </span>
{% endif %}
```

**Add to CSS:**
```css
.status-requested { background: #d1ecf1; color: #0c5460; }
.status-approved { background: #d4edda; color: #155724; }
.status-rejected { background: #f8d7da; color: #721c24; }
.status-shipped_back { background: #fff3cd; color: #856404; }
```

---

## Data Flow: How Reverto Communicates Rejection

```
Warehouse rejects in Reverto
  ↓
Reverto needs to communicate this to Shopify for display
```

**Options:**

| Method | How | Effort |
|---|---|---|
| **Reverto Webhook** | Configure Reverto → webhook → update order metafield | Medium (need backend) |
| **Shopify Flow** | Trigger: Order tagged `return-rejected` → Update metafield | Low (if Reverto tags) |
| **Reverto Email + Manual** | Email customer + support manually updates order | High (manual) |
| **Reverto Metafield Sync** | Reverto writes to `order.metafields.return.*` directly | Check if supported |

**Check Reverto settings** for:
- Webhooks on status change
- Order tagging capability
- Metafield write access

---

## Products Needed

| Product | Purpose | Price |
|---|---|---|
| Return Shipping Fee | Shipping TO warehouse (paid at return request) | SAR 22 |
| Return Ship Back Fee | Shipping BACK to customer (paid after rejection) | SAR 22 |

**Can be same product** with different line item attribute/notes.

---

## Implementation Checklist

| Phase | Task | Status |
|---|---|---|
| **Request** | Create return shipping fee product | ⬜ |
| | Add return fee modal at return request | ⬜ |
| | Checkout flow for return fee payment | ⬜ |
| | Post-payment redirect to Reverto | ⬜ |
| **Approval** | Configure approval email + My Orders badge | ⬜ |
| | Refund calculation (minus SAR 22) | ⬜ |
| **Rejection** | Configure Reverto to write rejection status | ⬜ |
| | Display rejection notice + remarks | ⬜ |
| | "Pay to ship back" button + checkout flow | ⬜ |
| | Notify warehouse when payment complete | ⬜ |
| **Backend** | Webhook handler for Reverto status changes | ⬜ |
| | Draft order creation for ship-back payment | ⬜ |
| | Order metafield updates | ⬜ |
