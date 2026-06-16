# Changelog — IBKR Indian Tax Assistant

All notable changes to this project are documented here. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/). The project is pre-1.0.

> **Reference assistance only.** All tax outputs are estimates and **must be
> verified by a certified CA in India** before filing.

## [Unreleased]

Substantially complete system across deterministic engines, API, and web. Below
is grouped by area. Legend: **(tested)** = covered by automated tests;
**(gated)** = implemented but activates only with external credentials/keys.

### Tax engine — `packages/tax-core` (60 tests)
- Versioned, effective-dated **tax-rule engine** (models, loader, resolver,
  repository); rules resolved by sale date, no hardcoding. (tested)
- **Cost basis**: FIFO and Specific-ID matching. (tested)
- **Corporate-action replay** (`costbasis/replay.py` `process_events`): splits,
  reverse-splits, bonus, spin-offs, mergers, rights, with lot-adjustment audit.
  (tested)
- **Capital-gains calculator** emitting explainable `CalculationRecord`s and
  flagged `AssumptionFlag`s; STCG/LTCG, surcharge, cess. (tested)
- **FX engine** + strategy with providers: `static_sample` (in-memory),
  `rbi` (RBI reference) and `sbi_tt` (SBI TT). Providers are safe-by-default
  (raise without an injected transport). (tested; live RBI/SBI fetch gated)
- **Dividends + Foreign Tax Credit** (`dividends/`): gross/withholding/net +
  creditable FTC. (tested)
- **Schedule FA** builder (`schedulefa/`) for foreign-asset reporting. (tested)
- `Decimal`-only money (`money.py`); `float` rejected at boundaries. (tested)

### Ingestion — `packages/ibkr-ingest` (14 tests)
- **Flex Query XML parser** → normalized trades + corporate actions. (tested)
- **Live Flex Web Service client** (`flex_query/client.py`): real two-step
  SendRequest → GetStatement flow over stdlib `urllib`, with retry/backoff and
  an injectable opener for tests. (tested; live fetch gated on Flex token/query)
- **Sample statement provider** with built-in fixtures. (tested)

### Reports — `packages/tax-reports` (5 tests)
- Schedule builders + **CSV / HTML / XLSX** renderers with embedded disclaimer
  and full provenance (rule_set_id, FX source refs). (tested)

### Assistant — `packages/tax-assistant` (6 tests)
- Deterministic **keyword retrieval** over a user's `CapitalGainsResult`.
  (tested)
- **`TemplateExplainer`** — deterministic, no hallucination, record-ID
  citations. (tested)
- **`LLMExplainer`** protocol (implemented by the API's Anthropic explainer).

### API — `apps/api` (56 tests)
- FastAPI app with routers: health, auth, admin, rules, imports, calculations,
  questionnaire, documents, assistant, workflow, schedules. (tested)
- **Auth**: register / verify-email / login / password-reset / **MFA (TOTP)**;
  JWT (HS256) bearer tokens; **RBAC** roles (investor/tax_consultant/admin) with
  `require_role` guards. (tested)
- **Rotatable refresh tokens** persisted server-side as SHA-256 hashes with
  reuse detection (`refresh_token` table, migration `0003`), and an **email
  service** abstraction (console default; optional SMTP). (tested)
- **Admin**: users, rules, audit, health, idempotent rules reseed, versioned
  rules. (tested)
- **Persisted workflow**: authenticated ingest → persist → calculate → persist,
  plus run list/detail with ownership checks. (tested)
- **Documents** demo endpoints (CSV/HTML/XLSX). (tested)
- **Assistant** endpoints: demo (template) + `/explain` (Claude via
  `AnthropicExplainer`, default `claude-sonnet-4-6`, with deterministic
  fallback) + health. (tested; live LLM gated on `ANTHROPIC_API_KEY`)
- **Schedules** endpoints: Schedule FA + dividends/FTC. (tested)
- **Persistence service** (idempotent on `sha256`), **seed service**.
- **Observability**: structured JSON logging, `X-Request-ID` middleware,
  `/health`, `/health/ready`, `/metrics`, Sentry-ready hook. (tested)
- **Broker-token encryption** at rest (Fernet; KMS envelope intended for prod).
- **SQLAlchemy 2.0 models** + **Alembic** migrations (`0001_initial`,
  `0002_user_mfa_secret`, `0003_refresh_token`).
- **Celery** worker skeleton.

### Web — `apps/web` (30 Vitest tests)
- Next.js marketing site with SEO (metadata, OpenGraph/Twitter, JSON-LD,
  sitemap/robots) and **INR pricing** with GST.
- Auth (login/register), onboarding wizard, dashboard, results, assistant,
  schedule-fa pages.
- Admin portal: users, rules, health, audit.
- Tested: api-client, pricing, utils, assistant, disclaimer. (tested)

### Shared types — `packages/ts-shared`
- TypeScript mirror of API response shapes (Decimal-as-string), enums, and
  questionnaire types.

### Infrastructure & CI/CD
- **Reference Terraform** for AWS (ap-south-1) + Cloudflare: VPC/network, RDS
  Postgres, ElastiCache Redis, S3 (SSE-KMS), ECS/Fargate + ALB + autoscaling,
  ACM/DNS, Secrets Manager (out-of-band values). (gated on cloud apply)
- **docker-compose** (postgres, redis, api, worker, web) + **Makefile** targets.
- **GitHub Actions**: Python tests with **85% tax-core coverage floor**, web
  typecheck (required), Docker build validation, **CodeQL** scanning,
  Dependabot.

### Documentation
- Architecture, system design, database/ER, **API reference**, **changelog**,
  security, cost estimates, deployment, tax rules, pricing/GTM, roadmap,
  disclaimer.

### Known gaps (credential/account-gated or planned)
- Live IBKR Flex token/query id + Client Portal OAuth end-to-end wiring.
- Live RBI/SBI FX HTTP fetch transports + caching into `fx_rate`.
- `ANTHROPIC_API_KEY` for live Claude answers.
- Production cloud apply + custom domain + DNS/SSL; transactional email creds.
- Persisted (per-calc-run) document endpoint (currently HTTP 501).
