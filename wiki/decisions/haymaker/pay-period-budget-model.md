---
title: "Decision: Pay-period budget model, not calendar months"
type: decision
tags: [decision, adr, budget, finance, domain-model]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Pay-period budget model, not calendar months

**Date:** 2026-04-16 (reconstructed from docs/debt-payoff-forecaster.md and commit history)
**Status:** `accepted`

## Context

Most budgeting apps organize income and expenses by calendar month. Haymaker is used by someone whose income arrives on a fixed pay cycle. The budget module needed a foundational abstraction for organizing cashflow.

## Options Considered

1. **Calendar month** — standard; aligns with billing cycles for most fixed expenses (rent, subscriptions). All budgeting software defaults to this. Mismatch between paycheck timing and month boundary creates edge cases ("I get paid on the 15th, so my month starts the 15th").
2. **Pay period** — a rolling window anchored to payday (`pay_date` + `pay_period_days` in `BudgetSettings`). Income always lands at the start of the period; expenses are modeled within that window. Cashflow projected as a sequence of pay periods forward.

## Decision

Pay-period abstraction stored in `BudgetSettings` (`pay_date`, `pay_period_days`). All income and expense occurrences are tied to these periods. Cashflow is calculated as a rolling sequence of "what comes in / what goes out within each period."

This maps to reality: the user thinks "I have $X this paycheque; after rent, groceries, and debt minimums, what's left?" not "what's my January deficit?" The pay period is the planning unit.

The **debt payoff forecaster** uses pay periods as its simulation unit: it iterates forward period-by-period, applying minimum payments, cascading freed minimums to the next priority debt when one is paid off, and inserting lump sums at their scheduled dates. Calendar-month simulation would create irregular period lengths and misalign with actual payment timing.

## Consequences

- Cashflow forecast and scenario planning are all pay-period native — comparisons across periods are apples-to-apples.
- Debt forecaster binary search (goal-based payment mode) operates in pay periods; result shows target payoff date and required payment per period.
- Credit card integration (CC fields sync bidirectionally: APR, minimum payment, balance, due day) aligns CC due dates with pay periods for cashflow planning.
- "Expected/actual balance" display compares projected vs actual bank balance at any point in the current period.
- Calendar-month reporting would require period-to-calendar conversion — not currently needed.

## Related Pages

- [[wiki/projects/haymaker]]

## Sources

- `raw/repos/haymaker/docs/debt-payoff-forecaster.md` — debt strategies, payment modes, lump sum logic
- `raw/repos/haymaker` — apps/api/app/models.py (BudgetSettings, IncomeSource, RecurringExpense, Debt), apps/api/app/routes.py (debt-forecast endpoints)
