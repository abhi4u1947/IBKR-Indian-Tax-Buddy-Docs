# Go-Live Runbook

A precise, copy-pasteable runbook to take the **IBKR Indian Tax Assistant** from
the repo to a live, public deployment.

!!! danger "Reference assistance only — CA verification required"
    Going live does **not** make this a tax-advice service. Every computation
    response carries the CA-verification disclaimer, and all outputs **must be
    verified by a certified Chartered Accountant (CA) in India before filing.**

!!! info "How to read this runbook"
    - Steps are **numbered** and ordered. Run them top-to-bottom for a clean
      first deploy.
    - Anything marked **:material-key: EXTERNAL** requires an account, secret, or
      credential **you must supply** — it is not in the repo.
    - Commands are copy-pasteable. Replace `<...>` placeholders.
    - The architecture is **AWS (ap-south-1 / Mumbai) + Cloudflare + Vercel**,
      matching the reference Terraform in `infra/`. Local dev uses Docker Compose.

---

## 1. Prerequisites & accounts

Install locally:

- **Docker + Docker Compose** (local smoke test)
- **Python 3.11+**, **Node 18+**, **pnpm** (engine work / web build)
- **Terraform >= 1.6** (cloud infra)
- **AWS CLI v2** (secrets + ECS)
- **git**, **make**, **openssl**

Accounts / credentials to create (**:material-key: EXTERNAL** — all of these are
things you provide):

| # | Dependency | Why | Notes |
|---|---|---|---|
| 1 | **AWS account** | ECS/Fargate, RDS, ElastiCache, S3, KMS, ACM, Secrets Manager | All in `ap-south-1` for India data residency |
| 2 | **Cloudflare account + a domain** | DNS + SSL + ACM DNS validation | You need the **zone id** and an **API token** scoped to the zone |
| 3 | **Vercel account** | Hosts the Next.js web app (`apps/web`) | Not provisioned by Terraform |
| 4 | **Anthropic API key** | Live Claude-backed assistant answers | Optional — falls back to deterministic explainer |
| 5 | **IBKR Flex Web Service** | Live statement fetch | Needs a **Flex token** + **query id** |
| 6 | **SMTP / email provider** | Verification + password-reset email | e.g. SES, Postmark, Mailgun, SendGrid |
| 7 | **Container registry** | Hold the API/worker image | GHCR / ECR — must be pullable by ECS |
| 8 | **Sentry** (optional) | Error tracking | Provides `SENTRY_DSN` |

---

## 2. Local smoke test

Validate the full stack locally before touching the cloud.

```bash
git clone https://github.com/abhi4u1947/IBKR-Indian-Tax-Buddy.git
cd IBKR-Indian-Tax-Buddy

cp .env.example .env          # edit values as needed (local defaults work)
make up                       # builds + starts postgres, redis, api, worker, web
make seed                     # load versioned tax rules into the DB
```

Verify:

```bash
curl -s http://localhost:8000/health            # liveness (no DB)
curl -s http://localhost:8000/health/ready      # rule-config + DB connectivity
curl -s http://localhost:8000/metrics | head    # Prometheus exposition
open http://localhost:8000/docs                 # interactive OpenAPI docs
open http://localhost:3000                       # web app
```

Run the engine test suites (no Docker needed):

```bash
make install-core             # editable installs of the four Python packages
make test                     # tax-core (coverage) + ingest + reports + assistant
```

Tear down when done:

```bash
make down
```

---

## 3. Backend secrets & environment variables

Every backend setting comes from `apps/api/src/ibkr_tax_api/settings.py`
(pydantic-settings, read from the environment / `.env`). Below is the **complete
list**, what each is, and how to generate it. In production these are stored in
**AWS Secrets Manager** (see §4), not in a file.

### Core

| Variable | Default | What it is / how to set |
|---|---|---|
| `ENVIRONMENT` | `local` | Set to `prod`. |
| `SECRET_KEY` | `dev-insecure-change-me` | JWT / signing key. **Generate:** `openssl rand -hex 32`. **:material-key: must be set in prod.** |

### Datastores

