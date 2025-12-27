# Pricing Calculator Reference

> Internal tool for calculating client pricing and ROI projections.

## Standard Pricing

### Setup Fee

| Component | Price |
|-----------|-------|
| Configuration & Integration | $1,500 |
| AI Training & Customization | $500 |
| Testing & Verification | $300 |
| Documentation & Handoff | $200 |
| **Total Setup** | **$2,500** |

### Monthly Service

| Plan | Monthly Fee | Included |
|------|-------------|----------|
| Standard | $497 | Up to 500 emails/month, email support |
| Growth | $997 | Unlimited emails, priority support, monthly review |

---

## ROI Calculation Tool

### Input Variables

```
A = Monthly inquiries
B = Current response time (hours)
C = Current booking rate (%)
D = Average deal value ($)
E = Close rate (%)
```

### Formulas

**Current Monthly Revenue:**
```
Current Meetings = A × (C / 100)
Current Customers = Current Meetings × (E / 100)
Current Revenue = Current Customers × D
```

**Projected Monthly Revenue (with Co-Pilot):**
```
// Use 35% booking rate for sub-5-minute response
Projected Booking Rate = 35%
Projected Meetings = A × 0.35
Projected Customers = Projected Meetings × (E / 100)
Projected Revenue = Projected Customers × D
```

**Impact:**
```
Additional Revenue = Projected Revenue - Current Revenue
Monthly ROI = Additional Revenue / Monthly Fee
Annual ROI = (Additional Revenue × 12 - Setup Fee) / (Monthly Fee × 12 + Setup Fee)
```

---

## Example Calculations

### Small Business (50 inquiries/month)

| Metric | Value |
|--------|-------|
| Monthly inquiries | 50 |
| Current response time | 14 hours |
| Current booking rate | 10% |
| Projected booking rate | 35% |
| Deal value | $2,000 |
| Close rate | 40% |

**Results:**
- Current: 5 meetings → 2 customers → $4,000/mo
- Projected: 17.5 meetings → 7 customers → $14,000/mo
- **Additional Revenue: $10,000/month**
- **Monthly ROI: 20x** ($10,000 / $497)

### Medium Business (150 inquiries/month)

| Metric | Value |
|--------|-------|
| Monthly inquiries | 150 |
| Current response time | 12 hours |
| Current booking rate | 12% |
| Projected booking rate | 35% |
| Deal value | $5,000 |
| Close rate | 35% |

**Results:**
- Current: 18 meetings → 6.3 customers → $31,500/mo
- Projected: 52.5 meetings → 18.4 customers → $91,875/mo
- **Additional Revenue: $60,375/month**
- **Monthly ROI: 60x** ($60,375 / $997)

---

## Break-Even Analysis

### Standard Plan ($497/mo + $2,500 setup)

| Deal Value | Deals to Break Even (Monthly) | Deals for Setup |
|------------|-------------------------------|-----------------|
| $1,000 | 0.5 | 2.5 |
| $2,500 | 0.2 | 1.0 |
| $5,000 | 0.1 | 0.5 |
| $10,000 | 0.05 | 0.25 |

### Time to Recoup Setup

| Additional Revenue/Month | Months to Recoup |
|--------------------------|------------------|
| $2,500 | 1 month |
| $5,000 | 2 weeks |
| $10,000 | 1 week |
| $25,000 | 3 days |

---

## Discount Guidelines

### Volume Discounts

| Annual Commitment | Discount |
|-------------------|----------|
| Prepay 6 months | 5% |
| Prepay 12 months | 10% |

### Referral Credits

- Referral results in signed client: 1 month free service
- Applicable to referring client's account

### Multi-Inbox Pricing

| Additional Inboxes | Price Per Inbox |
|-------------------|-----------------|
| 2nd inbox | +$297/month |
| 3rd+ inbox | +$197/month each |

---

## Competitive Positioning

### vs. Human Virtual Assistant

| Aspect | Email Co-Pilot | VA |
|--------|---------------|-----|
| Monthly cost | $497-997 | $2,000-4,000 |
| Response time | < 5 min | 30 min - 4 hours |
| Availability | 24/7/365 | Business hours |
| Scalability | Unlimited | Constrained |
| Consistency | 100% | Variable |

### vs. Generic Chatbot

| Aspect | Email Co-Pilot | Chatbot |
|--------|---------------|---------|
| Channel | Email (where leads are) | Website only |
| Context | Full business training | Generic |
| Scheduling | Real calendar integration | Link to Calendly |
| Voice | Matches your brand | Robotic |

### vs. Manual Process

| Aspect | Email Co-Pilot | Manual |
|--------|---------------|--------|
| Response time | < 5 min | 8-14 hours |
| After-hours | Instant | Next day |
| Consistency | 100% | Variable |
| Time investment | 0 hours | 15+ hours/month |

---

## Proposal Template Values

When preparing proposals, use these standard values:

```
Setup Fee: $2,500
Standard Monthly: $497
Growth Monthly: $997

Expected booking rate improvement: 2-3x
Expected response time: < 5 minutes
Expected time savings: 15+ hours/month

Standard assumptions:
- Current booking rate: 10-15%
- Projected booking rate: 30-40%
- Industry close rate: 30-50%
```

---

## Notes

- All pricing in USD
- Prices effective December 2024
- Subject to annual review
- Custom enterprise pricing available
