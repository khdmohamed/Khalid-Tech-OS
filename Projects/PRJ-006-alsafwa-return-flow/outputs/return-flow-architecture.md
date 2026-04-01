# Return Flow Architecture — Alsafwa (Reverto App)

**Project:** PRJ-006
**Date:** 2026-04-01
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

## Reverto App — Key Configuration Areas

### 1. Return Fee Configuration

**Location:** Reverto Admin → Settings → Return Fees

| Setting | Value | Notes |
|---|---|---|
| Return Fee Type | Flat Fee | SAR 22 per return |
| Fee Deduction | From Refund | Deducted from final refund amount |
| Minimum Return Value | SAR 100 | Client proposal (pending approval) |
| Restocking Fee | SAR 22 | Outbound shipping cost recovery |

**Fee Logic:**
```
Total Deduction = SAR 22 (outbound) + SAR 22 (return) = SAR 44
If (Refund Amount < SAR 44) → Customer pays difference
Net Refund = (Order Total - SAR 44) - Any other deductions
```

### 2. Return Policy Acceptance

**Location:** Reverto Admin → Settings → Return Policy

| Setting | Configuration |
|---|---|
| Policy Acceptance Required | ✅ Enabled |
| Acceptance Point 1 | Before Return Request Submission |
| Acceptance Point 2 | Checkout (via Liquid customization) |
| Policy Display | Modal/Dialog checkbox "I agree to Return Policy" |

**Implementation:**
- Reverto handles return page policy acceptance (built-in)
- Checkout policy requires Liquid theme edit (custom)

### 3. Return Reasons

**Location:** Reverto Admin → Settings → Return Reasons

| Reason | Enabled | Action |
|---|---|---|
| Wrong Item | ✅ | - |
| Changed Mind | ✅ | - |
| Defective/Damaged | ✅ | - |
| Item is Damaged | ❌ | **Removed per client** |

### 4. Warehouse Actions

**Location:** Reverto Admin → Returns Management

| Action | Trigger | Customer Notification |
|---|---|---|
| Approve | Warehouse receives + inspects | Email + My Orders status |
| Reject | Warehouse finds issue | Email + My Orders + Remarks |
| Request Payment | Rejected return | Pay button appears (SAR 22) |

---

## Return Flow — Step by Step

### Phase 1: Customer Initiates Return

```
1. Customer → My Orders → Request Return
2. Select item(s) → Select reason (NO "Item is Damaged")
3. Redirect: Return Policy page (Reverto)
4. Checkbox: "I accept the Return Policy"
5. Click: Continue → Pay Return Fee
6. Payment Gateway: Process SAR 22
7. Success → Return Order Created
8. Tracking: Available in My Orders
```

### Phase 2: Warehouse Processing

```
9. Warehouse receives item
10. Scan/lookup return order in Reverto
11. Inspection:
    ├─ APPROVED → Mark as Approved
    │              └─ Trigger: Approval email
    │              └─ Update: My Orders "Approved"
    │              └─ Process: Refund minus SAR 44
    │
    └─ REJECTED → Enter rejection remarks
                   └─ Trigger: Rejection email with remarks
                   └─ Update: My Orders "Rejected" + remarks
                   └─ Action: Pay button appears (SAR 22 again)
                   └─ Customer pays → Item shipped back
```

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

### Settings → Integrations

| Integration | Status |
|---|---|
| Shopify | ✅ Connected |
| Business Central | ⚠️ Configure webhooks |
| Payment Gateway | ✅ Use store gateway |

---

## Data Flow — Shopify ↔ BC

```
┌─────────────────────────────────────────────────────────────────┐
│                        RETURN CREATION                           │
├─────────────────────────────────────────────────────────────────┤
│  1. Customer submits return in Reverto (Shopify)                │
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
│  2. Shopify calculates: (Order Total - SAR 44 - Discounts)      │
│  3. Shopify → Payment Gateway: Process refund                   │
│  4. Webhook → BC: Refund posted                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Shopify Liquid — Checkout Policy Dialog

**File:** `sections/checkout.liquid` or `theme.liquid`

```liquid
{% comment %} Return Policy Acceptance at Checkout {% endcomment %}

<script>
document.addEventListener('DOMContentLoaded', function() {
  const checkoutBtn = document.querySelector('[name="checkout"]');
  const policyCheckbox = document.getElementById('return-policy-agree');

  if (checkoutBtn && policyCheckbox) {
    checkoutBtn.addEventListener('click', function(e) {
      if (!policyCheckbox.checked) {
        e.preventDefault();
        alert('Please accept the Return Policy to proceed');
        return false;
      }
    });
  }
});
</script>

<div class="return-policy-dialog" style="margin: 20px 0; padding: 15px; border: 1px solid #ddd; border-radius: 4px;">
  <h4>{{ 'Return Policy' | t }}</h4>
  <p>{{ 'Returns are subject to a SAR 22 fee per item. Minimum return order value: SAR 100.' | t }}</p>
  <a href="/pages/return-policy" target="_blank">{{ 'View full policy' | t }}</a>
  <label style="display: flex; align-items: center; margin-top: 10px;">
    <input type="checkbox" id="return-policy-agree" name="return_policy_agree" required>
    <span style="margin-left: 8px;">{{ 'I have read and agree to the Return Policy' | t }}</span>
  </label>
</div>

{% comment %} Display return fee at checkout {% endcomment %}
<div class="checkout-return-fee-info" style="font-size: 0.9em; color: #666;">
  {{ 'Note: A SAR 22 return shipping fee applies to all returns.' | t }}
</div>
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

## Open Questions

| Question | Impact | Status |
|---|---|---|
| SAR 44 deduction model approval? | Determines if payment required upfront | Pending client |
| Minimum return order SAR 100? | Prevents low-value returns | Pending client |
| Checkout policy dialog location? | Theme customization required | To be confirmed |
| BC webhook for return sync? | Inventory visibility | To be configured |

---

## Next Steps

1. ✅ Confirm SAR 44 deduction model with client
2. ⏳ Configure Reverto fee settings (SAR 22)
3. ⏳ Remove "Item is Damaged" return reason
4. ⏳ Configure notification templates (Arabic/English)
5. ⏳ Implement checkout policy dialog (Liquid)
6. ⏳ Set up BC webhooks for return sync
7. ⏳ Test full return flow (UAT)
