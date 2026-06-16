# API Reference — IBKR Indian Tax Assistant

> **Reference assistance only.** Every computation response carries the
> CA-verification disclaimer; outputs are estimates and **must be verified by a
> certified CA in India** before filing.

Concise catalogue of the FastAPI surface. Derived from
`apps/api/src/ibkr_tax_api/main.py` and `routers/*.py`. Interactive docs are
served at `/docs` (Swagger) and `/redoc`.

**Auth column:** No = public · JWT = `Authorization: Bearer <token>` required ·
JWT+admin = bearer token with `admin` role (`require_role`). Tokens are HS256
JWTs obtained from `POST /api/v1/auth/login`.

**Disclaimer column:** ✓ = the response body includes the CA-verification
disclaimer field.

## Root & observability (`main.py`, `routers/health.py`)

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| GET | `/` | No | App name, version, disclaimer | ✓ |
| GET | `/health` | No | Liveness check (no DB touch) | — |
| GET | `/health/ready` | No | Readiness: rule-config load + DB connectivity (always HTTP 200) | — |
| GET | `/metrics` | No | Prometheus text-format metrics (uptime, in-process counters) | — |

Middleware: CORS allowlist, `RequestIdMiddleware` (`X-Request-ID` propagation +
JSON access logs). Sentry initialized only if `SENTRY_DSN` is set.

## Auth (`routers/auth.py`, prefix `/api/v1/auth`)

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| POST | `/api/v1/auth/register` | No | Register an unverified user; returns a (dev) verification token | — |
| POST | `/api/v1/auth/verify-email` | No | Mark email verified from a valid token | — |
| POST | `/api/v1/auth/login` | No | Authenticate; returns JWT access token | — |
| POST | `/api/v1/auth/password-reset/request` | No | Begin password reset (always 200; returns dev reset token) | — |
| POST | `/api/v1/auth/password-reset/confirm` | No | Complete password reset with a valid token | — |
| POST | `/api/v1/auth/mfa/enable` | JWT | Begin MFA enrollment; returns `otpauth://` provisioning URI | — |
| POST | `/api/v1/auth/mfa/verify` | JWT | Confirm TOTP code to complete MFA | — |

JWT claims: `sub` (user id), `role`, `type="access"`, `iat`, `exp` (60 min
default). Verification/reset tokens are purpose-scoped JWTs. Auth actions are
written to `audit_log`.

## Admin (`routers/admin.py`, prefix `/api/v1/admin`)

All endpoints require **JWT + admin**.

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| GET | `/api/v1/admin/users` | JWT+admin | List all users | — |
| GET | `/api/v1/admin/rules` | JWT+admin | List tax-rule sets in DB | — |
| GET | `/api/v1/admin/audit` | JWT+admin | Recent audit-log entries (`limit`, default 200) | — |
| GET | `/api/v1/admin/health` | JWT+admin | DB connectivity + row counts | — |
| POST | `/api/v1/admin/rules/reseed` | JWT+admin | Idempotently (re)seed rules from YAML; reports inserted/updated/unchanged | — |
| GET | `/api/v1/admin/rules/{asset_class}/{residency}` | JWT+admin | Versioned rule sets for an (asset_class, residency) pair | ✓ |

## Rules (`routers/rules.py`, prefix `/api/v1/rules`)

In-memory rule repository (no DB).

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| GET | `/api/v1/rules` | No | List all loaded tax-rule versions | — |
| GET | `/api/v1/rules/resolve` | No | Resolve the rule in force on `as_of` for `asset_class` + `residency` (404 if none) | — |

## Imports (`routers/imports.py`, prefix `/api/v1/imports`)

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| POST | `/api/v1/imports/sample` | No | Run the built-in sample statement; return normalized trade/corp-action counts | — |

## Calculations (`routers/calculations.py`, prefix `/api/v1/calculations`)

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| POST | `/api/v1/calculations/demo` | No | Run the sample pipeline end-to-end → `CapitalGainsResult` | ✓ |

## Questionnaire (`routers/questionnaire.py`, prefix `/api/v1/questionnaire`)

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| GET | `/api/v1/questionnaire` | No | Current tax-context questionnaire schema | — |

## Documents (`routers/documents.py`, prefix `/api/v1/documents`)

Demo generators run the sample pipeline through `tax-reports`; each rendered
document embeds the disclaimer and full FX/rule provenance.

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| GET | `/api/v1/documents/demo/capital-gains.html` | No | HTML capital-gains report | ✓ (in document) |
| GET | `/api/v1/documents/demo/capital-gains.csv` | No | CSV capital-gains schedule | ✓ (in document) |
| GET | `/api/v1/documents/demo/capital-gains.xlsx` | No | XLSX report | ✓ (in document) |
| POST | `/api/v1/documents/{calc_run_id}/capital-gains-report` | JWT (planned) | Persistence-backed report — **returns HTTP 501 (not yet implemented)** | — |

## Assistant (`routers/assistant.py`, prefix `/api/v1/assistant`)

Grounded strictly on the user's own computed records; citations are record IDs.

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| POST | `/api/v1/assistant/demo/explain` | No | Deterministic explanation (`TemplateExplainer`, no LLM) | ✓ |
| POST | `/api/v1/assistant/explain` | No | Grounded explanation via `AnthropicExplainer` (Claude) if `ANTHROPIC_API_KEY` set, else falls back to `TemplateExplainer` | ✓ |
| GET | `/api/v1/assistant/health` | No | Whether an LLM-backed explainer is configured (no secrets revealed) | — |

LLM: `AnthropicExplainer`, default model `claude-sonnet-4-6` (configurable via
`ASSISTANT_MODEL`), lazy client init, robust fallback on any error.

## Workflow (`routers/workflow.py`, prefix `/api/v1/workflow`)

Authenticated, persisted pipeline tied to the current user; ownership enforced.

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| POST | `/api/v1/workflow/run` | JWT | Run the sample pipeline and persist under the current user (one transaction) | ✓ |
| GET | `/api/v1/workflow/runs` | JWT | List the current user's persisted calc runs (most recent first) | — |
| GET | `/api/v1/workflow/runs/{run_id}` | JWT | One run with records + assumptions (404 if not owned) | ✓ |

## Schedules (`routers/schedules.py`, prefix `/api/v1/schedules`)

Deterministic demo endpoints (no DB).

| Method | Path | Auth | Description | Disclaimer |
|---|---|---|---|---|
| GET | `/api/v1/schedules/demo/schedule-fa` | No | Schedule FA rows from sample trades (`period_start`, `period_end`) | ✓ |
| GET | `/api/v1/schedules/demo/dividends` | No | Foreign-dividend / FTC computation (`financial_year`) | ✓ |

## Conventions

- **Decimal as string.** All monetary/quantity values serialize as strings to
  preserve precision (never floats).
- **Disclaimer.** Embedded in the OpenAPI description and returned in every
  computation/result payload.
- **Errors.** Standard FastAPI/Pydantic validation (422); 401/403 for
  auth/RBAC; 404 for not-found / not-owned; 501 for the not-yet-implemented
  persisted report endpoint.
