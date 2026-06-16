---
title: Home
layout: home
nav_order: 1
description: "Documentation for IBKR Indian Tax Buddy."
permalink: /
---

# IBKR Indian Tax Buddy
{: .fs-9 }

Turn your Interactive Brokers (IBKR) account statements into the figures you
need for your Indian income-tax return.
{: .fs-6 .fw-300 }

[Get started](docs/getting-started){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View on GitHub](https://github.com/abhi4u1947/IBKR-Indian-Tax-Buddy){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What it does

If you are an Indian resident investing through Interactive Brokers, filing
your return means pulling apart broker statements that were never designed for
Indian tax rules. IBKR Indian Tax Buddy reads your IBKR activity data and helps
you assemble the numbers you typically need at filing time:

- **Capital gains** on the sale of US shares and ETFs, split into short-term and
  long-term, with proceeds and cost converted to INR.
- **Dividend income** received from foreign securities, along with the US tax
  withheld at source (relevant for your foreign tax credit claim).
- **Foreign Asset (Schedule FA)** disclosure details for the holdings in your
  IBKR account, reported on the calendar-year basis the schedule requires.

The goal is to take the manual, error-prone work of reconciling broker
statements — and the currency conversion that goes with it — and turn it into a
clear, reviewable set of figures.

## How to use these docs

| Guide | What it covers |
|:------|:---------------|
| [Getting started](docs/getting-started) | Exporting the right statement from IBKR and producing your first set of figures. |
| [Concepts](docs/concepts) | How Indian tax rules apply to a foreign brokerage account — capital gains, dividends, and Schedule FA. |
| [FAQ](docs/faq) | Common questions about statements, currency conversion, and edge cases. |

## Disclaimer

IBKR Indian Tax Buddy is a tool to assist with **preparing** tax figures. It is
**not** tax advice and is not a substitute for a qualified tax professional.
Tax rates, thresholds, and disclosure rules change with each Finance Act —
always review the computed figures against your own records and current law, or
with your tax advisor, before filing.
