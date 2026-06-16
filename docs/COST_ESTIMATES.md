# Cost Estimates — IBKR Indian Tax Assistant

Indicative monthly infrastructure costs (USD). Tax filing in India is seasonal
(peak Jun–Sep around the July ITR deadline), so the architecture should scale
elastically and scale to near-zero off-season.

## Early stage (MVP, < 1k users)

| Component | Service | Est. /mo |
|---|---|---|
| Frontend hosting | Vercel (Pro) | $20 |
| Backend (API + worker) | Fly.io / Railway small instances | $25–50 |
| Managed PostgreSQL | Neon / Supabase / RDS small | $20–40 |
| Redis | Upstash (pay-as-you-go) | $5–15 |
| Object storage | AWS S3 (statements + PDFs) | $5 |
| DNS / CDN / WAF | Cloudflare (Free/Pro) | $0–20 |
| Error tracking | Sentry (Team) | $0–26 |
| Email (verification, alerts) | Resend / SES | $0–10 |
| AI assistant | Claude API (usage-based) | $20–100 |
| **Total** | | **~$120–290** |

## Growth stage (10k–50k users, seasonal peaks)

| Component | Notes | Est. /mo |
|---|---|---|
| Frontend | Vercel + bandwidth | $20–150 |
| Backend | AWS ECS/Fargate autoscaling (2–8 tasks) | $150–600 |
| PostgreSQL | RDS (Multi-AZ) + read replica | $200–500 |
| Redis | ElastiCache | $50–150 |
| S3 + data transfer | growing document store | $20–80 |
| Cloudflare | Pro/Business | $20–200 |
| Observability | Sentry + metrics/logs | $50–200 |
| AI assistant | Claude API at scale (cache + batch) | $200–1,500 |
| **Total** | | **~$700–3,400** |

## Cost-control levers
- **Seasonality:** scale workers/DB down off-peak; serverless DB (Neon) sleeps when idle.
- **AI cost:** ground responses on stored `CalculationRecord`s (RAG), cache explanations, batch where possible, use the smallest sufficient model for routine explanations.
- **FX rates:** cache RBI/SBI rates in `fx_rate` — fetch once, reuse across users.
- **Egress:** Cloudflare in front of S3/API to cut transfer costs.

## Unit economics (illustrative)
At Investor tier ₹1,499/FY (~$18) and ~$2–6 infra+AI cost per active filing
user/season, gross margin is healthy (>80%) before support/marketing. See
`PRICING_GTM.md` for pricing and go-to-market.

> Figures are planning estimates only; validate against live provider pricing.
