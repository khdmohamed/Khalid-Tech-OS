# ZATCA Phase 2 Reference

## API Endpoints
- **Sandbox:** https://gw-fatoora.zatca.gov.sa/e-invoicing/developer-portal
- **Production:** https://gw-fatoora.zatca.gov.sa/e-invoicing/core

## Invoice Types
| Type | Code | Clearance Required |
|---|---|---|
| Standard Tax Invoice (B2B) | 388 | Yes (clearance) |
| Simplified Tax Invoice (B2C) | 381 | No (reporting only) |

## Key Fields
- `UUID`: unique per invoice
- `PIH`: Previous Invoice Hash (chain)
- `QR`: TLV Base64 encoded
- `CSID`: Compliance/Production certificate

## CSID Lifecycle
- Onboarding → Compliance CSID → Production CSID
- Renewal required before expiry
- Device-level, not company-level
