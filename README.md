# QuickbiteExpress
# 🍔 QuickBite Express — Crisis Impact & Recovery Analysis

## 📌 Project Overview

**QuickBite Express** is a Bengaluru-based food-delivery startup (founded 2020) operating across 8 Indian cities. In **June 2025**, a viral food-safety incident combined with a week-long monsoon delivery outage triggered a severe trust crisis: customers disengaged, ratings fell, restaurant partners drifted away, and acquisition costs rose.

This project analyzes **9 months of data (Jan–Sep 2025)** to answer one central question:

> **Did QuickBite recover after the crisis, or did the crisis become the new normal — and where should recovery investment go first?**

The analysis compares two phases throughout:

| Phase | Period | Months |
|---|---|---|
| **Pre-Crisis** | Jan – May 2025 | 5 |
| **Crisis** | Jun – Sep 2025 | 4 |

---

## 🎯 Business Questions Answered

1. How severely did order volume decline, pre-crisis vs. crisis?
2. Which cities suffered the sharpest percentage decline?
3. Which high-volume restaurants declined the most?
4. How did cancellation rates change, and where?
5. Did delivery SLA compliance collapse during the crisis?
6. How did customer ratings move month by month?
7. What themes dominate negative reviews during the crisis?
8. What is the estimated revenue loss versus pre-crisis?
9. How many loyal, highly-rated customers churned anyway?
10. Which top-spend customers declined most, and what patterns do they share?

---

## 🔑 Key Findings

| Metric | Pre-Crisis | Crisis | Change |
|---|---|---|---|
| Monthly orders (avg) | 22,761 | 8,840 | **−61.2%** |
| Net revenue | ₹3.76 Cr | ₹1.09 Cr | **−70.9%** |
| Cancellation rate | 6.06% | 11.93% | **~2× increase** |
| SLA compliance | 43.6% | 12.2% | **−31.4 pts** |
| Avg delivery delay | 2.0 min | 17.6 min | **~9× longer** |
| Avg rating | 4.49 (May) | 2.31 (Sep) | **−2.18 pts** |

**Headline insights**

- 📉 **The crisis became the new baseline.** Orders fell ~60% in June and never recovered — September volume (8,694) is essentially flat with June (9,293).
- 👥 **7 in 10 customers churned.** Of 99,790 active customers, 70,970 (71.1%) ordered before June and then stopped.
- 💎 **VIPs were hit hardest, not least.** Top-5% spenders churned at 85.9% (vs. 71.1% overall), with ~₹0.47 Cr in annual spend at risk — a pattern that points to a *trust failure*, not price sensitivity.
- ❤️ **Even happy loyal customers left.** Of 50 loyal customers (≥5 pre-crisis orders), 42 churned — and 22 of those were still rating the platform above 4.5.
- 🚚 **Operations, not price or cuisine, drove the exodus.** Cloud Kitchens and Restaurants declined almost identically (70.5% vs. 71.0%) while SLA compliance collapsed.
- 🏙️ **Tier-1 metros hold the biggest recovery prize.** Bengaluru alone lost ~₹65.8 lakh in revenue; Bengaluru, Mumbai, and Delhi together account for the largest share of the shortfall.

---

## 💡 Recommendations

1. **Fix delivery reliability first** — SLA must recover from 12% before any win-back marketing will stick.
2. **Target the 22 loyal, high-rating churners** — the lowest-risk, highest-probability win-backs.
3. **Launch a VIP retention program** — ₹0.47 Cr in annual spend at risk from the top 5% alone.
4. **Run a public food-safety audit & certification campaign** — addresses the root cause, not just symptoms.
5. **Concentrate recovery budget in Bengaluru, Mumbai, and Delhi.**

---

## 📊 Dashboard Structure

The Power BI report contains **8 navigable pages**:

| # | Page | Purpose |
|---|---|---|
| 1 | **Home** | Landing page & navigation |
| 2 | **Executive Summary** | Headline KPIs, monthly orders/revenue trend, city decline, revenue component shortfall |
| 3 | **Customer Analysis** | Churned / New / Retained / Loyal segmentation by city |
| 4 | **Top 5% Customers** | VIP segment — revenue at risk, order frequency, cuisine and city patterns |
| 5 | **Delivery & Operations** | SLA compliance, delay trends, cancellation by city, vehicle type & time bucket |
| 6 | **Restaurant Partnership** | Retention, risk levels, cuisine and partner-type comparison |
| 7 | **Ratings & Sentiment** | Rating trend, sentiment trend, negative review themes |
| 8 | **Additional Insights** | Revenue loss by city tier, recovery potential, return segments |

> 📸 *Add screenshots to `/screenshots` and embed them here, e.g.:*
> `![Executive Summary](screenshots/executive_summary.png)`

---

## 🗂️ Dataset

8 CSV files, ~1M+ total rows:

| Table | Type | Rows | Description |
|---|---|---|---|
| `fact_orders` | Fact | 149,166 | One row per order (amounts, COD flag, cancellation flag) |
| `fact_order_items` | Fact | 342,994 | Line items per order |
| `fact_delivery_performance` | Fact | 149,166 | Actual vs. expected delivery time, distance |
| `fact_ratings` | Fact | 68,825* | Rating, review text, sentiment score |
| `dim_customer` | Dimension | 107,776 | Signup date, city, acquisition channel |
| `dim_restaurant` | Dimension | 19,995 | City, cuisine, partner type, prep-time bucket |
| `dim_delivery_partner` | Dimension | 15,000 | Vehicle type, employment type, rating |
| `dim_menu_item` | Dimension | 342,671 | Item, category, veg flag, price |

