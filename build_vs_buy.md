# Build vs. Buy — Transparency-in-Coverage rate data

*Scoping memo for the policy team. Texas 2036, Data/Research.*

## The decision in one line
You are not really deciding whether to **parse JSON** — that's solved (see the
working prototype). You're deciding whether to take on the **data-cleaning
burden**: an estimated **70–92% of raw MRF negotiated rates are "ghost rates"**
(posted for provider/code pairs with no real volume). Vendors' value-add is
normalization, de-duplication, ghost-rate removal, and provider attribution —
the same work we'd otherwise do ourselves.

> Peer-reviewed: across 61 insurers, **91.8%** of negotiated rates were ghost rates;
> median insurer 84.3%. Even within the 100 most common billing codes, 70.3% were ghost.

## What's actually free (do this first — near-zero cost)
A pilot that needs no budget and lets our Stata analysts validate coverage and the
ghost-rate burden on real data:
1. **BCBSTX direct** — our prototype already pulls these monthly (free, raw).
2. **Turquoise researcher dataset** — free, non-commercial DUA; 14 shoppable services, all US hospitals.
3. **Turquoise Community Tier** — free/low-cost, near-full backend access.
4. **Payerset free samples** — on the Snowflake Marketplace.
5. **Trilliant free DuckDB lake** — 6B+ rates, *but hospital MRFs, not payer* (validation only).

## Vendor landscape (verified May 2026)

| Vendor | Sells | TX/BCBSTX coverage | Delivery (Stata-ready?) | Price |
|---|---|---|---|---|
| **Serif Health** | Raw→normalized rates | 200+ payers, all 50 states, incl. BCBS | Portal + API; **Parquet/CSV** bulk via Snowflake/SFTP; **pre-filter by NPI/TIN/CPT/geo** | **~$1,000/mo** for a region (starting); bulk license unpublished. Advertises **permissive IP/caching** terms |
| **Payerset** | Rate data lake | **Documented deep BCBSTX** (3.6B provider, 4.6B rate records) | Snowflake/Databricks/cloud; **Parquet**, NPI↔TIN map | **Published**: Data Lake **$75K** (≤100 codes) / **$150K** (≤500) / **$288K** (all)/yr |
| **Turquoise Health** | Clear/Core Rates + research tiers | National payers + each state BCBS (BCBSTX in scope) | Filterable by payer/provider/code/geo | License + annual sub (**undisclosed**); **academic HD4A** + free researcher tier |
| Zelis | Payer-facing analytics | n/a | Not a research data license | Not a fit |
| Healthcare Bluebook / Valenz | Claims-based "Fair Price" estimates | n/a | Benchmark product, **not raw MRF rates** | Not a fit |
| Trilliant (enterprise) | Analytics platform | Hospital + payer MRFs | Dashboards only, **no flat-file extracts** | Demo-only |
| DoltHub / open | Free SQL datasets | Small, **stale (~Q3 2023)** | SQL/CSV | Free (validation only) |

*Not negotiated-rate sources (common confusions): Nightingale Open Science = medical imaging; Garner/Quote = claims-based benefit navigation; "MedPriceBenchmarks" could not be verified.*

## Recommendation
- **Run the free pilot now** (above) in parallel with finishing the in-house pipeline.
- **For a paid recurring Texas pull, shortlist Serif Health + Payerset**, and request a **Turquoise academic/HD4A** quote. Serif is the best entry point (transparent starting price, Parquet, pre-filtering, publishing-friendly IP terms); Payerset is the comparison baseline (public prices, documented BCBSTX depth).
- **Insist on pre-delivery filtering** (target payers × Texas geography × our code set) so extracts land in Stata without a big-data tier. *Stata 19/StataNow reads Parquet natively; older Stata → stage in DuckDB, export filtered Parquet/CSV.*

## Rough annual budget envelope
| Tier | Cost/yr | What |
|---|---|---|
| Exploratory | **$0** | BCBSTX direct + Turquoise researcher/Community + Trilliant free lake |
| Light subscription | **~$12K–$30K** | Serif-style regional API/portal |
| Full enterprise license | **~$75K–$300K** | Payerset Data Lake / Turquoise full |
| Build in-house | **not "free"** | Data-engineering FTE time + cloud storage/compute for TB-scale monthly refresh + ongoing ghost-rate cleaning |

## RFI questions to send each vendor
1. Confirm BCBSTX + specific TX payers/networks/lines of business (fully-insured, ASO, exchange), refresh cadence, historical depth.
2. Flat-file extracts we control (Parquet/CSV) vs dashboard/API only? Pre-delivery filtering by payer × TX geography × our CPT/HCPCS/DRG list?
3. How are **ghost rates** and rate **methodology** (case-rate vs per-unit vs % of charges) handled — flagged or filtered?
4. Provider attribution: NPI/TIN crosswalk quality; duplicate resolution.
5. Exact pricing, term, and **nonprofit/academic** pricing.
6. **Licensing**: may we publish derived statistics/methods openly, redistribute aggregates, cache internally? Any caching/IP restrictions?
7. Free/trial **Texas sample** so we can test Stata ingestion and validate against BCBSTX's own MRFs.

## Why methodology normalization matters (a caution for any path)
MRFs lack a standard methodology field. One analysis found CPT 99282 ranging **$820 (per-unit) to $4,651 (case-rate)** — a ~500% spread driven purely by *how* the rate was expressed, not real price differences. And a single payer can use **400+ contract-naming variants**. Whether we build or buy, our published statistics must control for `negotiated_type`/methodology and normalize plan/network naming, or the numbers will mislead.
