# B2B CRM Sales Opportunity Analysis

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-7B5EA7?style=flat&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-336791?style=flat&logo=postgresql&logoColor=white)
![Data Modelling](https://img.shields.io/badge/Data%20Modelling-0D6EFD?style=flat&logoColor=white)

End-to-end sales pipeline analysis for a B2B CRM dataset — opportunity tracking, rep performance benchmarking, and revenue forecasting in Power BI. Built to simulate a real analyst handoff to a VP of Sales.

---

## Contents

1. [Business Problem](#business-problem)
2. [Dashboard Overview](#dashboard-overview)
3. [Key Findings](#key-findings)
4. [Data Model](#data-model)
5. [DAX Highlights](#dax-highlights)
6. [How to Open](#how-to-open)
7. [What I'd Do With More Data](#what-id-do-with-more-data)

---

## Business Problem

A B2B sales company wants to understand where pipeline is being won or lost across stages, reps, and products. The key stakeholder questions: *Are we tracking to target? Which reps need coaching vs. scaling? Where is deal velocity lowest?*

This dashboard gives a Sales VP a single view to answer all three without pulling raw CRM exports every week.

---

## Dashboard Overview

### Page 1 — Pipeline & Revenue

| Overview Dashboard | Pipeline Funnel |
|---|---|
| ![Overview](screenshots/overview_dashboard.png) | ![Funnel](screenshots/pipeline_funnel.png) |
| Monthly won revenue vs. target with pipeline coverage ratio. Q3 dip visible — flagged in findings. | Stage-by-stage conversion rates. Prospect → Qualified has the highest drop-off at 38%. |

### Page 2 — Rep Performance

| Rep Performance | Product Mix |
|---|---|
| ![Reps](screenshots/rep_performance.png) | ![Product](screenshots/product_mix.png) |
| Win rate, average deal size, and deal velocity per rep. Sortable by any metric. | Revenue split by product category across regions. Enterprise tier drives 71% of total closed-won value. |

---

## Key Findings

| Total Pipeline | Win Rate | Avg Sales Cycle |
|:---:|:---:|:---:|
| **$10.0M** | **48.2%** | **48 days** |

**Pipeline concentration risk**
Top 3 reps account for 61% of closed-won revenue. Bottom quartile shows a 40% longer average sales cycle, suggesting a coaching opportunity rather than a capacity issue.

**Q3 coverage ratio dropped to 2.1×**, below the 3× benchmark — a leading indicator of the Q4 miss. The gap appeared in July and widened through September with no corrective action visible in the data.

**Prospect → Qualified is the biggest leak:** 38% of opportunities stall at this stage. Combined with a median time-in-stage of 19 days, this points to a qualification criteria problem, not a volume problem.

**Enterprise tier outperforms SMB** on both win rate (+11 pp) and deal size (3.8× larger). Current rep allocation does not reflect this — SMB accounts for 54% of rep hours.

---

## Data Model

Star schema with one fact table and four dimension tables. The date table is generated in DAX to enable independent cross-filtering on both `created_date` and `close_date`.

```
fact_opportunities          dim_date               dim_rep
─────────────────────       ────────────────────   ────────────────────
PK  opportunity_id    ───── PK  date_key            PK  rep_key
    amount                      date                    rep_name
    stage                       month                   region
FK  created_date_key            quarter                 team
FK  close_date_key              fiscal_year
FK  rep_key           ──────────────────────────────────────────────────
FK  product_key                                     dim_product
                                                    ────────────────────
                                                    PK  product_key
                                                        product_name
                                                        category
                                                        tier
```

> The date table is role-played twice — once for `created_date` and once for `close_date` — so filters on either field work independently across all visuals.

---

## DAX Highlights

### Win rate (filtered by stage)

```dax
-- Won / (Won + Lost) — excludes open pipeline from the denominator
Win Rate =
  DIVIDE(
    CALCULATE(COUNTROWS(fact_opportunities), fact_opportunities[stage] = "Won"),
    CALCULATE(COUNTROWS(fact_opportunities),
      fact_opportunities[stage] IN {"Won", "Lost"})
  )
```

### Rolling 3-month revenue

```dax
-- Used to smooth seasonal noise on the revenue trend line
Revenue 3M Rolling =
  CALCULATE(
    [Total Revenue],
    DATESINPERIOD(dim_date[date], LASTDATE(dim_date[date]), -3, MONTH)
  )
```

### Pipeline coverage ratio

```dax
-- Open pipeline / remaining target. Drops below 3x = risk signal
Coverage Ratio =
  DIVIDE([Open Pipeline], [Remaining Target])
```

---

## How to Open

1. Download [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free)
2. Clone this repo or download the `.pbix` file directly
3. Open `CRM_Sales_Opportunities.pbix` in Power BI Desktop

> The dataset is fully synthetic — no real customer or revenue data is included. No gateway or credentials are required.

---

## What I'd Do With More Data

With real CRM data I'd layer in lead source attribution to close the loop between marketing spend and pipeline quality. The rep performance page would be significantly more useful with quota attainment data — raw revenue figures without targets can't distinguish a low-performer from someone with a small territory.

I'd also build a stage-transition probability model in Python (logistic regression or a simple Markov chain) and surface it as an embedded visual to give reps a live win-probability score per deal.

---

*Dataset is synthetic and generated for portfolio purposes. All figures are illustrative.*
