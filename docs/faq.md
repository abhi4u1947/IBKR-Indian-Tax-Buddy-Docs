---
title: FAQ
layout: default
nav_order: 4
---

# Frequently asked questions
{: .no_toc }

1. TOC
{:toc}

---

## Which statement should I export from IBKR?

A **Flex Query** in CSV is preferred because you control exactly which sections
are included and it parses reliably. If you only have the standard **Activity
Statement**, export it in CSV for the same period. See
[Getting started](getting-started) for the sections to include.

## What date range should the statement cover?

It depends on the figure:

- **Capital gains and dividends** follow the Indian **financial year**
  (1 April – 31 March).
- **Schedule FA** follows the **calendar year** (1 January – 31 December).

Exporting a single wide range that covers both windows is the simplest
approach; the tool selects the right transactions for each computation.

## How are USD amounts converted to INR?

Different income types use different reference rates and dates — capital gains
convert cost and proceeds at the rates on their respective dates, dividends at
the receipt date, and Schedule FA balances at the prescribed SBI TT buying
rates. See [Currency conversion](concepts#currency-conversion) for details.
Because cost and proceeds convert at different dates, your **INR gain can differ
from your USD gain**.

## How is the US tax withheld on dividends treated?

The US typically withholds tax on dividends paid to Indian residents (commonly
25% under the India–US treaty when your W-8BEN is on file). You can usually
claim a **foreign tax credit** for this against your Indian tax on the same
income, which generally requires filing **Form 67** before your return. The tool
reports both the gross dividend and the tax withheld.

## Do I have to file Schedule FA even if I made no profit?

If you are a **Resident and Ordinarily Resident**, yes — Schedule FA is a
**disclosure** requirement that applies to holding the foreign asset, regardless
of whether any income arose. Non-disclosure carries significant penalties under
the Black Money Act.

## The tool's numbers don't match my own calculation. What now?

Differences usually come from:

- **Currency rates** — which rate source and which date were used.
- **Holding-period classification** — the 24-month short/long-term split for
  foreign equity.
- **Lot matching** — which purchase lots a sale was matched against.
- **Corporate actions** — splits, mergers, or symbol changes in the statement.

Treat the tool's output as a **prepared draft** to review, not a final number.
Reconcile against your own records and, where the amounts are material, with a
tax professional.

## Is this tool tax advice?

No. IBKR Indian Tax Buddy helps you **prepare** figures from your statements. It
is not tax advice and not a substitute for a qualified professional. Rates and
rules change with each Finance Act — verify everything before filing.

## Where do I report bugs or ask about features?

Open an issue on the
[project repository](https://github.com/abhi4u1947/IBKR-Indian-Tax-Buddy).
