# Roadmap — IBKR Indian Tax Assistant

> **Reference assistance only.** All outputs are estimates and **must be verified
> by a certified CA in India** before filing. CA verification is mandatory at
> every milestone.

Status legend: ✅ implemented + tested · 🔌 implemented but **credential/key/
account-gated** (activates when secrets are supplied) · ⏳ planned.

## Milestone 1 — Architecture + deterministic core ✅
- Monorepo scaffold, docker-compose, Makefile, docs.
- **`tax-core`**: versioned tax-rule engine, FIFO/Specific-ID cost basis,
  corporate-action replay, FX engine, Indian capital-gains calculator with
  explainable records + flagged assumptions.
- **`ibkr-ingest`**: Flex Query XML parser + sample provider.
- API scaffold with `/health`, `/api/v1/rules`, demo calculation, questionnaire;
  SQLAlchemy models + initial Alembic migration; rule seeding.
- Web scaffold: marketing site, onboarding wizard, dashboard, SEO, INR pricing.

## Milestone 2 — Persistence + import pipeline ✅ (live transports 🔌)
- ✅ SQLAlchemy repos wired end-to-end; imports, lots, calc runs persisted
  (idempotent on `sha256`) via the authenticated `/api/v1/workflow/*` endpoints.
- ✅ **Live IBKR Flex Web Service client** (`flex_query/client.py`): real
  two-step SendRequest → GetStatement flow with backoff, unit-tested with an
  injected opener. 🔌 Requires a real **Flex Query token + query id** to fetch
  live statements.
- ✅ FX providers (`fx/providers/rbi.py`, `sbi_tt.py`): conversion engine,
  strategy, caching, RBI reference + SBI TT providers. 🔌 The RBI/SBI **HTTP
  transports** are injectable and safe-by-default (no network without a supplied
  transport); a live fetch transport is the remaining wiring.
- ✅ Celery skeleton present for long-running historical imports.

## Milestone 3 — Auth, RBAC, multi-tenant ✅
- ✅ Registration / email verification / login / password reset / **MFA (TOTP)**.
- ✅ JWT (HS256) bearer auth; RBAC roles investor / tax_consultant / admin;
  `require_role` guards; server-side enforcement; ownership checks on workflow
  runs.
- ✅ Broker-token encryption at rest (Fernet; KMS envelope intended for prod).
- ✅ Server-side **rotatable refresh tokens** (hashed, reuse detection) and an
  **email service** abstraction (console default; SMTP optional).
- ✅ Admin API + web admin portal: user management, tax-rule listing + versioned
  rules + reseed, audit review, system health.
- 🔌 OAuth social login (e.g. Google) remains optional/future.

## Milestone 4 — Documents + dividends + Schedule FA ✅
- ✅ **`tax-reports`**: CSV / HTML / XLSX document generation with embedded
  disclaimer and full provenance (rule_set_id, FX source refs). Demo endpoints
  `GET /api/v1/documents/demo/capital-gains.{html,csv,xlsx}`.
- ✅ **Dividends + Foreign Tax Credit** (`tax-core/dividends/`): gross /
  withholding / net + creditable FTC; `GET /api/v1/schedules/demo/dividends`.
- ✅ **Schedule FA** helper (`tax-core/schedulefa/`);
  `GET /api/v1/schedules/demo/schedule-fa`.
- ⏳ Persisted (per-calc-run) report generation endpoint currently returns 501;
  demo generators are complete.

## Milestone 5 — Grounded AI assistant ✅ (live LLM 🔌)
- ✅ **`tax-assistant`**: deterministic keyword retrieval over the user's own
  `CapitalGainsResult`, `TemplateExplainer` (no hallucination, citations are
  record IDs), and an `LLMExplainer` protocol.
- ✅ API `AnthropicExplainer` (Claude, default model `claude-sonnet-4-6`) with
  automatic fallback to the deterministic explainer;
  `POST /api/v1/assistant/explain`, `/demo/explain`, `GET /assistant/health`.
- 🔌 Live Claude answers require **`ANTHROPIC_API_KEY`** (without it the
  deterministic `TemplateExplainer` is used).

## Milestone 6 — Production infra (partly done)
- ✅ **CI/CD** (GitHub Actions): Python tests with **85% tax-core coverage
  floor**, web typecheck (required), Docker build validation, CodeQL security
  scanning, Dependabot.
- ✅ **Observability**: structured JSON logging, `X-Request-ID` middleware,
  `/health`, `/health/ready`, `/metrics`, Sentry-ready hook.
- ✅ **Reference IaC**: Terraform modules for AWS (ap-south-1) + Cloudflare —
  network, RDS Postgres, ElastiCache Redis, S3 (SSE-KMS), ECS/Fargate + ALB +
  autoscaling, ACM/DNS, Secrets Manager. Out-of-band secret values.
- 🔌 / ⏳ **Remaining to go live** (see below).

## What remains — credential / account-gated

These are coded or referenced but require external accounts/secrets to activate:

1. **Live IBKR**: a real Flex Query **token + query id** (Flex Web Service), and
   end-to-end **Client Portal OAuth** wiring (linking + refresh-token rotation).
2. **Live FX fetch**: concrete **RBI reference** and **SBI TT** HTTP transports
   plugged into the providers, with caching into `fx_rate`.
3. **Live LLM**: **`ANTHROPIC_API_KEY`** for Claude-backed explanations.
4. **Production cloud apply**: `terraform apply` with **AWS + Cloudflare
   credentials**, a **custom domain**, and **DNS/SSL** issuance; Vercel for the
   web app.
5. **Email provider credentials**: SMTP/transactional-email settings to switch
   `EMAIL_BACKEND` from `console` (dev default) to `smtp` for real verification /
   password-reset delivery.
6. **Production secrets**: rotate `SECRET_KEY`, set up a KMS for envelope
   encryption of broker tokens, configure `SENTRY_DSN`.

## Cross-cutting (ongoing)
- Yearly tax-rule updates via config (Budget cycle); append-only versioning.
- Security hardening + DPDP Act 2023 compliance.
- Expand test coverage (currently **~171 tests**) further across API + web.
- **CA verification remains mandatory** for all generated outputs.
