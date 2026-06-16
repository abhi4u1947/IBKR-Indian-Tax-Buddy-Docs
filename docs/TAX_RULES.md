# Tax Rule Engine — IBKR Indian Tax Assistant

> **Reference only.** Tax law is complex and changes frequently. The rules below
> are encoded as **versioned data** and flagged as assumptions where
> interpretation is involved. **A certified Indian CA must verify all outputs.**

## Why a rule engine (not hardcoded rules)

Indian tax rules change with each Union Budget (and via circulars/notifications).
Hardcoding rates/thresholds means code changes (and risk) every year. Instead:

- Rules live in `config/tax_rules/india_residents.yaml` as **effective-dated,
  versioned** rule sets, loaded into versioned DB tables.
- The engine resolves the applicable rule **by the sale date**, so historical
  filings always use the law in force at the time.
- Updates are config + data changes (PR → seed), with a full **audit trail**.

## What a rule set carries

`asset_class × residency_status × [effective_from, effective_to)` →
holding-period threshold, STCG/LTCG rate treatment (incl. slab vs flat,
indexation flag, LTCG surcharge cap), surcharge bands (by total income), cess —
plus governance: `is_assumption`, `confidence`, `requires_user_confirmation`,
`legal_reference`, `notes`, and a `source_config_hash`.

## Encoded treatment for foreign securities (residents / ROR)

> **Key interpretive point (flagged).** Foreign listed shares are generally **not**
> eligible for the concessional Section 112A regime (which applies to STT-paid
> *Indian* listed equity). They are commonly treated like unlisted/"other"
> capital assets: **24-month** LTCG threshold; STCG taxed at the taxpayer's
> **slab** rate.

| Regime | LTCG threshold | LTCG rate | Indexation | STCG |
|---|---|---|---|---|
| Up to 22-Jul-2024 | 24 months | 20% | Yes | Slab |
| From 23-Jul-2024 (Finance (No.2) Act 2024) | 24 months | **12.5%** | **No** | Slab |

Plus 4% Health & Education Cess and individual surcharge bands (10/15/25/37%),
with an LTCG surcharge cap (15%) flagged as an assumption to confirm.

Parallel (lower-confidence, flagged) rule sets exist for **foreign ETFs**,
**foreign mutual funds** (specified-fund/debt-orientation nuances), and a
**catch-all** for other foreign securities.

## How classification works (precise)

- LTCG boundary = `acquisition_date + relativedelta(months = threshold)`
  (true calendar months, **not** a 30-day approximation).
- Long-term iff `sale_date >= boundary` (inclusive flag configurable).
- Cost is FX-converted on the **acquisition** date, proceeds on the **sale**
  date, per the configured FX strategy.

## FX strategy (configurable, flagged)

`config/tax_rules/fx_strategy.yaml` selects how foreign amounts convert to INR.
Default: **SBI TT buying rate on the last day of the month preceding the
transaction**. Alternatives: RBI reference rate on the transaction date;
settlement-date basis. Every conversion stores its rate, date and source
reference for audit.

## Updating rules (yearly)
1. Edit `config/tax_rules/india_residents.yaml` — add a **new** effective-dated
   version (never edit a past one in place).
2. Open a PR (CI validates structure, types, and non-overlapping ranges).
3. Deploy → `make seed` (idempotent; writes `tax_rule_audit`).

## Assumptions surfaced to the user
Every calculation collects `AssumptionFlag`s (tax-rule interpretation, FX basis,
corporate-action holding-period tacking). The UI must show these and require the
user to acknowledge **CA verification** before final reports are generated.
