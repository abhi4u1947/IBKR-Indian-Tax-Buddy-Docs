# System Design — IBKR Indian Tax Assistant

Companion to `ARCHITECTURE.md`; component-level detail.

> **Reference assistance only.** Outputs are estimates and **must be verified by
> a certified CA in India** before filing.

## Components

### Web (Next.js)
- Marketing site (SSR/ISR for SEO), with pages: home, pricing, features,
  how-it-works, faq, blog (+ dynamic posts), about, contact, security, privacy,
  terms, disclaimer.
- Authenticated app (route groups): `login` / `register`, `dashboard`,
  `onboarding`, `results`, `assistant`, `schedule-fa`.
- Admin group: `admin` dashboard, `admin/users`, `admin/rules`, `admin/health`,
  `admin/audit`.
- Typed API client → FastAPI (shares `packages/ts-shared` types).
- SEO: metadata, OpenGraph/Twitter cards, JSON-LD (organization + software app),
  `sitemap`/`robots`; analytics toggled by env. **INR pricing** with GST.
- Tested with Vitest (**30 tests**: api-client, pricing, utils, assistant,
  disclaimer).

### API (FastAPI)
- Routers: `health`, `auth`, `admin`, `rules`, `imports`, `calculations`,
  `questionnaire`, `documents`, `assistant`, `workflow`, `schedules`. Pydantic v2
  models; **Decimal serialized as string**.
- Auth: JWT (HS256) bearer tokens; RBAC via `require_role`; register / verify
  email / login / password-reset / **MFA (TOTP)**.
- DI provides concrete `RuleRepository` and `FxRateProvider` to the pure
  `tax-core` engines; persistence service ties ingest → persist → calculate →
  persist for authenticated users.
- LLM assistant: `AnthropicExplainer` (Claude, default `claude-sonnet-4-6`) when
  `ANTHROPIC_API_KEY` is set, with automatic fallback to the deterministic
  `TemplateExplainer`.
- OpenAPI auto-generated; disclaimer embedded in the spec + every result payload.
- Tested with pytest (**56 tests**). Full endpoint catalogue in
  `API_REFERENCE.md`.

### Workers (Celery)
- Skeleton present (Redis broker) for long-running historical imports and
  document generation. Example task chains ingest → cost-basis → calculator →
  persistence.

### Engines (`tax-core`, `ibkr-ingest`, `tax-reports`, `tax-assistant`)
- **tax-core** — pure deterministic engines: rules, cost basis (FIFO/SpecID),
  corporate-action replay (`process_events`), FX engine + RBI/SBI/sample
  providers, capital-gains calculator, dividends + FTC, Schedule FA. 60 tests.
- **ibkr-ingest** — Flex Query XML parser, **live `FlexWebServiceClient`**
  (real two-step SendRequest → GetStatement flow over stdlib `urllib`, with an
  injectable opener for tests), sample provider. 14 tests.
- **tax-reports** — schedule builders + CSV/HTML/XLSX renderers with embedded
  disclaimer and full provenance (rule_set_id, FX source refs). 5 tests.
- **tax-assistant** — deterministic keyword retrieval over a user's
  `CapitalGainsResult`, `TemplateExplainer` (no hallucination), and an
  `LLMExplainer` protocol implemented by the API's `AnthropicExplainer`. 6 tests.

### Data
- PostgreSQL (integrity + audit), Redis (cache + queue), S3 (documents).

## Key design decisions

| Decision | Rationale |
|---|---|
| Engines as pure packages behind `Protocol`s | Testable in isolation; reused by API + workers; storage-agnostic |
| Rules as versioned data, resolved by sale date | No yearly code changes; correct historical filings; auditable |
| `Decimal` end-to-end, `float` rejected | Tax-grade precision; no rounding drift |
| Explainable `CalculationRecord`s | Audit reports + grounded assistant (no hallucinations) |
| Assumptions as first-class flags | Surface interpretation to user + mandatory CA verification |
| Live transports injectable + safe-by-default | Network code is unit-tested with fakes; activates only with real credentials/keys |
| Flex Query first for IBKR | Best source of historical/tax-lot data; OAuth Client Portal for linking |
| LLM grounded only on user records | No hallucination; deterministic fallback always available |

## API surface (summary)

Public/demo (no auth): `GET /`, `GET /health`, `GET /health/ready`,
`GET /metrics`, `GET /api/v1/rules`, `GET /api/v1/rules/resolve`,
`POST /api/v1/imports/sample`, `POST /api/v1/calculations/demo`,
`GET /api/v1/questionnaire`, `GET /api/v1/documents/demo/*`,
`POST /api/v1/assistant/demo/explain`, `POST /api/v1/assistant/explain`,
`GET /api/v1/assistant/health`, `GET /api/v1/schedules/demo/*`.

Auth (JWT): `POST /api/v1/auth/*` (register/verify/login/password-reset/mfa),
`POST/GET /api/v1/workflow/*` (persisted run + list + detail).

Admin (JWT + `admin` role): `GET /api/v1/admin/users|rules|audit|health`,
`POST /api/v1/admin/rules/reseed`, `GET /api/v1/admin/rules/{asset}/{residency}`.

See `API_REFERENCE.md` for the full per-endpoint catalogue (method, path, auth,
disclaimer).

## Observability
- Structured single-line JSON logging (no PII/secrets) with `request_id`
  correlation via `X-Request-ID` middleware.
- Health endpoints `GET /health` (liveness) and `GET /health/ready`
  (readiness: rule load + DB connectivity).
- `GET /metrics` — hand-rolled Prometheus text exposition (uptime, in-process
  counters); no extra dependency required.
- Sentry-ready: `init_sentry()` is a no-op unless `SENTRY_DSN` is configured.

## Testing strategy
- Engine unit tests (in-memory fakes; exact `Decimal` assertions); network
  transports tested with injected openers/transports (no real network).
- API tests via TestClient (most run without a DB; persisted-workflow tests use
  a session).
- Web tests via Vitest.
- CI enforces an **85% coverage floor on tax-core** (the trust anchor); web
  typecheck is a required gate.
