# Return Flow — Cart-Based Payment Workaround

**Project:** PRJ-006
**Date:** 2026-04-02
**Approach:** Add fee to cart → Checkout → Pay → Redirect to Reverto

---

## Proposed Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CART-BASED RETURN FLOW                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. Customer: My Orders → Clicks "Request Return"                          │
│     └─ Instead of direct link to Reverto                                    │
│     └─ Run JavaScript that:                                                │
│        • Add "Return Shipping Fee" product to cart                          │
│        • Store order context in sessionStorage                              │
│        • Redirect to /checkout                                              │
│                                                                              │
│  2. Checkout: Customer sees cart with SAR 22 fee                            │
│     └─ Cart attribute notes which order this is for                        │
│     └─ Customer completes payment                                           │
│                                                                              │
│  3. Thank You Page: After successful payment                                │
│     └─ JavaScript checks sessionStorage                                     │
│     └─ If "pendingReturn" exists:                                          │
│        • Clear sessionStorage                                              │
│        • Redirect to Reverto return page                                   │
│        • Return fee already paid - show message                            │
│                                                                              │
│  4. Reverto: Customer completes return request                             │
│     └─ Fee already paid - no additional payment needed                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Implementation: Theme Changes

### Step 1: Create "Return Shipping Fee" Product

| Setting | Value |
|---|---|
| Product Title | Return Shipping Fee |
| Product Type | Service |
| Price | SAR 22.00 |
| Taxable | No |
| Requires Shipping | No |
| Inventory Policy | Don't track inventory |
| Visibility | Hidden (remove from all channels) |
| **Variant ID** | **COPY THIS ID** (needed for code) |

---

### Step 2: Modify main-order.liquid

**Location:** `sections/main-order.liquid`

**Find the return button:**
```liquid
<a href="/account?tab=Orders&page=1&sectionId=template--26051152118058__reverto"
  class="action-button btn-return">
  Request Return
</a>
```

**Replace with payment-first flow:**

```liquid
{% comment %} Return Request Button - UPDATED FOR UPFRONT PAYMENT {% endcomment %}
{% if button_action == 'return' %}
  <button onclick="initiateReturnWithPayment('{{ order.id }}', '{{ order.order_number }}')"
          class="action-button btn-return">
    {% if request.locale.iso_code == 'ar' %}
      طلب إرجاع
    {% else %}
      Request Return
    {% endif %}
  </button>
{% endif %}

{% comment %} Return Fee Modal {% endcomment %}
<div id="return-fee-modal" style="display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5);z-index:9999;align-items:center;justify-content:center;">
  <div style="position:relative;background:white;padding:30px;border-radius:8px;max-width:500px;width:90%;box-shadow:0 4px 20px rgba(0,0,0,0.15);">

    <h3 style="margin-top:0;">{% if request.locale.iso_code == 'ar' %}رسوم الشحن{% else %}Return Shipping Fee{% endif %}</h3>

    {% if request.locale.iso_code == 'ar' %}
      <p>سيتم فرض رسوم شحن قدرها 22 ريال لطلب الإرجاع.</p>
      <p style="font-weight:500;">سيتم توجيهك إلى صفحة الدفع لإتمام الدفع.</p>
      <label style="display:flex;align-items:start;gap:10px;">
        <input type="checkbox" id="return-fee-agree" style="margin-top:4px;">
        <span>أوافق على دفع 22 ريال كرسوم شحن الإرجاع والمتابعة إلى صفحة الدفع</span>
      </label>
    {% else %}
      <p>A return shipping fee of <strong>SAR 22</strong> will be charged for your return.</p>
      <p style="font-weight:500;">You will be redirected to checkout to complete the payment.</p>
      <label style="display:flex;align-items:start;gap:10px;">
        <input type="checkbox" id="return-fee-agree" style="margin-top:4px;">
        <span>I agree to pay SAR 22 return shipping fee and proceed to checkout</span>
      </label>
    {% endif %}

    <div style="margin-top:25px;display:flex;gap:10px;justify-content:flex-end;">
      <button onclick="hideReturnModal()" class="action-button" style="background:#6c757d;">
        {% if request.locale.iso_code == 'ar' %}إلغاء{% else %}Cancel{% endif %}
      </button>
      <button onclick="proceedToCheckoutWithFee()" class="action-button btn-return">
        {% if request.locale.iso_code == 'ar' %}متابعة للدفع{% else %}Continue to Payment{% endif %}
      </button>
    </div>
  </div>
</div>

{% comment %} Return fee payment flow JavaScript {% endcomment %}
<script>
// Configure this with your Return Shipping Fee variant ID
const RETURN_FEE_VARIANT_ID = {{ return_shipping_fee_variant_id | default: 'null' }};
let currentReturnOrder = null;
let currentReturnOrderName = null;

function initiateReturnWithPayment(orderId, orderName) {
  currentReturnOrder = orderId;
  currentReturnOrderName = orderName;
  showReturnModal();
}

function showReturnModal() {
  const modal = document.getElementById('return-fee-modal');
  modal.style.display = 'flex';
  document.body.style.overflow = 'hidden';
}

function hideReturnModal() {
  const modal = document.getElementById('return-fee-modal');
  modal.style.display = 'none';
  document.body.style.overflow = '';
}

function proceedToCheckoutWithFee() {
  const checkbox = document.getElementById('return-fee-agree');

  if (!checkbox.checked) {
    alert('{% if request.locale.iso_code == "ar" %}الرجاء الموافقة على رسوم الشحن والمتابعة{% else %}Please agree to the shipping fee to continue{% endif %}');
    return;
  }

  // Show loading state
  const proceedBtn = event.target;
  proceedBtn.disabled = true;
  proceedBtn.textContent = '{% if request.locale.iso_code == "ar" %}جاري التحضير...{% else %}Preparing...{% endif %}';

  // Add return fee to cart
  const cartData = {
    items: [{
      id: RETURN_FEE_VARIANT_ID,
      quantity: 1
    }],
    attributes: {
      '_return_for_order_id': currentReturnOrder,
      '_return_for_order_name': currentReturnOrderName,
      '_return_fee_payment': 'true'
    }
  };

  fetch('/cart/add.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(cartData)
  })
  .then(response => response.json())
  .then(data => {
    // Store context for after payment
    sessionStorage.setItem('pendingReturnOrderId', currentReturnOrder);
    sessionStorage.setItem('pendingReturnOrderName', currentReturnOrderName);
    sessionStorage.setItem('returnFeePaid', 'true');

    // Redirect to checkout
    window.location.href = '/checkout';
  })
  .catch(error => {
    console.error('Error:', error);
    alert('{% if request.locale.iso_code == "ar" %}حدث خطأ. الرجاء المحاولة مرة أخرى.{% else %}An error occurred. Please try again.{% endif %}');
    proceedBtn.disabled = false;
    proceedBtn.textContent = '{% if request.locale.iso_code == "ar" %}متابعة للدفع{% else %}Continue to Payment{% endif %}';
  });
}

// Close modal on outside click
document.getElementById('return-fee-modal')?.addEventListener('click', function(e) {
  if (e.target === this) {
    hideReturnModal();
  }
});
</script>
```

