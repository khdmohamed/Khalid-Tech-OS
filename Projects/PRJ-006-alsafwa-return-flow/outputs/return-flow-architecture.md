# Return Flow Architecture — Alsafwa (Reverto App)

**Project:** PRJ-006
**Date:** 2026-04-02 (Updated)
**App:** Reverto (Shopify Returns & Exchanges)

---

## System Context

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Customer  │──────│   Shopify   │──────│   Reverto   │
│  (Browser)  │      │  (Store)    │      │    App      │
└─────────────┘      └─────────────┘      └─────────────┘
                            │                      │
                            ▼                      ▼
                     ┌─────────────┐      ┌─────────────┐
                     │  Business   │◄─────│  Payments   │
                     │  Central    │      │  Gateway    │
                     └─────────────┘      └─────────────┘
```

---

## Complete Return Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PHASE 1: RETURN REQUEST                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. Customer → My Orders → Request Return                                    │
│  2. Select item(s) → Select reason (NO "Item is Damaged")                    │
│  3. Modal: "SAR 22 return shipping fee will be charged"                     │
│  4. Customer agrees → Checkout → Pays SAR 22                                │
│  5. Return request created (status: REQUESTED)                              │
│  6. Tracking available in My Orders                                         │
│                                                                              │
│  → SAR 22 covers: Shipping FROM customer TO warehouse                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         PHASE 2: WAREHOUSE PROCESSING                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. Warehouse receives returned item                                       │
│  2. Warehouse inspects item in Reverto                                      │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  OPTION A: APPROVED                                                 │    │
│  ├─────────────────────────────────────────────────────────────────────┤    │
│  │  1. Warehouse marks: Approved                                      │    │
│  │  2. Customer receives: Approval email + "Approved" badge in Orders  │    │
│  │  3. Refund processed: (Order amount - SAR 22 shipping fee)         │    │
│  │  4. Warehouse keeps/disposes item                                  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  OPTION B: REJECTED                                                 │    │
│  ├─────────────────────────────────────────────────────────────────────┤    │
│  │  1. Warehouse marks: Rejected + enters remarks                     │    │
│  │  2. Customer sees in My Orders:                                     │    │
│  │     - "Rejected" badge (red)                                       │    │
│  │     - Rejection reason/remarks                                     │    │
│  │     - Message: "Do you want this item back?"                       │    │
│  │     - [Pay SAR 22] button to ship back                            │    │
│  │  3. Customer chooses:                                              │    │
│  │     - No action → Warehouse disposes item                         │    │
│  │     - Clicks [Pay SAR 22] → Pays → Item shipped back              │    │
│  │                                                                      │    │
│  │  → SAR 22 covers: Shipping FROM warehouse TO customer                │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Fee Structure

| Scenario | Fee | When Paid | Covers |
|---|---|---|---|
| Return request | SAR 22 | At request (before warehouse) | Shipping → Warehouse |
| Return approved | SAR 0 | Already paid | Deducted from refund |
| Return rejected + item wanted back | SAR 22 | After rejection | Shipping → Customer |
| Return rejected + item not wanted | SAR 0 | - | Disposal |

**Summary:** Customer pays **once or twice** depending on warehouse decision.

---

## Reverto App — Key Configuration Areas

### 1. Return Fee Configuration

**Location:** Reverto Admin → Settings → Return Fees

| Setting | Value | Notes |
|---|---|---|
| Return Fee Type | Flat Fee | SAR 22 per return |
| Fee Collection | At time of return request | Before warehouse receives |
| Fee Refundable | ❌ No | Non-refundable |
| Minimum Return Value | ❌ None | Per client (no minimum) |

### 2. Return Policy Acceptance

| Setting | Configuration |
|---|---|
| Policy Acceptance Required | ✅ Enabled |
| Acceptance Point | Before Return Request Submission |
| Policy Display | Modal/Dialog checkbox "I agree to Return Policy" |

### 3. Return Reasons

| Reason | Enabled | Action |
|---|---|---|
| Wrong Item | ✅ | - |
| Changed Mind | ✅ | - |
| Defective/Damaged | ✅ | - |
| Item is Damaged | ❌ | **Removed per client** |

### 4. Warehouse Actions

| Action | Trigger | Customer Notification |
|---|---|---|
| Approve | Warehouse receives + approves | Email + My Orders badge |
| Reject | Warehouse rejects + remarks | Email + My Orders + remarks + Pay button |

---

## Configuration Checklist — Reverto Admin

### Settings → General

| Setting | Value |
|---|---|
| Allow Returns | ✅ Yes |
| Return Window | X days from delivery |
| Auto-approve returns | ❌ No (manual warehouse approval) |

### Settings → Fees

| Setting | Value |
|---|---|
| Return Shipping Fee | SAR 22 |
| Fee Collection | At time of return request |
| Fee Refundable | ❌ No (non-refundable) |

### Settings → Notifications

| Notification | Enabled | Template |
|---|---|---|
| Return Received | ✅ | Custom: Arabic + English |
| Return Approved | ✅ | Custom: Arabic + English |
| Return Rejected | ✅ | Custom: includes remarks |
| Payment Required | ✅ | Custom: link to pay |

---

## Theme Implementation: Rejected Return Display

**File:** `sections/main-order.liquid`

```liquid
{% comment %} Check for rejected return {% endcomment %}
{% assign return_rejected = false %}
{% assign rejection_reason = '' %}

