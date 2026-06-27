# Bank Customer Churn Analysis
### End-to-End Data Analytics Project | Snowflake · Power BI · DAX · SQL

---

## Project Overview

This project analyzes customer churn behavior for a bank dataset of 10,000 customers.
The goal was to identify churn drivers, classify customers into behavioral segments,
build a data-driven risk scoring model, and deliver a three-page interactive Power BI
report for business stakeholders.

**Overall churn rate: 20.37% | Balance lost to churn: €185.59M**

---

## Tech Stack

| Layer | Tool |
|---|---|
| Cloud Storage | AWS S3 |
| Data Warehouse | Snowflake |
| Data Preparation | SQL (Snowflake) |
| Visualization | Power BI Desktop |
| Business Logic | DAX |

---

## Data Pipeline

Raw data was stored in an AWS S3 bucket. A secure storage integration was created
between Snowflake and S3 using IAM role-based authentication. An external stage was
configured pointing to the S3 bucket, and the CSV file was loaded directly into a
Snowflake table using the COPY INTO command.

---

## Data Preparation

### Bucketing

Continuous variables were converted into categorical ranges to enable meaningful
grouping and visualization:

- **Age** was grouped into six bands: 18–24, 25–34, 35–44, 45–54, 55–64, and 65+
- **Tenure** was classified into four lifecycle stages: New Customer (0–2 years),
  Early Stage (3–5 years), Established (6–8 years), and Loyal (9–10 years)
- **Balance** was split into Zero Balance, Below 50K, 50K–100K, 100K–150K,
  150K–200K, and 200K+
- **Estimated Salary** was grouped into four equal quartile bands
- **Credit Score** was bucketed using standard FICO ranges: Poor, Fair, Good,
  Very Good, and Excellent

### Status Labels

Three binary numeric columns were converted into readable text labels for
dashboard display — Active/Inactive for membership status, Exited/Not Exited
for churn status, and Yes/No for credit card holding.

---

## Risk Scoring Model

A composite risk score was built entirely from the actual churn rates observed
in the data. Rather than assigning arbitrary point values, each variable
contributes its deviation from the overall churn rate of 20.37% — meaning
only variables that genuinely differ from the average add positive or negative
risk. The contributions from six variables (Geography, Activity Status,
Number of Products, Gender, Balance, and Age) are summed and normalized
to a 0–100 scale.

Customers are then classified into four risk tiers based on their score:

| Risk Category | Churn Rate |
|---|---|
| Critical High | 100.00% |
| High | 91.13% |
| Medium | 38.76% |
| Low | 7.75% |

The churn rates increase cleanly across all four tiers, confirming the
model is predictive and not decorative.

---

## Customer Segmentation

Eight behavioral personas were defined using a combination of activity status,
number of products held, account balance, and tenure. The goal was to move
beyond demographic grouping and classify customers by what they actually do.

| Persona | Description | Churn Rate |
|---|---|---|
| Committed | Active, 2 products, balance held, tenure ≥ 4 years | 5.38% |
| Developing | Active, 2 products, balance held, tenure < 4 years | 5.87% |
| Growth Potential | Active, 1 product, balance held | 17.36% |
| Dormant | Inactive, low to mid balance | 18.96% |
| Standard | Active, zero balance (catch-all) | 26.62% |
| High-Value Silent | Inactive, high balance (≥97K) | 29.62% |
| Product-Heavy | Active, 3 or more products | 80.28% |
| Passive Multi-Product | Inactive, 3 or more products | 90.22% |

Persona logic was ordered intentionally — inactive customers with 3+
products are classified as Passive Multi-Product before the Product-Heavy
condition is applied, correctly separating active and inactive over-banked
customers who show distinctly different churn behaviour.

---

## DAX Measures and Calculated Columns

Four DAX calculations were built in Power BI:

- **Churn Rate** — percentage of customers who exited out of total customers
- **Non Churn Rate** — percentage of customers who were retained
- **Bar Color** — a conditional color measure applied to churn charts; bars
  showing 50%+ churn display in red, 25–49% in navy, and below 25% in grey.
  Color is used only on the churn page where bars represent risk levels,
  not on overview pages where bars represent customer volume distribution
- **Value Tier** — classifies each customer into Premium, Untapped Wealth,
  Income Rich, or Standard based on their balance and salary combination
- **Product Opportunity** — classifies each customer's product relationship
  as Cross-Sell, Well Penetrated, Multi-Product, or Limited Adoption based
  on balance and number of products held

---

## Dashboard Structure

The report is structured across three pages, each serving a distinct audience
and analytical purpose.

### Page 1 — Customer Profile Overview
Provides a demographic and distributional view of the entire customer base.
Covers geography, age, bank balance, credit score, and the relationship between
activity status and exit status. Intended for general stakeholder understanding
of who the bank's customers are.

### Page 2 — Customer Segmentation & Opportunities
Presents the behavioral segmentation model. Shows persona distribution, a
validation table confirming churn rates per persona, value tier breakdown,
product opportunity classification, and exit risk distribution. Intended for
strategy and CRM teams to identify which customer groups need attention and
what kind of intervention suits each.

### Page 3 — Customer Churn Analysis
Focuses on churn drivers and financial impact. Shows churn rates broken down
by geography, activity status, products held, age, and balance. Includes the
exit risk validation table and retention recommendations. Intended for retention
teams and senior management.

---

## Key Findings

- Germany churns at **32.44%** despite holding the highest-value customers —
  65.84% of German customers are in the 100K–150K balance band
- Churn peaks at ages **55–64 (49.83%)** — nearly 1 in 2 customers exits
- **2 products is the safest cohort at 7.58% churn**; 3 products jumps to
  82.71% and 4 products reaches 100%
- **Passive Multi-Product** customers are the highest-churn persona at 90.22%
- Churned customers hold a higher average balance (€91K) than retained
  customers (€72K) — the bank is losing its wealthier customers at a
  disproportionate rate
- **€185.59M** in balance lost to churn; €579.27M retained
- Tenure, estimated salary, and credit score show no meaningful variation
  in churn and were excluded as retention signals

---

## Retention Recommendations

- Prioritise inactive + high balance customers — outreach before they exit
- Germany needs a dedicated retention strategy — service quality and
  product fit review, not a generalised campaign
- Customers with 3+ products need immediate portfolio review —
  over-selling is the strongest churn trigger in the dataset
- Cross-sell strategy should target moving 1-product customers to exactly
  2 products — the optimal retention point
- Strengthen Early Stage onboarding — 37.61% of High Risk customers
  are new customers disengaging before building loyalty
- Investigate the female churn gap — female churn (25.07%) significantly
  exceeds male (16.46%), warranting a review of product relevance
  and service experience by gender
- Tenure, salary, and credit score should not be prioritised in retention
  efforts — the data shows no churn variation across these variables
