# Command: /zatca-checklist

Run through ZATCA Phase 2 compliance checklist for a given invoice or implementation.

## Checks
- [ ] UBL 2.1 XML structure valid
- [ ] QR code generated (TLV encoded)
- [ ] CSID active and not expired
- [ ] Cryptographic stamp present
- [ ] Buyer/Seller info complete (VAT numbers)
- [ ] Tax-inclusive pricing (for B2C)
- [ ] Invoice hash chained correctly
- [ ] Clearance/Reporting mode correct for invoice type