\* 17 fully blank rows removed during cleaning (68,842 raw).

---

## 🧱 Data Model (Star Schema)

```
                      Date
                       │
                       ▼
dim_delivery_partner ─▶ fact_orders ◀─ dim_customer
                       │   ▲   ▲
                       │   │   └── dim_restaurant ──▶ dim_menu_item ──▶ fact_order_items
                       ▼   │
      fact_delivery_performance   fact_ratings
              (1:1)                 (1:1)
```

- **9 relationships**, all 1:* single-direction except the two 1:1 extension tables (`fact_ratings`, `fact_delivery_performance`), which are bidirectional.
- Dedicated **Date table** (marked as date table) with month-number sort columns.
- Redundant keys (`fact_ratings.customer_id`, `fact_ratings.restaurant_id`, `fact_order_items.restaurant_id`) intentionally left unrelated to avoid ambiguous filter paths.

---

## 🧹 Data Cleaning (Power Query)

- Fixed data types across all 8 tables
- Standardized mixed date formats (ISO vs. `DD-MM-YYYY`) with explicit `en-GB` parsing
- Removed 17 blank rows from `fact_ratings`
- Left `delivery_partner_id` nulls (5,635) untouched — all are cancelled orders, which is valid business logic
- Kept `avg_prep_time_min` as a categorical bucket rather than forcing a numeric type
- Engineered row-level fields in Power Query: `Phase`, `Month_`, `Month_Num`, `Time_Bucket`, `Order_Date`, `City_Tier`, `Delivery_Delay_Mins`

---

## 🧮 DAX Highlights

A few of the more interesting patterns used:

**Phase-based comparison with `CALCULATE`**
```dax
PreCrisis Orders = CALCULATE([Total Orders], fact_orders[Phase] = "Pre-Crisis")
Crisis Orders    = CALCULATE([Total Orders], fact_orders[Phase] = "Crisis")
Order Percent Decline = DIVIDE([PreCrisis Orders] - [Crisis Orders], [PreCrisis Orders])
```

**Monthly-average comparison (periods have unequal length)**
```dax
Order Freq Drop % =
VAR PreCrisisMonthly = DIVIDE(CALCULATE([Total Orders], fact_orders[Phase] = "Pre-Crisis"), 5)
VAR CrisisMonthly    = DIVIDE(CALCULATE([Total Orders], fact_orders[Phase] = "Crisis"), 4)
RETURN DIVIDE(PreCrisisMonthly - CrisisMonthly, PreCrisisMonthly)
```

**VIP segmentation with a global percentile threshold**
```dax
VIP_Spend_Threshold = PERCENTILEX.INC(ALL(dim_customer), dim_customer[PreCrisis_Spend], 0.95)
VIP_Flag = IF(dim_customer[PreCrisis_Spend] >= [VIP_Spend_Threshold], "VIP", "Regular")
```

**Composite recovery estimate**
```dax
Recovery Potential =
VAR RevenueLost       = [Revenue Lost]
VAR RetentionStrength = 1 - DIVIDE([Churned Customers], [Active Customers])
VAR SentimentFactor   = DIVIDE([Avg Sentiment] + 1, 2)
RETURN RevenueLost * RetentionStrength * SentimentFactor
```

Concepts demonstrated: `CALCULATE`, context transition, `DIVIDE`, `SWITCH(TRUE())`, `VAR/RETURN`, `FILTER` + `AVERAGEX`, `ALL`, `TOPN`, `RELATEDTABLE` / `RELATED`, `DISTINCTCOUNT`, `PERCENTILEX.INC`, and a custom Date table.

---

## 🛠️ Tools & Skills

- **Power BI Desktop** — report design, navigation, bookmarks, conditional formatting, slicer sync
- **Power Query (M)** — cleaning, type fixes, derived columns
- **DAX** — measures, calculated columns, segmentation, time-phase analysis
- **Data modeling** — star schema, relationship design, cross-filter direction
- **Business analysis** — customer/restaurant segmentation, root-cause analysis, executive storytelling

---

## 📁 Repository Structure

```
quickbite-express-crisis-analysis/
│
├── data/                       # Raw CSV files (8 tables)
├── powerbi/
│   └── QuickBite_Express.pbix  # Power BI report
├── screenshots/                # Dashboard page images
├── docs/
│   ├── QuickBite_Express_Crisis_Recovery_Analysis.pptx   # Presentation deck
│   ├── QuickBite_Express_Presentation_Script.md          # Speaker script
│   └── QuickBite_Express_DAX_Interview_Guide.md          # DAX explanations
└── README.md
```

---

## ▶️ How to Use

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/quickbite-express-crisis-analysis.git
   ```
2. Open `powerbi/QuickBite_Express.pbix` in **Power BI Desktop**.
3. If prompted, update the data source path: **Home → Transform Data → Data source settings** → point to the local `data/` folder.
4. Click **Refresh** to reload the model.

---

## 📝 Notes & Assumptions

- **Crisis cutoff:** June 1, 2025 — consistent with the visible collapse in order volume and the ~2× jump in cancellation rate beginning that month.
- **Order-volume measures** at customer/restaurant level exclude cancelled orders; headline order counts and cancellation rate include them.
- **Restaurant threshold:** the problem statement references restaurants with ≥50 pre-crisis orders, but no restaurant in this dataset exceeds 21 pre-crisis orders, so a ≥10 threshold was used for the "top declining restaurants" analysis.
- `Risk_Level` and `Return_Segment` band thresholds are analyst-defined and documented in the DAX guide.

---

## 👤 Author

**[Neelamma]**
Data Analyst | Power BI • DAX • SQL


---

⭐ *If you found this project useful, consider giving it a star!*
