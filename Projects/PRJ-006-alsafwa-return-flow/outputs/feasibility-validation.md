# Return Flow Architecture — Feasibility Validation

**Project:** PRJ-006
**Date:** 2026-04-02
**Purpose:** Validate if proposed architecture is technically achievable

---

## Validation Results Summary

| Feature | Feasibility | Confidence | Notes |
|---|---|---|---|
| Upfront payment at return request | ⚠️ **UNCERTAIN** | LOW | Depends on Reverto app capabilities |
| Reverto rejection display | ⚠️ **MAYBE** | MEDIUM | Requires metafield/tag sync from Reverto |
| Rejected return re-payment | ✅ **FEASIBLE** | HIGH | Can use Draft Order API or cart add |
| Return status badges | ✅ **FEASIBLE** | HIGH | If Reverto writes data to order |
| Checkout policy dialog | ✅ **FEASIBLE** | HIGH | Custom Liquid + JS |

**Overall:** Architecture has **uncertain dependencies** on Reverto app features that need verification.

---

## Critical Unknowns: Reverto App Capabilities

### 1. Does Reverto Support Upfront Payment?

**Question:** Can Reverto require payment BEFORE return submission?

| Evidence | Finding |
|---|---|
| Theme code search | No payment flow found in Reverto snippets |
| Common returns app behavior | Most apps process returns POST-receipt, not pre-payment |
| Industry standard | Return labels typically generated AFTER approval |

**⚠️ RISK:** Reverto may NOT support "pay before return" model.

**Options if unsupported:**
| Option | Description | Complexity |
|---|---|---|
| A. Use Draft Order API | Create invoice, send to customer, they pay before return | Medium |
| B. Cart-based workaround | Add fee to cart, customer pays, then return request | High (UX fragmentation) |
| C. Change requirement | Process returns free, charge fee only on approval | Low (business logic change) |

**Action Required:** Check Reverto Admin → Settings → Payments for "Require payment before return" option.

---

### 2. How Does Reverto Communicate Return Status?

**Question:** When warehouse rejects return, how does theme know?

**Found in code:**
```liquid
{% comment %} Reverto uses metafields {% endcomment %}
shop.metafields.reverto_app.theme_settings
shop.metafields.reverto_app.customer_schema
```

**Possible mechanisms:**
| Mechanism | Feasible? | How to Verify |
|---|---|---|
| Order metafields | Maybe | Check if `order.metafields.return.status` exists |
| Order tags | Likely | Test rejection, check if order gets tagged |
| Reverto database only | Likely | Status stored in app, not Shopify |

**⚠️ RISK:** Reverto may store status internally without exposing to Shopify.

**Action Required:**
1. Create test return
2. Reject in Reverto
3. Check order tags/metafields for return status

---

### 3. Can We Intercept Return Request Flow?

**Question:** Can we show modal/add fee before customer completes return in Reverto?

**Current flow in theme:**
```liquid
{# From main-order.liquid #}
<a href="/account?tab=Orders&page=1&sectionId=template--26051152118058__reverto"
  class="action-button btn-return">
  Request Return
</a>
```

**Problem:** Return button goes directly to Reverto's embedded app interface.

| Approach | Feasibility | Notes |
|---|---|---|---|
| Intercept before Reverto | ❌ Difficult | Reverto controls its own UI |
| Customize Reverto UI | ❌ Not possible | App embed is sandboxed |
| Post-payment redirect | ✅ Possible | Pay → Then redirect to Reverto |

**⚠️ RISK:** Cannot easily inject payment step into Reverto's flow.

---

## Feature-by-Feature Feasibility

### ✅ FEASIBLE: Rejected Return Re-Payment

**Requirement:** Customer pays SAR 22 again to get rejected item back.

**Solution:**
```javascript
// Add return shipping fee to cart, redirect to checkout
fetch('/cart/add.js', {
  body: JSON.stringify({
    items: [{ id: returnFeeVariantId, quantity: 1 }],
    attributes: { 'Return Shipping For Order': orderNumber }
  })
})
.then(() => window.location.href = '/checkout')
```

**Status:** Standard Shopify functionality. No app dependency.

---

### ✅ FEASIBLE: Return Status Badges

**Requirement:** Show "Return Requested/Approved/Rejected" badges on order page.

**Dependency:** Order has metafield or tag with return status.

```liquid
{% if order.metafields.return.status %}
  <span class="status-{{ order.metafields.return.status }}">
    {{ order.metafields.return.status }}
  </span>
{% endif %}
```