---

### Step 3: Add Post-Checkout Redirect

**Create new snippet:** `snippets/post-checkout-return-redirect.liquid`

```liquid
{% comment %}
  This snippet should be included in theme.liquid
  It runs on the thank you page to redirect to Reverto after payment
{% endcomment %}

<script>
(function() {
  // Only run on thank you page
  if (window.location.pathname !== '/thank_you') return;

  // Check if this was a return fee payment
  const pendingReturn = sessionStorage.getItem('pendingReturnOrderId');
  const returnFeePaid = sessionStorage.getItem('returnFeePaid');

  if (pendingReturn && returnFeePaid === 'true') {
    // Clear sessionStorage
    sessionStorage.removeItem('pendingReturnOrderId');
    sessionStorage.removeItem('pendingReturnOrderName');
    sessionStorage.removeItem('returnFeePaid');

    // Show message then redirect
    const thankYouMsg = document.querySelector('.main-page-title') || document.querySelector('h1');
    if (thankYouMsg) {
      const orderName = sessionStorage.getItem('pendingReturnOrderName') || '';
      thankYouMsg.innerHTML = `
        {% if request.locale.iso_code == 'ar' %}
        <h2>تم دفع رسوم الشحن بنجاح!</h2>
        <p>سيتم الآن توجيهك إلى إكمال طلب الإرجاع.</p>
        {% else %}
        <h2>Return Shipping Fee Paid!</h2>
        <p>You will now be redirected to complete your return request.</p>
        {% endif %}
      `;
    }

    // Redirect to Reverto after 2 seconds
    setTimeout(function() {
      var revertoUrl = '/account?tab=Orders&page=1&sectionId=template--26051152118058__reverto&return_fee_paid=true';
      window.location.href = revertoUrl;
    }, 2000);
  }
})();
</script>
```

**Add to `layout/theme.liquid` before closing </body>:**
```liquid
{% render 'post-checkout-return-redirect' %}
```

---

### Step 4: Update main-order.liquid Return Button Logic

**Original code showed order-specific return button. Need to ensure context passes correctly:**