| Variable | Default | What it is / how to set |
|---|---|---|
| `DATABASE_URL` | `postgresql+psycopg2://postgres:postgres@localhost:5432/ibkr_tax` | SQLAlchemy URL. In prod the **`database` Terraform module** generates the master creds and a ready-to-use URL in its own secret, injected into the tasks (see `infra/README.md`). |
| `REDIS_URL` | `redis://localhost:6379/0` | Cache. In prod use the ElastiCache `rediss://...` output. |
| `CELERY_BROKER_URL` | `redis://localhost:6379/1` | Celery broker. Point at ElastiCache in prod. |

### CORS & config paths

| Variable | Default | What it is / how to set |
|---|---|---|
| `CORS_ORIGINS` | `http://localhost:3000,http://localhost:5173` | Comma-separated allowed origins. Set to your web domain, e.g. `https://taxassistant.<yourdomain>`. |
| `TAX_RULES_PATH` | `config/tax_rules/india_residents.yaml` | Versioned tax-rule file. Defaults are correct; override only to relocate. |
| `FX_STRATEGY_PATH` | `config/tax_rules/fx_strategy.yaml` | FX strategy file. As above. |

### AI assistant (optional)

| Variable | Default | What it is / how to set |
|---|---|---|
| `ANTHROPIC_API_KEY` | `""` | **:material-key: EXTERNAL.** When empty, the deterministic `TemplateExplainer` is used (no key required). Set it to enable the live Claude-backed `AnthropicExplainer`. |
| `ASSISTANT_MODEL` | `claude-sonnet-4-6` | Claude model id used by the assistant. |

### Token TTLs

| Variable | Default | What it is / how to set |
|---|---|---|
| `ACCESS_TOKEN_TTL_MINUTES` | `30` | Access-token lifetime (short). |
| `REFRESH_TOKEN_TTL_DAYS` | `30` | Refresh-token lifetime (long; rotated on use, stored hashed). |

### Email

| Variable | Default | What it is / how to set |
|---|---|---|
| `APP_BASE_URL` | `http://localhost:3000` | Base URL the frontend serves verification / reset links from. Set to your web domain. |
| `EMAIL_BACKEND` | `console` | `console` logs the email (no network); set to `smtp` to actually send. |
| `SMTP_HOST` | `""` | **:material-key: EXTERNAL.** SMTP host (only used when `EMAIL_BACKEND=smtp`). |
| `SMTP_PORT` | `587` | SMTP port. |
| `SMTP_USER` | `""` | SMTP username. |
| `SMTP_PASSWORD` | `""` | **:material-key: EXTERNAL.** SMTP password. |
| `SMTP_FROM` | `no-reply@ibkr-tax.local` | From address. Use a verified sender on your domain. |
| `SMTP_USE_TLS` | `true` | STARTTLS toggle. |

### Observability

| Variable | Default | What it is / how to set |
|---|---|---|
| `LOG_LEVEL` | `INFO` | Stdlib JSON log level. |
| `SENTRY_DSN` | `""` | **:material-key: EXTERNAL (optional).** Set to enable Sentry; no-op when empty. |

### IBKR Flex (live ingest) — :material-key: EXTERNAL

Live statement fetch uses the **IBKR Flex Web Service**. Supply your Flex
**token** and **query id** to the ingest layer (e.g. as
`IBKR_FLEX_TOKEN` / `IBKR_FLEX_QUERY_ID` env values consumed by the live Flex
client). Without them the parser/sample provider still works; only live fetch is
gated.

!!! note
    Generate `SECRET_KEY` and any other random secret with
    `openssl rand -hex 32`. Never commit real secret values — they go into
    Secrets Manager (§4), not `.env` or `*.tfvars`.

---

## 4. Cloud infrastructure (Terraform)

The reference IaC in `infra/` provisions the AWS topology (VPC, ECS/Fargate api
+ worker, RDS PostgreSQL, ElastiCache Redis, S3, KMS, ACM, Secrets Manager) and
Cloudflare DNS validation, all in **ap-south-1**.

### 4.1 Build & push the API/worker image first

The `compute` module needs a pullable `container_image`. Build from
`apps/api/Dockerfile` (build context is the repo root):

```bash
# Example with GitHub Container Registry (GHCR)
docker build -f apps/api/Dockerfile -t ghcr.io/abhi4u1947/ibkr-tax-api:latest .
echo "$GHCR_TOKEN" | docker login ghcr.io -u abhi4u1947 --password-stdin
docker push ghcr.io/abhi4u1947/ibkr-tax-api:latest
```