**Status:** Code is simple. Depends on Reverto writing the data.

**Verification needed:** Does Reverto write to `order.metafields.return.*`?

---

### ⚠️ UNCERTAIN: Upfront Payment at Return Request

**Requirement:** Customer pays SAR 22 when requesting return.

**Challenge:** Reverto app controls the return request UI.

**Possible Workarounds:**

#### Workaround A: Pre-Payment via Draft Order
```
1. Customer clicks Request Return
2. Modal: "You must pay SAR 22 first"
3. Create Draft Order via API → Send invoice email
4. Customer pays invoice → Returns to site
5. Redirect to Reverto to complete return
```

**Pros:** Clean separation, uses Draft Order API
**Cons:** Two-step process, customer must return after payment

#### Workaround B: Hidden Product in Cart
```
1. Customer clicks Request Return
2. Add "Return Fee" product to cart
3. Redirect to checkout
4. After payment, redirect to Reverto
5. Session storage tracks original order
```

**Pros:** Single checkout flow
**Cons:** More complex state management

---

### ✅ FEASIBLE: Checkout Policy Dialog

**Requirement:** Show return policy checkbox at checkout.

```liquid
<div class="return-policy-dialog">
  <input type="checkbox" id="return-policy-agree" required>
  <label>I agree to the Return Policy</label>
</div>

<script>
  document.querySelector('[name="checkout"]').addEventListener('click', (e) => {
    if (!document.getElementById('return-policy-agree').checked) {
      e.preventDefault();
      alert('Please accept the Return Policy');
    }
  });
</script>
```

**Status:** Standard Liquid customization. No dependency on Reverto.

---

## Recommended Validation Steps

### Step 1: Reverto App Audit (Do this first!)

In Reverto Admin, check:

| Setting | Where to Look | What to Find |
|---|---|---|
| Upfront payment | Settings → Payments | "Require payment before return" toggle? |
| Fee collection | Settings → Fees | Can fee be collected at request vs approval? |
| Webhooks | Settings → Integrations | Webhook options for status changes? |
| Order tagging | Settings → Notifications | "Add tag to order" options? |
| Metafields | Settings → Advanced | "Write to order metafields" options? |

### Step 2: Test Return Flow

1. Create test order in Konooz store
2. Initiate return as customer
3. Check if payment is prompted
4. As warehouse, reject return
5. Check order tags/metafields

### Step 3: Ask Reverto Support

Key questions:
1. Can customers pay return fee BEFORE submitting return request?
2. Do you write return status to order metafields or tags?
3. Can you trigger webhooks on status change?
4. Can we customize the return request flow?

---

## Alternative: Simplified Flow

If Reverto doesn't support upfront payment, consider:

```
SIMPLIFIED FLOW (No Upfront Payment)
┌─────────────────────────────────────────────────────────────┐
│ 1. Customer requests return (free)                           │
│ 2. Warehouse receives item                                  │
│ 3. If APPROVED: Refund minus SAR 22                         │
│ 4. If REJECTED: Email customer + ask if they want it back   │
│    → Customer clicks link → Pays SAR 22 → Item shipped back │
└─────────────────────────────────────────────────────────────┘
```

**Benefits:**
- Works with any returns app
- No upfront payment complexity
- Customer only pays second time if they want rejected item back

**Downside:**
- More returns initiated (no upfront friction)
- Warehouse receives unwanted items

---

## Decision Matrix

| Issue | If Reverto Supports It | If Reverto Doesn't Support It |
|---|---|---|
| Upfront payment | ✅ Proceed as planned | ⚠️ Use Draft Order workaround OR change flow |
| Status metafields | ✅ Badges auto-populate | ⚠️ Manual tags OR Shopify Flow workaround |
| Rejection webhooks | ✅ Automated triggers | ⚠️ Manual process OR polling |

---

## Next Actions

| Priority | Action | Owner |
|---|---|---|
| 🔴 Critical | Audit Reverto Admin for payment/metafield settings | Khalid |
| 🔴 Critical | Test return flow in staging | Khalid |
| 🟡 Important | Contact Reverto support for capabilities | Khalid |
| 🟢 Nice to have | Document Draft Order API fallback | Backend |

---

## Conclusion

**Architecture is CONDITIONALLY FEASIBLE** depending on Reverto app capabilities.

**Showstoppers if Reverto lacks:**
- Upfront payment → Requires workaround or flow change
- Order metafield sync → Requires alternative status tracking

**Recommended:** Complete Reverto audit before committing to development.
