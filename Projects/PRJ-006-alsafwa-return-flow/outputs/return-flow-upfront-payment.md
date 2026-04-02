# Return Flow Architecture — Upfront Payment Model

**Project:** PRJ-006 — Alsafwa Return Flow
**Date:** 2026-04-02
**Updated:** Corrected flow - payment at return REQUEST, not after rejection

---

## Correct Flow: Pay First, Then Return

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CUSTOMER RETURN REQUEST                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. Customer goes to My Orders → Clicks "Request Return"                    │
│     └─ Opens Reverto return request page                                    │
│                                                                              │
│  2. Customer selects items to return + reason                               │
│                                                                              │
│  3. BEFORE submitting:                                                      │
│     └─ Show modal: "Return shipping fee of SAR 22 will be charged"          │
│     └─ Checkbox: "I agree to pay SAR 22 return shipping fee"                │
│                                                                              │
│  4. Customer clicks "Continue to Payment"                                   │
│     └─ Redirect to checkout with SAR 22 fee                                 │
│     └─ OR: Add fee to cart + redirect to checkout                           │
│                                                                              │
│  5. Customer pays SAR 22                                                     │
│     └─ Payment successful                                                   │
│                                                                              │
│  6. AFTER payment: Return request is created/submitted                       │
│     └─ Status: REQUESTED                                                    │
│     └─ Tracking available                                                   │
│                                                                              │
│  7. Warehouse processes return:                                             │
│     ├─ APPROVED → Refund processed (minus SAR 22)                           │
│     └─ REJECTED → Item shipped back to customer (already paid)              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Key Difference: Rejected Return = Already Paid

When warehouse rejects a return:
- **Customer already paid SAR 22** at return request
- No need for re-payment
- Item is shipped back to customer's address
- The SAR 22 covers the return shipping cost

**Customer only pays once** — at the time of return request.

---

## Implementation Options

### Option A: Reverto App Built-in Payment (Best)

Check if Reverto has built-in "pay before return" feature:
- Reverto Admin → Settings → Return Fees
- Look for "Require payment before return submission"
- Configure fee amount (SAR 22)

**If Reverto supports this:** Enable and configure — no custom code needed.

---

### Option B: Custom Flow via Theme (If Reverto Doesn't Support)

If Reverto doesn't have upfront payment, build custom flow:

#### Step 1: Create Return Shipping Fee Product

| Setting | Value |
|---|---|
| Product Title | Return Shipping Fee |
| Product Type | Service |
| Price | SAR 22.00 |
| Taxable | No |
| Requires Shipping | No |
| Inventory Policy | Don't track inventory |
| Visibility | Hidden (remove from all channels) |

#### Step 2: Modify Return Flow in Theme

**Location:** Where "Request Return" button appears

