# 📊 Startup Investment Intelligence

**A Power BI decision-support system that turns 2,818 raw funding-round records into three levels of usable intelligence: market pulse, sector/investor strategy, and single-company due diligence.**

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat-square&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-Data%20Model-blue?style=flat-square)
![Power Query](https://img.shields.io/badge/Power%20Query-ETL-orange?style=flat-square)
![Status](https://img.shields.io/badge/status-complete-brightgreen?style=flat-square)

---

## 1. The Business Problem

Venture capital and startup-ecosystem data is abundant but **fragmented**: funding rounds live in one system, investor mandates live in someone's head or a CRM, and "is this company actually investable" is usually answered by a partner skimming a pitch deck under time pressure.

That creates three concrete, recurring pain points for anyone doing deal sourcing, portfolio strategy, or market research:

1. **No single view of market direction.** Is capital accelerating or pulling back this quarter? Which sectors are heating up? Without a live rollup, teams rely on quarterly PDF reports from third parties (PitchBook, CB Insights) that are backward-looking and expensive.
2. **No structured way to compare sectors and investors against each other.** "Where should we focus our next fund's thesis?" and "which investors actually write checks at our stage?" are questions usually answered anecdotally, not with a cross-tabulation of actual deal data.
3. **No repeatable, explainable way to screen a single startup.** Analysts re-derive "is this company ready, and who should we introduce them to" from scratch for every company that comes across their desk, with no consistent scoring logic and no audit trail for *why* a match was suggested.

**This project exists to close that gap**: one connected data model, three purpose-built views, and a repeatable scoring layer — instead of three disconnected spreadsheets rebuilt every time someone asks a question.

---

## 2. Who This Is For

| User | What they get | What they'd otherwise do |
|---|---|---|
| **Fund partners / leadership** | A 5-second read on total market activity, trend direction, and where the fund's thesis sectors rank | Wait for a quarterly PitchBook/CB Insights report |
| **Deal team / associates** | A sector × stage intensity map and a ranked investor league table to plan outreach and co-investment | Manually cross-reference spreadcheets of prior deals |
| **Analysts screening inbound deals** | A one-click, per-company readiness score and ranked investor-fit list with a plain-English rationale | Re-build a one-off comparison in Excel for every company |

---

## 3. Analytical Approach

### 3.1 Design principle: three altitudes, one model
Rather than one crowded dashboard, the report is deliberately split by **altitude of decision**:

- **Executive Overview** → macro / "should we care right now" (seconds to read)
- **Sector & Investor Analysis** → strategic / "where do we play and with whom" (minutes to read)
- **Startup Scorecard** → tactical / "should we act on this one deal" (a working tool, not just a chart)

All three pages sit on **one shared semantic model**, so a KPI on page 1 and a filtered row on page 3 are always mutually consistent — a design choice that matters more than it sounds, since it's what lets a partner glance at page 1, drill into a sector on page 2, and land on a specific company on page 3 without the numbers ever disagreeing.

### 3.2 Data model: star schema

```
                    ┌────────────────┐
                    │  dim_startup    │
                    └────────┬────────┘
                             │
┌────────────────┐   ┌──────▼──────────────┐   ┌────────────────────┐
│  dim_investor    │──▶│ fact_funding_rounds │◀──│  ref_real_unicorns  │
└────────────────┘   └──────┬──────────────┘   └────────────────────┘
                             │
                 ┌───────────┴────────────┐
                 │                        │
         ┌───────▼────────┐   ┌───────────▼──────────────────┐
         │ ml_training_set │   │ supplementary_funding_events  │
         └─────────────────┘   └───────────────────────────────┘
```

| Table | Role | Grain |
|---|---|---|
| `fact_funding_rounds` | Transactional core | One row per funding round (amount, stage, post-money value, lead investor) |
| `dim_startup` | Descriptive dimension | One row per company (sector, country, founding year, headcount, revenue status) |
| `dim_investor` | Descriptive dimension | One row per investor (type, HQ, historical deal count) |
| `ml_training_set` | Feature table | Powers the Investment Readiness Signal and Investor Fit Score — treats "is this company ready" and "who should invest" as **scored problems**, not static lookups |
| `ref_real_unicorns` | Benchmark reference | Ground-truth outcomes to sanity-check the scoring logic against |
| `supplementary_funding_events` | Enrichment | Fills gaps in the core fact table (e.g. older or informally-reported rounds) |

This is a deliberate **star schema**, not a flat export — it's what makes the report fast to filter and lets one investor selection instantly re-slice every visual on every page.

### 3.3 The scoring layer (what makes this more than a report)
The Scorecard page isn't just a filtered table — it runs two derived scores per startup:

- **Investment Readiness Signal (0–100 gauge)**: a composite read on how investable a company looks *right now*, based on its profile (revenue status, funding stage, round cadence)
- **Investor Fit Score (0–100, per investor)**: ranks the active investor base against the selected startup, with a **Match Basis** column that states *why* in plain language (e.g. "Check size closely aligns") instead of returning an opaque number

That explainability layer is the difference between "a dashboard with a startup filter" and an actual **screening tool** an analyst could defend in an investment committee meeting.

---

## 4. Walkthrough & What Each Page Actually Tells You

### Page 1 — Executive Overview
*Global startup funding activity, 2020–2026*

![Executive Overview](screenshots/01-executive-overview.png)

**What's on it:** headline KPIs (Total Funding Tracked $33.53B, 2,818 Total Rounds, 73 Active Investors, $11.90M Avg Round Size, 1,747 Unique Startups), a quarterly funding trend line, sector ranking bar chart, geographic funding map, and a funding-stage donut.

**What it tells you, analytically:**
- **AI and Fintech dominate sector share** — together they account for roughly **half of tracked sector funding** (AI ≈ 26%, Fintech ≈ 24% of the sector breakdown shown), more than double Cybersecurity's share at the bottom of the ranking.
- **The market skews late.** The stage donut shows **Series B and Series C alone make up ~58.5% of deal volume**, with Pre-Seed under 2%. Read together with the quarterly trend, this says the tracked market is dominated by follow-on capital into already-de-risked companies, not new-company formation — an important framing note for anyone using this data to reason about "the startup market" broadly.
- **The 2026 pipeline point is a leading indicator, not a year-end result** — it sits well below prior years because the year isn't complete. Flagged explicitly so it isn't misread as a market collapse.

### Page 2 — Sector & Investor Analysis
*Where capital concentrates, and who is deploying it*

![Sector & Investor Analysis](screenshots/02-sector-investor-analysis.png)

**What's on it:** a conditionally-formatted **Sector × Stage funding intensity matrix**, an **average raised per startup by founding cohort** bar chart, and a **Top Investors by Capital Deployed** league table.

**What it tells you, analytically — and this is the most interesting finding in the whole model:**
- **Capital intensity is front-loaded at Seed, not growth stage — across almost every sector.** In the matrix, Seed-stage totals *exceed* Series A totals, which exceed Series B, and so on, in nearly every row (e.g. Artificial Intelligence: Seed 5.90M → Series A 2.31M → Series B 1.88M → Series C 0.54M → Series D 0.20M). That's the **opposite of the typical VC pattern** where check size grows with each round. Read against Page 1's donut (deal *volume* concentrated at Series B/C), the combined story is: **many small early bets, fewer but larger-ticket-per-round follow-ons that in aggregate still trail Seed-stage totals in this dataset** — a genuinely useful, non-obvious pattern this dashboard is built specifically to surface, and exactly the kind of thing that gets lost in a flat spreadsheet.
- **Founding-cohort economics favor older companies.** Average capital raised per startup is highest for the 2016 cohort and generally declines for more recently founded cohorts — expected, since older companies have simply had more time to stack multiple rounds, but useful as a normalization check before comparing cohorts head-to-head.
- **The investor league table separates deal count from check size.** Sequoia Capital leads 10th by deal count (29 deals) but has by far the **highest average round size ($34.48M)** in the top 10 — more than double the next-highest (Y Combinator, $17.50M). That distinction — high-conviction, fewer/larger bets vs. high-volume, smaller-check strategies — is exactly the kind of segmentation a fund would use to decide who to co-invest with vs. who to compete against.
- **A documented data-quality caveat sits directly on the page**: `hq_country` is flagged as *"verified unreliable for some records."* This is a deliberate design choice — surfacing a known data limitation in the report itself, rather than letting a viewer draw a false geographic conclusion.

### Page 3 — Startup Investment Scorecard
*Startup-level investment profile and readiness analysis*

![Startup Scorecard](screenshots/03-startup-scorecard.png)

**What's on it:** a company-selector slicer feeding a **Company Profile** table, an **Investment Readiness Signal** gauge, a ranked **Top Investor Matches** table with plain-language rationale, and a full **Funding History** ledger.

**What it tells you, analytically:**
- This page converts the entire model into a **single-company report on demand.** Selecting **AlphaAI** (Climate Tech / Clean Energy, Netherlands, founded 2019, Series A, 3 rounds) instantly recomputes:
  - a **67/100 Readiness Signal**
  - a re-ranked investor list topped by **Sequoia Capital (66.71 fit)**, all the way down to **Accel India (40.36 fit)**
  - a chronological funding ledger from a $0.18M Pre-Seed (NEA, 2022) to a $38.37M Series C (NVIDIA NVentures, 2023)
- Every fit score carries a **Match Basis** ("Check size closely aligns") — so an analyst can defend the recommendation in a meeting instead of just citing a number.
- With no company selected, the page defaults to an **unfiltered aggregate state** (visible in the raw export) rather than breaking — a small but deliberate UX safeguard for a report that will be used by non-technical stakeholders.

---

## 5. Key Findings Summary

1. **AI + Fintech ≈ half of all sector-level funding** in the dataset — the two categories any thesis-driven fund would need a point of view on.
2. **Deal volume is late-stage-heavy (Series B/C ≈ 58.5% of rounds)** while **capital intensity is Seed-heavy** in almost every sector — a genuine tension in the market structure that only shows up when volume and dollar-intensity are viewed side by side, which is exactly why this dashboard separates them onto different visuals instead of collapsing them into one number.
3. **Investor strategy bifurcates clearly**: high-volume/smaller-check investors (Y Combinator, 47 deals) vs. low-volume/large-check investors (Sequoia Capital, $34.48M avg) — a segmentation that should directly inform co-investment and competitive-positioning decisions.
4. **Older founding cohorts show higher average capital raised per startup** — a reminder to normalize for company age before comparing vintages.
5. **Known data-quality gaps are surfaced in-product** (`hq_country` reliability), reflecting analysis discipline rather than blind trust in the source feed.

---

## 6. Technical Implementation

- **Power Query (M)** for ingestion and shaping across six source tables
- **Star-schema data model** (see §3.2) enabling fast, consistent cross-filtering across all three pages
- **DAX** measures driving every KPI card, the sector × stage intensity matrix, the Readiness gauge, and the dynamic Investor Fit ranking
- **Custom navigation** — bookmarks + action buttons replace the default Power BI tab strip with a persistent top nav (`EXEC. OVERVIEW` / `SECTOR & INVESTOR` / `STARTUP SCORECARD`)
- **Parameterized drill-through** on the Scorecard page via a startup-selector slicer that re-renders the entire page, including the two scoring visuals
- **Conditional formatting** on the sector × stage matrix for at-a-glance intensity reading
- **In-report data-quality annotation** rather than a hidden caveat — the `hq_country` footnote is a report design decision, not an afterthought

## 7. Known Limitations

- `hq_country` in the investor table is explicitly flagged as unreliable for a subset of records — treat country-level investor geography as directional, not authoritative, until validated against the project data dictionary.
- 2026 figures represent a partial year and should not be compared directly to full prior years without normalization.
- The Investment Readiness Signal and Investor Fit Score are model-driven estimates meant to **augment** analyst judgment, not replace investment committee due diligence.

## 8. Repo Contents

```
Startup_Investment_Intelligence.pbix         → full Power BI source file (data model + report)
StartupInvestmentIntelligence_Dashboard.pdf  → static export of all 3 pages
screenshots/                                 → PNG exports used in this README
README.md                                    → you are here
```

## 9. Getting Started

1. Open `Startup_Investment_Intelligence.pbix` in **Power BI Desktop**.
2. Use the top navigation bar to move between **Exec. Overview**, **Sector & Investor**, and **Startup Scorecard**.
3. On the Scorecard page, use the **"Select a startup for report"** slicer to generate a live one-pager for any of the 1,747 companies — the gauge and investor rankings recompute instantly.

## 10. Roadmap

- [ ] Replace the retiring Bing filled-map visual with the **Azure Maps** visual
- [ ] Add a cohort-retention view (follow-on round conversion rate by stage) to complement the Seed-vs-later-stage intensity finding
- [ ] Formalize and publish the data dictionary referenced in the `hq_country` footnote
- [ ] Publish to a Power BI Service workspace with scheduled refresh and row-level security by fund/team

---

<p align="center"><i>Built with Power BI · Data current through 2026</i></p>
