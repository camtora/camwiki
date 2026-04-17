---
title: "Decision: Estimated COA revenue model using fixed per-product rates"
type: decision
tags: [decision, adr, revenue, coa, business-logic]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Estimated COA revenue model using fixed per-product rates

**Date:** 2026-04-17 (reconstructed from commit de479e1)
**Status:** `accepted`

## Context

DonorMap shows a revenue figure alongside donor counts. Actual revenue for recurring donors (monthly, quarterly, annual) is not known at acquisition time — it depends on churn, payment success, and lifetime. The dashboard needs a live number.

## Options Considered

1. **Actual PRODUCTAMOUNT only** — show first-payment amount for all products. Misleading for recurring donors (a $15/month donor isn't worth $15 total).
2. **Lifetime value (LTV) calculation** — query Snowflake historical churn to calculate true LTV. Too slow for 15-minute cache refresh; churn data changes slowly.
3. **Estimated COA rates** — fixed rates per product type, labelled as "EST. REVENUE" with COA note. Fast, reproducible, honest about being an estimate.

## Decision

Fixed Cost of Acquisition (COA) rates applied per product type:

| Product type | Rate |
|-------------|------|
| Recurring (monthly/quarterly/annual) — US | $305 USD |
| Recurring (monthly/quarterly/annual) — Canada | $350 CAD |
| OTG (one-time gift) | Actual `PRODUCTAMOUNT` from DB |

Global view converts US revenue to CAD at a fixed 1.43 USD/CAD rate.

Displayed as "EST. REVENUE" in the UI with an explicit COA annotation — transparent that these are estimates, not actuals.

## Consequences

- Numbers are management estimates, not accounting figures. Suitable for the broadcast dashboard use case (real-time acquisition monitoring) but not for financial reporting.
- COA rates are hardcoded in `refreshCache.ts` — changing rates requires a code deploy.
- OTG revenue is exact; recurring revenue is estimated. Mixed precision in the same total figure.

## Related Pages

- [[wiki/projects/donormap]]
- [[wiki/decisions/donormap/firestore-cache-intermediary]]

## Sources

- `raw/repos/donormap` — functions/src/refreshCache.ts (lines 145–195), docs/data-validation.md (lines 124–156), commit de479e1
