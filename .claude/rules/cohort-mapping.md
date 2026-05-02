# Cohort Mapping

All values of `tnt__customer_cohort_dropdown` from DevRev:

## Tier 1 — Dedicated Enterprise
- `1-Reliance` — RIL, Jio, reliancepbg, Reliance Grocery, [WMS] Reliance
- `1-DTDC` — DTDC, dtdc.in, dtdc.co

## Tier 2 — Strategic Accounts
- `2-Aramex` — Aramex Global, Aramex VW, Aramex Move, Aramex SDD, Aramex Freight, Aramex Oceania, Aramex RO
- `2-HNK` — Heineken, HNK-BR1-Primary, HNK-BR1-Secondary, heineken-br1

## Tier 3 — On Demand
- `3A-On Demand` — Flipkart, Swiggy, Box, Myntra, healthkart (some)
- `3B-S (On Demand)` — smaller on-demand accounts

## Tier 4 — B2C / 3PL
- `4A-B2C Shipper` — Rozana, healthkart, Kama Ayurveda, Sugarcosmetics, Wakefit
- `4A-B2C LSP` — B2C logistics providers
- `4B-S (B2C Shipper)` — smaller B2C shippers
- `4B-S (B2C LSP)` — smaller B2C LSPs

## Tier 5 — B2B
- `5A-B2B LSP` — proconnect, SBT, Kerry Logistics, Fmlogistic
- `5A-B2B Shipper` — B2B shippers
- `5B-S (B2B LSP)` — smaller B2B LSPs
- `5B-S (B2B Shipper)` — smaller B2B shippers

## Special
- `WMS` — WMS-specific accounts ([WMS] JGH, Milkbasket, Stockone, [WMS] Wellness Forever)
- `Exim` — EXIM product accounts
- `Platform` — platform/infra tickets
- `AI` — AI-related tickets
- `Roadmap` — feature roadmap items
- `TBD` — unclassified, needs manual triage

## Email Domain -> Cohort Fallback
- @ril.com -> 1-Reliance
- @dtdc.com, @dtdc.in -> 1-DTDC
- @aramex.com -> 2-Aramex
- @ibm.com (Heineken contractor) -> 2-HNK
- @flipkart.com -> 3A-On Demand
- @rozana.in -> 4A-B2C Shipper
- @wellnessforever.in -> 3A-On Demand or WMS (check applies_to_part)
- @proconnectlogistics.com -> 5A-B2B LSP
