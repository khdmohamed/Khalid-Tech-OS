# Skill: ZATCA Invoice Builder

Generate a ZATCA Phase 2 compliant UBL 2.1 XML invoice.

## Required Inputs
- Invoice type (simplified B2C / standard B2B)
- Seller VAT number
- Buyer VAT number (B2B only)
- Line items (description, qty, unit price, VAT rate)
- Invoice date/time
- CSID reference

## Output
- Valid UBL 2.1 XML
- TLV QR code data
- Validation notes for edge cases
