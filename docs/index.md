# IBKR Indian Tax Assistant

**Global tax assistant for Indian investors.** It helps **Indian tax residents**
who invest globally through **Interactive Brokers (IBKR)** compute Indian
capital-gains and dividend tax on foreign securities, and produce **audit-ready,
CA-reviewable working papers** — every number explained and sourced.

!!! danger "Reference assistance only — CA verification required"
    All tax computations produced by this application are **estimates** generated
    from imported data and configurable, versioned tax rules. **You must have all
    outputs reviewed and verified by a certified Chartered Accountant (CA)
    registered in India before filing any return or relying on the figures.**
    This product is **not** tax, legal, financial, or investment advice.

    > "Tax calculations are estimates and should be reviewed by a qualified tax
    > professional before filing."

[Go-Live Runbook :material-rocket-launch:](GO_LIVE_RUNBOOK.md){ .md-button .md-button--primary }
[Architecture :material-sitemap:](ARCHITECTURE.md){ .md-button }
[Screenshots :material-image-multiple:](SCREENSHOTS.md){ .md-button }

---

## Why this exists

Indian residents holding US/global stocks (RSUs, ESPPs, self-directed) face a
genuinely hard filing problem: FIFO / specific-lot cost basis across corporate
actions, INR conversion at the right RBI/SBI rates, STCG vs LTCG classification
for foreign assets (which changed materially in **Budget 2024**), dividend
withholding + foreign tax credit, and **Schedule FA** reporting. This tool
automates the math **transparently** — every figure is explained and sourced —
so you and your CA can trust and verify it.

## Feature overview

- **Versioned tax-rule engine** — rates, thresholds, surcharge and cess are
  effective-dated **data**, resolved by sale date. A new Budget is a config
  change, not a code change.
- **Cost basis** — FIFO and Specific-ID matching, with **corporate-action
  replay** (splits, bonus, spin-offs, mergers, rights) and a full lot-adjustment
  audit trail.
- **FX → INR engine** — `static_sample`, **RBI** reference and **SBI TT**
  providers (live fetch credential-gated).
- **Capital gains** — STCG / LTCG, surcharge and cess, each gain emitting an
  explainable `CalculationRecord`.
- **Dividends + Foreign Tax Credit** — gross / withholding / net plus creditable
  FTC.
- **Schedule FA** builder for foreign-asset reporting.
- **IBKR ingest** — Flex Query XML parser plus a **live Flex Web Service client**
  (live fetch needs a Flex token + query id).
- **Audit-ready reports** — CSV / HTML / XLSX document generation.
- **Grounded AI assistant** — deterministic `TemplateExplainer` with an optional
  Claude-backed `AnthropicExplainer`; **citations, no hallucinations**.
- **FastAPI backend** — auth / JWT / RBAC / MFA, admin, imports, calculations,
  questionnaire, documents, assistant, persisted workflow, observability,
  Alembic migrations and a Celery skeleton.
- **Next.js web app** — marketing + SEO + INR pricing + auth + onboarding +
  dashboard + results + assistant + Schedule FA + admin.

## Build status

Substantially complete across deterministic engines, API and web —
**~171 automated tests** (141 Python + 30 web).

| Suite | Tests |
|---|---|
| `packages/tax-core` | 60 |
| `packages/ibkr-ingest` | 14 |
| `packages/tax-reports` | 5 |
| `packages/tax-assistant` | 6 |
| `apps/api` | 56 |
| `apps/web` (Vitest) | 30 |
| **Total** | **~171** |

Legend: ✅ implemented + tested · 🔌 implemented but **credential/key-gated**
(activates with real secrets — see the [Go-Live Runbook](GO_LIVE_RUNBOOK.md)).

## Core principles (enforced in code)

1. **`Decimal` everywhere** — `float` is rejected at boundaries; tax-grade precision.
2. **No hardcoded tax rules** — versioned, effective-dated data resolved by the sale date.
3. **Effective-dating + audit trails** on rules and every lot adjustment.
4. **Interpretive choices are data** — surfaced as `AssumptionFlag`s requiring
   user confirmation **and CA verification**, never silently baked in.
5. **Explainability is first-class** — every gain emits a `CalculationRecord`
   (inputs, intermediates, rule version, FX sources).

## Where to go next

- **Take it live** → [Go-Live Runbook](GO_LIVE_RUNBOOK.md) — exact commands,
  env vars and secrets.
- **Understand the system** → [Architecture](ARCHITECTURE.md) ·
  [System Design](SYSTEM_DESIGN.md) · [Database](DATABASE.md) ·
  [API Reference](API_REFERENCE.md)
- **Tax & compliance** → [Tax Rules](TAX_RULES.md) · [Security](SECURITY.md) ·
  [Disclaimer](DISCLAIMER.md)
- **Business** → [Pricing & GTM](PRICING_GTM.md) · [Cost Estimates](COST_ESTIMATES.md)
- **See it** → [Screenshots](SCREENSHOTS.md)
