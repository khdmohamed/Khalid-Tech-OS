# Context — Alsafwa Return Flow & Checkout Completion

## Decisions Log
| Date | Decision | Rationale |
|---|---|---|
| 2026-04-01 | Return fee: 22 SAR flat charge | Client selected option 2 |
| 2026-04-01 | Remove "Item is Damaged" return reason | Per Mr. Mohammed confirmation |
| 2026-04-01 | Policy acceptance required at checkout AND return | Client requirement |
| 2026-04-02 | SAR 44 deduction model: REJECTED | Client chose original flow (per Mr. Mohammed) |
| 2026-04-02 | Minimum return order: NO restriction | Customer pays return fee regardless |

## Scope Dispute
| Item | Client Position | Techasus Position | Status |
|---|---|---|---|
| Checkout policy dialog + return fee display | IN SCOPE — "We provided complete structure" | OUT OF SCOPE — requires custom Shopify dev | **OPEN** |

**Client Argument:** "We did not receive any return structure from Techasus side we provided return complete structure so we need to complete this same in the scope"

## Pending Decisions
| Date | Proposal | Status |
|---|---|---|
| 2026-04-02 | Checkout policy dialog scope — resolve dispute | Awaiting Techasus leadership |

## Key Notes
- Client accepted timeline but ALL pending items must be complete before go-live
- Pending: VAT invoice, checkout translation, return flow, **checkout policy dialog (DISPUTED)**
- Urgent: "very short time" to go-live
- **Return flow:** Original flow with multiple payments (SAR 22 upfront, SAR 22 again if rejected)
- **No minimum return order** — customer pays fee regardless

### Return Flow Spec
1. Customer selects items → Return Policy page → must accept
2. After accept: Pay button for 22 SAR shipment fee
3. Payment successful → Return order created → Tracking available
4. Warehouse receives item:
   - **Approved**: Approval email + visible in My Orders
   - **Rejected**: Remarks via email + My Orders, customer gets Pay button (22 SAR again)
5. Re-paid → Item shipped back to customer

## References
<!-- Shopify store URL, BC environment, staging links -->