```liquid
{% comment %} Original return button - MODIFY THIS {% endcomment %}
{% if button_action == 'return' %}
  {% comment %} Instead of direct link to Reverto, show modal first {% endcomment %}
  <button onclick="showReturnFeeModal()" class="action-button btn-return">
    {% if request.locale.iso_code == 'ar' %}
      طلب إرجاع
    {% else %}
      Request Return
    {% endif %}
  </button>
{% endif %}

{% comment %} Return Fee Modal {% endcomment %}
<div id="return-fee-modal" style="display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5);z-index:9999;">
  <div style="position:relative;top:50%;left:50%;transform:translate(-50%,-50%);background:white;padding:30px;border-radius:8px;max-width:500px;">
    <h3>Return Shipping Fee</h3>

    {% if request.locale.iso_code == 'ar' %}
      <p>سيتم فرض رسوم شحن قدرها 22 ريال لإرجاع الطلب.</p>
      <p>هل توافق على دفع رسوم الشحن ومتابعة طلب الإرجاع؟</p>
      <label>
        <input type="checkbox" id="return-fee-agree-ar">
        أوافق على دفع 22 ريال كرسوم شحن الإرجاع
      </label>
    {% else %}
      <p>A return shipping fee of SAR 22 will be charged.</p>
      <p>Do you agree to pay the fee and continue with your return request?</p>
      <label>
        <input type="checkbox" id="return-fee-agree">
        I agree to pay SAR 22 return shipping fee
      </label>
    {% endif %}

    <div style="margin-top:20px;display:flex;gap:10px;">
      <button onclick="proceedToReturnPayment()" class="action-button btn-return">
        Continue to Payment
      </button>
      <button onclick="hideReturnFeeModal()" class="action-button">
        Cancel
      </button>
    </div>
  </div>
</div>

<script>
function showReturnFeeModal() {
  document.getElementById('return-fee-modal').style.display = 'block';
}

function hideReturnFeeModal() {
  document.getElementById('return-fee-modal').style.display = 'none';
}

function proceedToReturnPayment() {
  var checkbox = document.getElementById('return-fee-agree') || document.getElementById('return-fee-agree-ar');

  if (!checkbox.checked) {
    alert('{% if request.locale.iso_code == "ar" %}الرجاء الموافقة على رسوم الشحن{% else %}Please agree to the shipping fee{% endif %}');
    return;
  }

  {% comment %} Add return fee to cart and redirect to checkout {% endcomment %}
  var returnUrlFeeProductId = {{ return_fee_variant_id }}; // Set this variable

  fetch('/cart/add.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      items: [{
        id: returnUrlFeeProductId,
        quantity: 1
      }],
      attributes: {
        'Return For Order': '{{ order.order_number }}',
        'Return Type': 'Return Shipping Fee'
      }
    })
  })
  .then(response => response.json())
  .then(data => {
    {% comment %} Store order context for after payment {% endcomment %}
    sessionStorage.setItem('pendingReturnOrder', '{{ order.id }}');
    sessionStorage.setItem('pendingReturnOrderName', '{{ order.name }}');

    {% comment %} Redirect to checkout {% endcomment %}
    window.location.href = '/checkout';
  })
  .catch(error => console.error('Error:', error));
}

{% comment %} After checkout, redirect to Reverto to complete return request {% endcomment %}
if (window.location.pathname === '/thank_you') {
  var pendingReturn = sessionStorage.getItem('pendingReturnOrder');
  if (pendingReturn) {
    {% comment %}
      Payment successful - now redirect to Reverto to complete return
      The return fee is already paid, so Reverto should know
    {% endcomment %}
    sessionStorage.removeItem('pendingReturnOrder');

    {% comment %}
      Option 1: Redirect to Reverto with payment confirmed
      Option 2: Store payment confirmation in customer metafield for Reverto to check
    {% endcomment %}

    // Redirect to Reverto return page
    var revertoUrl = '/account?tab=Orders&page=1&sectionId=template--26051152118058__reverto&return_fee_paid=true';
    window.location.href = revertoUrl;
  }
}
</script>
```

---

### Option C: Shopify Flow + Draft Order (Simpler)

```
Customer clicks Request Return
  → Shopify Flow triggers
  → Creates draft order with SAR 22 fee
  → Sends invoice email to customer
  → Customer pays invoice
  → Redirected to Reverto to complete return
```

**Less seamless** but easier to implement.

---

## Rejected Return Flow (Simplified)

Since customer already paid SAR 22 upfront:

```
Warehouse rejects return
  → No re-payment needed
  → Reverto sends rejection email with remarks
  → Item shipped back to customer's address
  → SAR 22 covers this return shipping
```

**No payment link needed** at rejection stage.

---

## Configuration Checklist

| Item | Status |
|---|---|---|
| Check if Reverto has built-in upfront payment | ⬜ |
| Create "Return Shipping Fee" hidden product | ⬜ |
| Add return fee modal to order page | ⬜ |
| Add to cart → checkout flow | ⬜ |
| Post-checkout redirect to Reverto | ⬜ |
| Test: Return request → Pay → Return created | ⬜ |
| Test: Rejected return → Item shipped back | ⬜ |

---

## Summary

| Phase | Previous (Wrong) | Corrected |
|---|---|---|
| Return Request | Free to request | Pay SAR 22 first |
| Payment | After rejection (re-pay) | At return request (one-time) |
| Rejected Return | Customer pays again | Already paid — just ship back |
| Approved Return | Deduct SAR 44 | Deduct SAR 22 (already paid) |