> The repo already ships a `docker.yml` GitHub Actions workflow that can build
> and push this image on `main`.

### 4.2 Configure variables

```bash
cd infra
cp prod.tfvars.example prod.tfvars   # NON-secret config only — do NOT commit
```

Required values to fill in `prod.tfvars` (**:material-key: EXTERNAL** as noted):

| Var | Example | Notes |
|---|---|---|
| `region` | `ap-south-1` | Mumbai (data residency). |
| `environment` | `prod` | Used in resource names / secret ids. |
| `container_image` | `ghcr.io/abhi4u1947/ibkr-tax-api:latest` | From §4.1. |
| `container_port` | `8000` | API listen port. |
| `bucket_suffix` | `<aws-account-id>` | Keeps S3 names globally unique. |
| `domain` | `taxassistant.<yourdomain>` | **:material-key:** Your custom domain. |
| `cloudflare_zone_id` | `0123…` | **:material-key:** Cloudflare zone id. |
| `db_instance_class`, `multi_az`, `redis_node_type`, `api_desired_count`, … | see example | Sizing / HA knobs. |

### 4.3 Apply

```bash
export AWS_PROFILE=<your-aws-profile>      # :material-key: AWS credentials
export CLOUDFLARE_API_TOKEN=<token>        # :material-key: zone-scoped Cloudflare token

terraform init
terraform fmt -recursive
terraform validate
terraform plan  -var-file=prod.tfvars
terraform apply -var-file=prod.tfvars
```

Outputs include the ALB DNS name, DB endpoint and bucket names
(`terraform output`).

### 4.4 Set secret VALUES out-of-band

The `secrets` module creates each secret with a placeholder `CHANGE_ME` and
`ignore_changes` on the value, so Terraform never overwrites real values. After
the first apply, populate them:

```bash
ENV=prod
aws secretsmanager put-secret-value --secret-id "ibkr-tax-$ENV/SECRET_KEY"    --secret-string "$(openssl rand -hex 32)"
aws secretsmanager put-secret-value --secret-id "ibkr-tax-$ENV/IBKR_API_KEY"  --secret-string "<flex-token>"
aws secretsmanager put-secret-value --secret-id "ibkr-tax-$ENV/ANTHROPIC_KEY" --secret-string "<anthropic-key>"
```

- `DATABASE_URL` is wired automatically — the `database` module stores the
  generated master credentials and a ready-to-use `postgresql://` URL in its own
  secret, injected into the tasks.
- `REDIS_URL` can be set from the `cache` module output (`rediss://...`).

Force a new ECS deployment so tasks pick up the values:

```bash
aws ecs update-service --cluster ibkr-tax-$ENV-cluster --service ibkr-tax-$ENV-api    --force-new-deployment
aws ecs update-service --cluster ibkr-tax-$ENV-cluster --service ibkr-tax-$ENV-worker --force-new-deployment
```

---

## 5. Database migrations & seeding

Run as a release step **before** serving traffic. Alembic migrations
(`0001_initial`, `0002_user_mfa_secret`) live under `apps/api/alembic`.

```bash
# From a task/container with DATABASE_URL set (e.g. an ECS one-off task)
cd apps/api
alembic upgrade head
```

Seed the versioned tax rules (idempotent / append-only — safe every deploy):

```bash
python -m ibkr_tax_api.services.seed_rules
# or, against a live deployment as an admin:
#   POST /api/v1/admin/rules/reseed
# locally:
#   make seed
```

---

## 6. Frontend (Vercel)

The web app (`apps/web`) is deployed to **Vercel** (not provisioned by
Terraform).

1. Import the repo into Vercel; set the **root directory** to `apps/web`.
2. Set environment variables (**:material-key:** values you supply):
   - `NEXT_PUBLIC_API_URL` → your live API URL (e.g.
     `https://api.taxassistant.<yourdomain>`).
   - Any other `NEXT_PUBLIC_*` config the app expects.
3. Deploy. Vercel gives a `*.vercel.app` URL with automatic HTTPS.

### Custom domain + SSL (Cloudflare)

1. Add your domain to **Cloudflare**; point the registrar nameservers at it.
2. In Vercel, add `taxassistant.<yourdomain>` → verify → automatic SSL.
3. For the API, create a DNS record in Cloudflare pointing
   `api.taxassistant.<yourdomain>` at the **ALB DNS name** (Terraform output).
   The ACM cert + Cloudflare DNS validation are handled by the `dns` module.
