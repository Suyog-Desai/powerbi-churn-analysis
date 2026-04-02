# Customer Churn Analysis — Power BI Dashboard

An interactive Power BI dashboard analyzing telecom customer churn across 7,043 customers and $16M in revenue. Built to help business stakeholders move away from static reports and actually understand *why* customers leave — and which ones are most at risk.

📊 [View Dashboard (PDF)](Churn_Telecom.pdf) &nbsp;|&nbsp; 📁 [Download Report (.pbix)](Churn_Telecom.pbix) &nbsp;|&nbsp; 📂 [Dataset (.xlsx)](Customer%20Churn-Dataset.xlsx) &nbsp;|&nbsp; 💡 [DAX & Power Queries](Dax%26PowerQueries.txt)

---

## Dashboard Overview

| Metric | Value |
|--------|-------|
| Total Customers | 7,043 |
| Overall Churn Rate | **26.5%** |
| Total Revenue | $16M |
| Avg. Tenure | 32.37 months |
| Avg. Monthly Charges | $65 |

---

## What the Dashboard Shows

**Page 1 — Customer Overview**

The main page gives a full picture of the customer base with slicers for Gender, Senior Citizen, Contract Type, and Dependents — all cross-filtering every visual simultaneously.

- **Customer Distribution by Tenure** — customers bucketed into 6 tenure bands. The largest group is `< 1 Year` (2,069 customers), which is also where churn risk is highest
- **Customer Distribution by Total Services** — how many add-on services each customer subscribes to. Customers with 0 services churn at a significantly higher rate
- **Payment Method Breakdown** — Electronic check leads at 33.58%, followed by Mailed check (22.89%), Bank transfer (21.92%), Credit card (21.61%)
- **Service Subscription Summary** — side-by-side view of Yes/No/No Internet Service across Device Protection, Online Backup, Online Security, Streaming TV, and Streaming Movies

**Page 2 — Churn Deep Dive**

Filters lock to churned customers only, flipping all KPI cards and visuals to show churned segment metrics — revenue lost, tenure patterns, and service gaps among churned users.

---

## Data Model & DAX

The interesting part of this project wasn't the visuals — it was the data modeling underneath.

**Custom Calculated Table — ServiceStatusSummary**

Rather than analyzing each service column separately, I unpivoted all 5 service columns into a single normalized table using `UNION` + `SELECTCOLUMNS`:

```dax
ServiceStatusSummary = 
UNION(
    SELECTCOLUMNS('Churn-Dataset', "customerID", 'Churn-Dataset'[customerID],
        "Service", "Device Protection", "Status", 'Churn-Dataset'[DeviceProtection], 
        "Churn", 'Churn-Dataset'[Churn]),
    SELECTCOLUMNS('Churn-Dataset', "customerID", 'Churn-Dataset'[customerID],
        "Service", "Online Security", "Status", 'Churn-Dataset'[OnlineSecurity], 
        "Churn", 'Churn-Dataset'[Churn]),
    -- ... repeated for each service
)
```

This made it possible to filter all services by churn status in a single visual, instead of maintaining 5 separate measures.

**Churn Rate Measure**
```dax
Churn% = DIVIDE(ServiceStatusSummary[Churn_Count], ServiceStatusSummary[Count_of_customers], 0)
```

**Dynamic Title Cards** — KPI card titles change based on slicer selection:
```dax
customer_title = 
VAR Churn_selected = SELECTEDVALUE('Churn-Dataset'[Churn])
RETURN
SWITCH(TRUE(),
    Churn_selected = "Yes", "Churned Customers",
    Churn_selected = "No",  "Active Customers",
    "Total Customers"
)
```

Same pattern applied to the Revenue header — one measure drives both the number and the label dynamically.

**Tenure Bucketing — Power Query**
```m
= Table.AddColumn(#"Replaced Value", "Tenure_year",
    each if [tenure] < 12 then "< 1 Year"
    else if [tenure] < 24 then "1–2 Years"
    else if [tenure] < 36 then "2–3 Years"
    else if [tenure] < 48 then "3–4 Years"
    else if [tenure] < 60 then "4–5 Years"
    else "5+ years")
```

**TotalServices Calculated Column**
```dax
TotalServices = 
IF('Churn-Dataset'[OnlineSecurity] = "Yes", 1, 0) +
IF('Churn-Dataset'[OnlineBackup] = "Yes", 1, 0) +
IF('Churn-Dataset'[DeviceProtection] = "Yes", 1, 0) +
IF('Churn-Dataset'[TechSupport] = "Yes", 1, 0) +
IF('Churn-Dataset'[StreamingTV] = "Yes", 1, 0) +
IF('Churn-Dataset'[StreamingMovies] = "Yes", 1, 0)
```

---

## Key Insights

- Customers with `< 1 Year` tenure are the largest and most at-risk segment — early engagement programs would have the highest ROI here
- Customers with **0 add-on services** (2,200+) are prime churn candidates — no stickiness beyond the base plan
- **Electronic check** users represent the highest payment method share AND correlate with higher churn — likely month-to-month contract overlap
- Online Security has the lowest adoption (2,019 Yes vs 3,498 No) — a targeted upsell campaign here could improve both retention and revenue

---

## Tech Stack

- **Power BI Desktop** — data modeling, DAX, dashboard design
- **Power Query (M)** — data transformation and feature engineering
- **DAX** — calculated columns, measures, dynamic titles, calculated tables
- **Dataset** — Telecom customer churn dataset (7,043 records)

---

## Project Structure

```
powerbi-churn-analysis/
├── Churn_Telecom.pbix              # Power BI report file
├── Churn_Telecom.pdf               # Dashboard export (static view)
├── Customer Churn-Dataset.xlsx     # Raw dataset
├── Dax&PowerQueries.txt            # All DAX measures and Power Query code
└── README.md
```

---

## What I'd Add Next

- Predictive churn score using Python integration in Power BI (sklearn logistic regression)
- Row-level security so regional managers see only their customer segment
- Automated data refresh connected to a live SQL source
- Cohort analysis tracking churn rates by signup month

---

## Author

**Suyog Desai** — [GitHub](https://github.com/Suyog-Desai) · [LinkedIn](https://linkedin.com/in/suyog-desai) · [Portfolio](https://suyogdesai.framer.website)