{% if order.metafields.return.status == 'rejected' %}
  {% assign return_rejected = true %}
  {% assign rejection_reason = order.metafields.return.rejection_reason %}
{% endif %}

{% comment %} Display rejected return section {% endcomment %}
{% if return_rejected %}
  <div class="return-rejected-notice" style="margin: 2rem 0; padding: 1.5rem; background: #f8d7da; border: 1px solid #f5c6cb; border-radius: 8px;">
    <h4 style="color: #721c24;">
      <span class="order-status-badge status-rejected">Return Rejected</span>
    </h4>

    {% if rejection_reason != blank %}
      <p style="color: #721c24;"><strong>Reason:</strong> {{ rejection_reason }}</p>
    {% endif %}

    <p style="color: #721c24;">
      Do you want this item shipped back to you? Please pay the shipping cost.
    </p>

    <button onclick="payReturnShipping('{{ order.id }}')" class="action-button btn-warning">
      Pay SAR 22 for Return Shipping
    </button>

    <p style="color: #721c24; font-size: 0.9em; margin-top: 1rem;">
      Your item will be shipped to your address after payment.
    </p>
  </div>
{% endif %}

<script>
function payReturnShipping(orderId) {
  var returnFeeProductId = {{ return_shipping_fee_variant_id }};

  fetch('/cart/add.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      items: [{ id: returnFeeProductId, quantity: 1 }],
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
  });
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
      {% when 'requested' %}Return Requested
      {% when 'approved' %}Return Approved
      {% when 'rejected' %}Return Rejected
      {% when 'shipped_back' %}Shipped Back
    {% endcase %}
  </span>
{% endif %}
```

**CSS:**
```css
.status-requested { background: #d1ecf1; color: #0c5460; }
.status-approved { background: #d4edda; color: #155724; }
.status-rejected { background: #f8d7da; color: #721c24; }
.status-shipped_back { background: #fff3cd; color: #856404; }
```

---

## BC — Return Order Configuration

### Setup

| Area | Configuration |
|---|---|
| Item Tracking | Lot/Serial (if applicable) |
| Return Reason Codes | Map to Reverto reasons |
| Warehouse Receipt | Automatic on return approval |
| Refund Journal | Automatic trigger from Shopify |

### Return Reasons Mapping

| Reverto | BC Reason Code | Description |
|---|---|---|
| Wrong Item | WRONG_ITEM | Customer received incorrect item |
| Changed Mind | CHANGED_MIND | Customer no longer wants item |
| Defective | DEFECTIVE | Item arrived defective |
| *Item is Damaged* | *DISABLED* | Removed per client |

---

## Data Flow — Shopify ↔ BC

```
┌─────────────────────────────────────────────────────────────────┐
│                        RETURN CREATION                           │
├─────────────────────────────────────────────────────────────────┤
│  1. Customer pays SAR 22 → Return request created              │
│  2. Reverto creates:                                            │
│     - Shopify Return Order (refund_pending)                     │
│     - Return Record in Reverto DB                               │
│  3. Webhook → BC: Return Order created                          │
│  4. BC: Reserve inventory (expected return)                     │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      WAREHOUSE RECEIPT                          │
├─────────────────────────────────────────────────────────────────┤
│  1. Warehouse scans item → BC Item Ledger Entry (return)       │
│  2. BC → Reverto webhook: Item received                         │
│  3. Warehouse marks Approved/Rejected in Reverto                │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       REFUND PROCESSING                          │
├─────────────────────────────────────────────────────────────────┤
│  1. Approved: Reverto triggers refund via Shopify               │
│  2. Shopify calculates: (Order Total - SAR 22 - Discounts)     │
│  3. Shopify → Payment Gateway: Process refund                   │
│  4. Webhook → BC: Refund posted                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Decisions Log

| Date | Decision | Status |
|---|---|---|
| 2026-04-01 | Return fee: SAR 22 flat charge | ✅ Confirmed |
| 2026-04-01 | Remove "Item is Damaged" return reason | ✅ Confirmed |
| 2026-04-02 | SAR 44 deduction model: REJECTED | ✅ Client chose original flow |
| 2026-04-02 | Minimum return order: NO restriction | ✅ Confirmed |
| 2026-04-02 | Payment at return request (upfront) | ✅ Confirmed |
| 2026-04-02 | Rejected return: Pay again to ship back | ✅ Confirmed |

---

## Scope Dispute

| Item | Client Position | Techasus Position | Status |
|---|---|---|---|
| Checkout policy dialog + return fee display | IN SCOPE | OUT OF SCOPE (custom dev) | **OPEN** |

---

## Next Steps

| # | Task | Status |
|---|---|---|
| 1 | Configure Reverto fee settings (SAR 22 at request) | ⬜ Pending |
| 2 | Remove "Item is Damaged" return reason | ⬜ Pending |
| 3 | Configure notification templates (Arabic/English) | ⬜ Pending |
| 4 | Implement return fee modal at request | ⬜ Pending |
| 5 | Implement rejected return display + pay button | ⬜ Pending |
| 6 | Set up BC webhooks for return sync | ⬜ Pending |
| 7 | Test full return flow (UAT) | ⬜ Pending |