4. Update backend `CORS_ORIGINS` and `APP_BASE_URL` to the web domain, and the
   web app's `NEXT_PUBLIC_API_URL` to the API domain. Redeploy both.

---

## 7. Enabling live integrations

All of these are coded and unit-tested with fakes; they activate with real
secrets.

- **Live LLM assistant** — set `ANTHROPIC_API_KEY` (§3 / §4.4). Empty key →
  deterministic `TemplateExplainer`.
- **Live IBKR Flex** — supply the Flex **token** + **query id** to the ingest
  layer (§3). Without them, only the XML parser / sample provider run.
- **Live FX transports** — plug live **RBI** / **SBI TT** fetch transports into
  the FX providers. The providers are safe-by-default and raise without an
  injected transport, so live FX is opt-in.
- **Transactional email** — set `EMAIL_BACKEND=smtp` plus `SMTP_*` (§3).

---

## 8. Post-deploy verification checklist

```bash
API=https://api.taxassistant.<yourdomain>
WEB=https://taxassistant.<yourdomain>

curl -s $API/health            # 200 liveness
curl -s $API/health/ready      # 200; rule-config loaded + DB reachable
curl -s $API/metrics | head    # Prometheus exposition
open $API/docs                 # OpenAPI docs reachable
open $WEB                       # web app loads over HTTPS
```

Then confirm:

- [ ] `/health/ready` reports the rule config loaded and DB connected.
- [ ] `GET /api/v1/admin/health` shows expected table row counts (rules seeded).
- [ ] A demo calculation (`POST /api/v1/calculations/demo`) returns results
      **with the CA-verification disclaimer present**.
- [ ] Auth flow works end-to-end (register → verify-email → login); verification
      email is delivered (if `EMAIL_BACKEND=smtp`).
- [ ] Web app talks to the API (no CORS errors) and SSL is valid on both domains.
- [ ] Assistant answers (deterministic, or live if `ANTHROPIC_API_KEY` set).
- [ ] Sentry receives a test error (if `SENTRY_DSN` set).
- [ ] Daily DB snapshots are enabled; a test restore has been validated.

---

## 9. Rollback notes

- **Backend** — ECS keeps prior task-definition revisions. Roll back by deploying
  the previous image tag / revision:

  ```bash
  aws ecs update-service --cluster ibkr-tax-prod-cluster \
    --service ibkr-tax-prod-api --task-definition <previous-revision> \
    --force-new-deployment
  ```

- **Secrets** — Secrets Manager retains version history; restore a prior version
  with `aws secretsmanager update-secret-version-stage`, then force a new ECS
  deployment.
- **Frontend** — Vercel keeps every deployment; **Promote** a previous
  deployment to production (instant rollback).
- **Database migrations** — Alembic supports `alembic downgrade <revision>`, but
  **prefer roll-forward**. Always take a snapshot before `alembic upgrade head`;
  restore from snapshot if a migration is unsafe to downgrade.
- **Rule config** — rule seeding is append-only/versioned, so a bad rule change
  is corrected by seeding a new version (history + audit preserved), not by
  deleting data.

---

## 10. Quick reference — what you must supply

| Secret / account | Where it goes | Generate / source |
|---|---|---|
| AWS credentials | `AWS_PROFILE` env for Terraform / CLI | AWS IAM |
| Cloudflare API token + zone id | `CLOUDFLARE_API_TOKEN` env + `cloudflare_zone_id` tfvar | Cloudflare dashboard |
| Custom domain | `domain` tfvar, Vercel, DNS | Your registrar |
| `SECRET_KEY` | Secrets Manager `ibkr-tax-prod/SECRET_KEY` | `openssl rand -hex 32` |
| `ANTHROPIC_API_KEY` | Secrets Manager `ibkr-tax-prod/ANTHROPIC_KEY` | Anthropic console |
| IBKR Flex token + query id | ingest env / Secrets Manager | IBKR Flex Web Service |
| SMTP credentials | `SMTP_*` env / Secrets Manager | Your email provider |
| `SENTRY_DSN` (optional) | env / Secrets Manager | Sentry |
| Container image | `container_image` tfvar | Build from `apps/api/Dockerfile` → GHCR/ECR |