```liquid
{% comment %} Updated return button with order context {% endcomment %}
{% if button_action == 'return' %}
  <button onclick="initiateReturnWithPayment('{{ order.id }}', '{{ order.name }}')"
          class="action-button btn-return">
    {% if request.locale.iso_code == 'ar' %}
      طلب إرجاع
    {% else %}
      Request Return
    {% endif %}
  </button>
{% endif %}
```

---

## Cart Cleanup (Important!)

The return fee should be the ONLY item in cart. Add checkout validation:

**In `layout/theme.liquid` or checkout scripts:**

```javascript
// If cart has non-return items and return attribute, warn customer
fetch('/cart.js')
  .then(r => r.json())
  .then(cart => {
    const hasReturnFee = cart.attributes._return_fee_payment === 'true';
    const hasOtherItems = cart.items.some(item =>
      item.product_title !== 'Return Shipping Fee'
    );

    if (hasReturnFee && hasOtherItems) {
      // Warn customer they should checkout separately
      console.warn('Return fee and regular items in cart - should checkout separately');
    }
  });
```

---

## Data Flow Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                        STEP 1: REQUEST                          │
├─────────────────────────────────────────────────────────────────┤
│  Customer clicks "Request Return"                                │
│  → Show modal with SAR 22 fee                                    │
│  → Customer agrees                                               │
│  → Add "Return Shipping Fee" to cart                             │
│  → Store order context in sessionStorage                         │
│  → Redirect to /checkout                                         │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        STEP 2: PAYMENT                           │
├─────────────────────────────────────────────────────────────────┤
│  Cart has 1 item: Return Shipping Fee (SAR 22)                  │
│  Cart attributes:                                                │
│    - _return_for_order_id: 1234567890                           │
│    - _return_for_order_name: #1001                              │
│    - _return_fee_payment: true                                   │
│  Customer completes payment                                       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      STEP 3: POST-PAYMENT                         │
├─────────────────────────────────────────────────────────────────┤
│  Thank you page loads                                            │
│  → Script checks sessionStorage                                  │
│  → Finds 'pendingReturnOrderId'                                  │
│  → Shows "Payment successful! Redirecting..."                    │
│  → 2 second delay                                               │
│  → Redirects to Reverto return page                             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      STEP 4: COMPLETE RETURN                     │
├─────────────────────────────────────────────────────────────────┤
│  Customer is in Reverto app                                      │
│  → URL param: ?return_fee_paid=true                             │
│  → Customer selects items, reason                                │
│  → Submits return (no additional payment needed)                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Benefits of This Approach

| Benefit | Explanation |
|---|---|
| ✅ **No Reverto dependency** | Works regardless of Reverto's payment features |
| ✅ **Clean UX** | Standard Shopify checkout flow |
| ✅ **Payment guaranteed** | Return only processes after successful payment |
| ✅ **Order context preserved** | Cart attributes track which order |
| ✅ **Arabic support** | Modal and messages fully translatable |

---

## Limitations & Considerations

| Issue | Mitigation |
|---|---|---|
| Customer has items in cart | Clear cart first OR warn customer |
| Return fee variant ID changes | Use theme setting to store ID |
| Reverto doesn't know fee was paid | URL param indicates fee paid |
| Multiple return requests | Each creates new order (acceptable) |

---

## Configuration Required

| Setting | Value | Where |
|---|---|---|
| Return Shipping Fee Variant ID | `{{ variant_id }}` | Theme settings or hardcode |
| Modal Text (Arabic/English) | Customizable | In modal HTML |
| Redirect Delay | 2000ms | In post-checkout script |
| Reverto URL | `/account?tab=Orders&page=1&sectionId=...` | Verified in theme |

---

## Testing Checklist

| Test | Expected Result |
|---|---|
| Click "Request Return" | Modal appears with SAR 22 fee |
| Don't agree, click Cancel | Modal closes, stays on order page |
| Agree, click Continue | Added to cart, redirects to checkout |
| Checkout shows | SAR 22 Return Shipping Fee only |
| Complete payment | Thank you page, then redirect to Reverto |
| In Reverto | Can complete return without payment |

---

## Implementation Priority

| Priority | Task | Effort |
|---|---|---|
| 🔴 P0 | Create Return Shipping Fee product | 5 min |
| 🔴 P0 | Update return button with modal | 2 hours |
| 🔴 P0 | Add to cart + redirect logic | 2 hours |
| 🟡 P1 | Post-checkout redirect snippet | 1 hour |
| 🟢 P2 | Cart conflict handling | 1 hour |
| 🟢 P2 | Testing + refinements | 2 hours |

**Total Estimate:** ~8 hours development
