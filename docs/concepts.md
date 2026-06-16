---
title: Concepts
layout: default
nav_order: 3
---

# Concepts
{: .no_toc }

Background on how Indian tax rules apply to an Interactive Brokers account. This
is general explanatory material to help you read the tool's output — it is not
tax advice, and the exact rates and thresholds change with each Finance Act.

1. TOC
{:toc}

---

## Residency and the two calendars

How you are taxed depends on your **residential status** under the Income-tax
Act. A **Resident and Ordinarily Resident (ROR)** is taxed on worldwide income,
including gains and dividends from a foreign brokerage account, and must make
foreign-asset disclosures. Non-residents and RNOR individuals are treated
differently — confirm your status first.

Two calendars matter, and mixing them up is the most common mistake:

- **Income (capital gains, dividends)** is computed on the Indian **financial
  year**: 1 April – 31 March.
- **Schedule FA (foreign asset disclosure)** is reported on the **calendar
  year**: 1 January – 31 December — referred to in the schedule as the relevant
  "accounting period".

## Currency conversion

Amounts on an IBKR statement are in the trade currency (usually USD) and must be
converted to INR. The reference rate to use depends on the type of income:

- **Capital gains** — the **telegraphic transfer buying rate (TTBR)** of the
  SBI on the relevant date is commonly used, applied separately to the purchase
  and the sale. This means cost and proceeds are each converted at the rate on
  their own date, so currency movement between buy and sell affects the INR
  gain.
- **Dividends** — converted using the reference rate on (or around) the date of
  receipt.
- **Schedule FA** — the CBDT prescribes using the SBI TT buying rate on
  specified dates (e.g. for peak and closing balances).

The tool performs these conversions for you; always check which rate source and
dates your version uses.

## Capital gains

Gains on the sale of US shares and ETFs are **capital gains** for an Indian
resident. Foreign equity is generally **not** treated as "listed" equity for
Indian purposes, so the favourable rules and holding periods that apply to
Indian listed shares do **not** apply.

- **Holding period** — the split between short-term and long-term for these
  assets is based on a **24-month** threshold: held for **more than 24 months**
  is long-term, otherwise short-term.
- **Short-term capital gains (STCG)** are added to your total income and taxed
  at your **slab rate**.
- **Long-term capital gains (LTCG)** are taxed at the rate set by the current
  Finance Act for such assets. Recent Finance Acts have changed both the rate
  and the availability of indexation, so confirm the rate that applies to your
  filing year.
- Gains and losses are computed **per lot**: each sale is matched to the shares
  bought, and the INR cost and proceeds are converted at the rates on the
  respective dates.

## Dividend income

Dividends from foreign securities are taxable in India as **income from other
sources** and are added to total income at your **slab rate**.

- **US withholding tax** — the US typically withholds tax on dividends paid to
  Indian residents (the India–US tax treaty caps the rate, commonly **25%** when
  the broker has your W-8BEN on file).
- **Foreign tax credit (FTC)** — you can usually claim credit for the US tax
  withheld against your Indian tax on the same income, subject to the treaty and
  Indian rules. Claiming FTC generally requires filing **Form 67** before your
  return.
- The tool reports both the **gross dividend** and the **tax withheld** so you
  have both numbers the return needs.

## Foreign assets (Schedule FA)

A Resident and Ordinarily Resident must disclose foreign assets in **Schedule
FA** of the ITR, regardless of whether any income arose. An IBKR account
typically involves:

- **Foreign custodial / depository account** — the IBKR account itself.
- **Foreign equity and debt interest** — the individual shares and ETFs held.

For each, the schedule asks for details such as the institution, account opening
date, **peak balance** during the period, **closing balance**, and income
attributable to the holding — all on the **calendar-year** basis and converted
to INR using the prescribed SBI TT buying rates. The tool assembles these
per-holding figures from your positions and transactions.

{: .note }
> Schedule FA is a **disclosure** requirement, separate from how income is
> taxed. Non-disclosure of foreign assets carries significant penalties under
> the Black Money Act, so this section deserves careful review.

## Putting it together

A typical filing built from the tool's output uses:

| Figure | Where it goes on the return |
|:-------|:----------------------------|
| Short-term capital gains | Capital Gains schedule (STCG) |
| Long-term capital gains | Capital Gains schedule (LTCG) |
| Foreign dividends | Income from Other Sources |
| US tax withheld | Foreign Tax Credit (with Form 67) |
| Holdings & balances | Schedule FA |

See [Getting started](getting-started) for producing these figures, and the
[FAQ](faq) for common edge cases.
