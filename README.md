# IBKR Indian Tax Assistant — Documentation

The published documentation website for the **IBKR Indian Tax Assistant**.

🌐 **Live site:** https://abhi4u1947.github.io/IBKR-Indian-Tax-Buddy-Docs/

> ⚠️ **Reference assistance only.** All tax computations are **estimates** and
> **must be reviewed and verified by a certified Chartered Accountant (CA) in
> India before filing.** Not tax, legal, or financial advice.

## What's here

This repository holds the documentation site (MkDocs Material) and UI
screenshots for the IBKR Indian Tax Assistant. It is **generated** from the
`docs/` tree of the application repository and pushed here automatically — do
not hand-edit; change the docs in the application repo instead.

| Path | Contents |
|---|---|
| `docs/` | Architecture, system design, database/ER, API reference, tax rules, security, go-live runbook, pricing/GTM, roadmap, changelog, disclaimer |
| `docs/assets/screenshots/` | UI screenshots (landing, pricing, dashboard, results, assistant, Schedule FA, admin) |
| `mkdocs.yml` | Site configuration |
| `.github/workflows/pages.yml` | Builds + deploys to GitHub Pages on every push to `main` |

## Build locally

```bash
pip install -r requirements.txt
mkdocs serve   # http://127.0.0.1:8000
```
