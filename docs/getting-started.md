---
title: Getting started
layout: default
nav_order: 2
---

# Getting started
{: .no_toc }

This guide walks through getting the right data out of Interactive Brokers and
producing your first set of Indian tax figures.

1. TOC
{:toc}

---

## Prerequisites

- An **Interactive Brokers** account with access to statements.
- The **IBKR Indian Tax Buddy** tool. See the
  [project repository](https://github.com/abhi4u1947/IBKR-Indian-Tax-Buddy) for
  installation and the exact run command, which may change between releases.
- Your **financial year** in mind: India taxes on a 1 April – 31 March basis,
  while IBKR and US dividends report on a 1 January – 31 December calendar year.
  You will usually export data covering both windows (see
  [Concepts](concepts)).

## Step 1 — Export your IBKR statement

IBKR offers two kinds of exports. A **Flex Query** is preferred because it is
machine-readable and you control exactly which sections are included.

1. Log in to **Client Portal** (or Account Management).
2. Go to **Performance & Reports → Flex Queries**.
3. Create a new **Activity Flex Query** and include at least these sections:
   - **Trades** — required for capital gains.
   - **Cash Transactions** (dividends and withholding tax) — required for
     dividend income.
   - **Open Positions** and **Net Asset Value** — required for Schedule FA.
   - **Cash Report** / **Transfers** — helpful for reconciliation.
4. Set the **date period** to cover the year you are filing for. For Schedule FA
   you will need the **calendar year** (1 Jan – 31 Dec); for capital gains and
   dividends you will need the **Indian financial year** (1 Apr – 31 Mar). It is
   simplest to export a wide range that covers both.
5. Set the format to **CSV** (or **XML**), run the query, and save the file.

{: .note }
> If you only have access to the standard **Activity Statement**, export it for
> the same period in CSV. The tool's README documents which statement formats
> the current release accepts.

## Step 2 — Run IBKR Indian Tax Buddy

Point the tool at the statement file you exported. The typical flow is:

1. Provide the exported statement file as input.
2. Tell the tool which **financial year** you are computing for.
3. The tool parses trades, dividends, and positions, converts amounts to INR
   using the appropriate reference rates, and produces a summary.

Refer to the [project repository](https://github.com/abhi4u1947/IBKR-Indian-Tax-Buddy)
for the exact command-line invocation and any configuration options for your
version.

## Step 3 — Review the output

The tool produces figures grouped the way the return needs them:

- **Capital gains** — short-term and long-term, with INR proceeds and cost.
- **Dividend income** — gross dividends and US tax withheld.
- **Schedule FA** — per-holding disclosure details on the calendar-year basis.

Read [Concepts](concepts) to understand how each of these maps onto your
return, then verify every figure against your own records before filing.

## Next steps

- [Concepts](concepts) — the tax background behind each figure.
- [FAQ](faq) — currency conversion, missing data, and common edge cases.
