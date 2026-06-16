# Security Review — IBKR Indian Tax Assistant

This document is a security design + review checklist. The product handles highly
sensitive data: brokerage credentials/tokens, full transaction history, and PII.

## Threat model (summary)

| Asset | Threat | Mitigation |
|---|---|---|
| IBKR access/refresh tokens | Theft → account read access | Encrypt at rest (envelope encryption via KMS/secrets manager); never log; short-lived access tokens with **refresh-token rotation**; scope to read-only Flex where possible |
| Tax documents / statements | Exfiltration | S3 with SSE-KMS, per-object ACLs, signed time-limited URLs, no public buckets |
| PII (PAN, residency, income) | Leak / unauthorised access | Encryption in transit (TLS 1.2+) and at rest; RBAC; field-level access logging |
| User auth | Account takeover | Argon2/bcrypt password hashing, MFA/2FA (TOTP), email verification, rate limiting, secure session cookies |
| Tax-rule data | Tampering → wrong tax | Append-only versioned rules + `tax_rule_audit`; admin-only writes; config hash integrity |

## Controls

### Authentication & authorization (implemented + tested)
- **Implemented in the API** (`apps/api/src/ibkr_tax_api/routers/auth.py`,
  `security/`): registration, **email verification**, login, **password reset**,
  and **MFA/2FA (TOTP via `pyotp`)**, plus **refresh-token rotation** with
  reuse detection. Passwords are hashed with **bcrypt** (SHA-256 pre-hash to
  avoid bcrypt's 72-byte limit); the API **never stores plaintext passwords**,
  and refresh tokens are stored only as SHA-256 hashes.
- **JWT (HS256) bearer tokens** signed with `SECRET_KEY`; claims `sub`, `role`,
  `type`, `iat`, `exp` (60 min default). Verification/reset tokens are
  purpose-scoped JWTs.
- **Refresh tokens** are persisted server-side in `refresh_token` as a
  **SHA-256 hash** (raw value never stored), with **rotation on every use** and
  **reuse detection** (`rotated_from` chain → revoke the chain). Configurable TTL
  (`REFRESH_TOKEN_TTL_DAYS`, default 30).
- **Email** delivery is behind an interface (`services/email.py`): default
  `console` backend (logs, no network — safe for tests/dev) with an optional
  `smtp` backend enabled via `EMAIL_BACKEND=smtp` + SMTP settings.
- **RBAC** roles: `investor`, `tax_consultant`, `admin`, enforced server-side via
  `require_role` (admin endpoints) and per-user ownership checks on workflow
  runs — no trusting client role claims.
- Auth-sensitive actions are recorded in `audit_log`.
- 🔌 **Planned/optional:** OAuth social login (e.g. Google); real transactional
  email (verification/reset tokens are returned directly in dev today).

### Encryption
- **In transit:** automatic HTTPS (Cloudflare + platform TLS), HSTS.
- **At rest:** managed Postgres encryption; S3 SSE-KMS; **application-level
  encryption of broker tokens** — implemented today with **Fernet**
  (AES-128-CBC + HMAC) keyed from `SECRET_KEY`
  (`apps/api/src/ibkr_tax_api/security/crypto.py`); `token_encryption_key_id`
  stamps the key version for rotation. 🔌 **Production intent:** envelope
  encryption with a KMS-held key.
- **Secrets:** a secrets manager (AWS Secrets Manager / SSM / Doppler) — never in
  repo or images. `.env` is git-ignored. The reference Terraform provisions
  Secrets Manager entries with out-of-band values.

### IBKR credential handling
- Prefer **Flex Query token** (read-only statement export) over full trading
  credentials. The live `FlexWebServiceClient` is implemented; a real token +
  query id is required to fetch statements.
- Client Portal OAuth tokens are stored as **ciphertext** with
  refresh-token-rotation metadata (`refresh_token_expires_at`, `last_rotated_at`,
  `rotation_count`) on `broker_connection`. 🔌 End-to-end OAuth linking is the
  remaining wiring.
- The app **never stores IBKR username/password**.

### Application security
- Input validation via Pydantic v2; parameterised queries via SQLAlchemy (no string SQL).
- CORS allowlist; CSRF protection on cookie-based flows; security headers (CSP, X-Frame-Options, Referrer-Policy).
- Rate limiting + bot protection at the edge (Cloudflare).
- Dependency scanning (pip-audit / npm audit) and secret scanning in CI.

### Audit & retention
- Append-only `audit_log` for sensitive actions (linking accounts, generating reports, rule changes).
- **Data retention policy:** raw statements and generated documents retained per a configurable window; user-initiated export + deletion (DPDP Act 2023 alignment — data principal rights).
- Backups encrypted; tested restore.

### Observability tie-in
- Structured JSON logging (no secrets/PII in logs) with `X-Request-ID`
  correlation, Sentry-ready error tracking, `/metrics`, and `/health` +
  `/health/ready` checks — all implemented. See `SYSTEM_DESIGN.md` and
  `API_REFERENCE.md`.

## Compliance posture
- **India DPDP Act 2023**: consent, purpose limitation, data-principal rights, breach notification readiness.
- **Disclaimer enforced in-product**: outputs are estimates and **must be verified by a certified CA in India**; the product is reference assistance, not tax advice or a substitute for professional judgement.

## Pre-production checklist
- [ ] Secrets in a managed secrets store; rotation policy set
- [ ] All money paths use Decimal (verified by tests)
- [ ] Broker tokens encrypted + rotation tested
- [ ] MFA enforced for tax_consultant/admin
- [ ] RBAC tests for cross-tenant access
- [ ] Pen-test / dependency + secret scans clean
- [ ] Backup/restore drill done
- [ ] Privacy policy, terms, and CA-verification disclaimer published
