# Context — Alsafwa Return Flow & Checkout Completion

## Decisions Log
| Date | Decision | Rationale |
|---|---|---|
| 2026-04-01 | Return fee: 22 SAR flat charge | Client selected option 2 |
| 2026-04-01 | Remove "Item is Damaged" return reason | Per Mr. Mohammed confirmation |
| 2026-04-01 | Policy acceptance required at checkout AND return | Client requirement |

## Pending Decisions
| Date | Proposal | Status |
|---|---|---|
| 2026-04-01 | Deduction model: SAR 44 (22+22) from final refund, min return order SAR 100 | Awaiting client confirmation |

## Key Notes
- Client accepted timeline but ALL pending items must be complete before go-live
- Pending: VAT invoice, checkout translation, return flow, checkout policy dialog
- Urgent: "very short time" to go-live
- Checkout policy dialog IS in scope
- Checkout translation = pre go-live activity (final work)

### Alternative Deduction Model (Proposed)
- SAR 44 total (22 outbound + 22 return) deducted from FINAL refund
- If refund < SAR 44: customer pays balance
- Minimum return order: SAR 100
- Benefit: reduces transactions, finance work, no repeated payments

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
