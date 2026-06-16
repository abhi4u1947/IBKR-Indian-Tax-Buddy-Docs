# Pricing & Go-To-Market — IBKR Indian Tax Assistant (India)

> The product is **reference assistance only**; outputs are estimates and **must
> be verified by a certified Chartered Accountant (CA) in India** before filing.
> This positioning is core to our marketing: *we make your CA faster, we don't
> replace them.*

## 1. Target customers (ICP)

1. **Retail Indian investors with US/global stocks via IBKR** — RSUs/ESPPs from
   MNC employers, NRIs-turned-residents, self-directed global investors. Pain:
   Schedule FA, FA/CG reporting, FX conversion, STCG/LTCG classification.
2. **Chartered Accountants & tax practitioners** — handle many such clients each
   July; need a fast, auditable working-paper generator.
3. **Wealth/RIA firms & employer ESOP desks** — bulk/seat licensing.

## 2. Pricing (INR, GST 18% extra)

| Tier | Price | For | Highlights |
|---|---|---|---|
| **Free** | ₹0 | Try-before-buy | 1 account, < 25 transactions, on-screen STCG/LTCG estimate, watermarked summary |
| **Investor** | **₹1,499 / financial year** | Individual filers | Unlimited transactions, IBKR Flex import, FIFO/Spec-ID, RBI/SBI FX to INR, dividend + foreign-tax-credit schedules, Schedule FA helper, PDF + Excel, AI explanations |
| **Investor+** | **₹2,999 / FY** | Multi-account / complex | Multiple IBKR accounts, corporate-action handling, prior-year carry-forward, priority support |
| **Professional (CA)** | **₹7,999 / year** | Tax consultants | Up to 25 clients, bulk import, white-label reports, client workspace, audit pack; +₹299/extra client |
| **Firm / Enterprise** | Custom | RIAs, ESOP desks | SSO, seats, API access, SLA, onboarding |

Notes: prices are **per financial year** for investor tiers (aligns with the ITR
cycle); GST (18%) shown separately at checkout; UPI/cards/netbanking via Razorpay
or Stripe India. Early-bird and student/first-time-filer discounts supported via
coupon codes.

## 3. Positioning & messaging

- **Primary promise:** "File your IBKR foreign-stock taxes in India — correctly,
  in minutes, ready for your CA to sign off."
- **Trust pillars:** accuracy (Decimal math, versioned rules, audit trail),
  transparency (every number explained + sourced), and **CA verification** built
  into the flow (not a black box).
- **Differentiators vs spreadsheets / generic tools:** IBKR-native Flex import,
  RBI/SBI FX done right, post-Budget-2024 LTCG handling as *versioned data*,
  Schedule FA support, and explainable, audit-ready outputs.

## 4. SEO strategy

- **Pillar pages / high-intent keywords:** "tax on US stocks in India",
  "foreign capital gains tax India", "Schedule FA reporting", "RSU tax India",
  "IBKR India tax", "LTCG on foreign shares Budget 2024", "FA schedule ITR".
- **Programmatic/long-tail:** per-broker, per-asset, per-FY guides; "STCG vs LTCG
  for foreign shares FY 2024-25"; FX-rate explainer pages.
- **On-page:** Next.js SSR/ISR for crawlability, JSON-LD (`SoftwareApplication`,
  `FAQPage`, `Organization`), clean metadata + OG/Twitter cards, `sitemap.ts`,
  `robots.ts`, fast Core Web Vitals.
- **Content engine:** blog with accurate, reference-only explainers (each with the
  CA-verification disclaimer), updated each budget cycle. Internal linking to pricing
  + onboarding.
- **Authority:** guest posts, CA community partnerships, comparison pages.

## 5. Marketing & analytics integration

- **Analytics:** GA4 and/or privacy-friendly **Plausible** (toggled via
  `NEXT_PUBLIC_GA_ID` / `NEXT_PUBLIC_PLAUSIBLE_DOMAIN`); conversion events
  (signup, import, calc-complete, upgrade), UTM capture.
- **Lifecycle:** transactional + drip email (Resend/SES) — verification,
  onboarding nudges, "your estimate is ready", July-deadline reminders.
- **Acquisition channels:** SEO/content (primary), CA partner/referral program,
  community (r/IndiaInvestments, X/LinkedIn finance), targeted ads in tax season,
  employer ESOP partnerships.
- **Referral/affiliate:** coupon-based referral for users and a revenue-share
  affiliate for CAs/finfluencers.

## 6. Sales motion

- **Self-serve PLG** for Free/Investor (in-app upgrade, Razorpay/Stripe checkout).
- **Assisted/sales-led** for Professional/Firm: demo booking, client-import
  onboarding, annual contracts, seat expansion.
- **Seasonal pushes:** Apr–Sep campaigns around the ITR deadline; off-season:
  content + CA partnerships + product depth.

## 7. Promotion calendar (illustrative)

| Window | Focus |
|---|---|
| Apr–May | "Start early" content + early-bird pricing |
| Jun–Jul | Peak acquisition; deadline reminders; CA partner push |
| Aug–Sep | Late-filer + revised-return messaging |
| Oct–Mar | Budget watch, product depth, content/SEO compounding |

## 8. KPIs
Acquisition (organic sessions, signups), activation (import → calc completion),
conversion (free→paid), retention (FY-over-FY renewal), CA seats, NRR for Firm
tier, CAC payback, and gross margin (see `COST_ESTIMATES.md`).

> All marketing must preserve the compliance message: estimates, reference-only,
> **verify with a certified Indian CA**. No "guaranteed savings" or "tax advice"
> claims.
