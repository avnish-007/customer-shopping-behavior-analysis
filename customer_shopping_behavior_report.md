# Customer Shopping Behavior Analysis — Report

## 1. Project Overview

This project is an end-to-end analysis of retail customer shopping behavior. It starts
from a raw transaction CSV, cleans and enriches it with Python (pandas), loads the
clean table into PostgreSQL, answers ten business questions with SQL, and visualizes
the results in a Power BI dashboard.

**Pipeline**

```
customer_shopping_behavior.csv
        │  (Python / pandas — cleaning + feature engineering)
        ▼
customer_shopping_behavior_analysis.ipynb
        │  (SQLAlchemy → PostgreSQL, database: customer_behavior, table: customer)
        ▼
customer_behavior_sql_queries.sql   (10 business queries, Q1–Q10)
        │  (results captured as q1.png … q10.png)
        ▼
customer_behavior_dashboard.pbix    (Power BI dashboard)
```

**Tools used:** Python 3.9+, pandas, Jupyter Notebook, PostgreSQL (via SQLAlchemy/psycopg2),
SQL, Power BI.

**Files in this repo**

| File | Purpose |
|---|---|
| `customer_shopping_behavior.csv` | Raw dataset, 3,900 rows × 18 columns |
| `customer_shopping_behavior_analysis.ipynb` | Cleaning, feature engineering, PostgreSQL load |
| `customer_behavior_sql_queries.sql` | Q1–Q10 business queries on the `customer` table |
| `customer_behavior_dashboard.pbix` | Power BI dashboard |
| `q1.png` … `q10.png` | Screenshots of each query's result |

---

## 2. Dataset Summary

- **Shape:** 3,900 rows × 18 columns. One row = one transaction.
- **Missing values:** only 37 nulls, all in `Review Rating` (later imputed).
- **No duplicates / type issues:** IDs run 1–3,900; numeric columns are clean.

**Columns**

| Column | Type | Range / Values |
|---|---|---|
| Customer ID | int | 1 – 3,900 |
| Age | int | 18 – 70 (mean 44.1) |
| Gender | category | Male (2,652), Female (1,248) |
| Item Purchased | category | 25 products (Blouse, Pants, Jewelry …) |
| Category | category | Clothing (1,737), Accessories (1,240), Footwear (599), Outerwear (324) |
| Purchase Amount (USD) | int | 20 – 100 (mean 59.76, total revenue **$233,081**) |
| Location | category | 50 US states |
| Size | category | S, M, L, XL |
| Color | category | 25 colors |
| Season | category | Spring (999), Fall (975), Winter (971), Summer (955) — almost even |
| Review Rating | float | 2.5 – 5.0 (mean 3.75; 37 nulls) |
| Subscription Status | Yes/No | No (2,847), Yes (1,053) |
| Shipping Type | category | 6 types, ~627–675 orders each — almost even |
| Discount Applied | Yes/No | No (2,223), Yes (1,677 → 43% of orders) |
| Promo Code Used | Yes/No | Identical to Discount Applied on every row (redundant) |
| Previous Purchases | int | 1 – 50 (mean 25.4) |
| Payment Method | category | 6 methods, ~612–677 orders each — almost even |
| Frequency of Purchases | category | 7 frequencies, ~539–584 orders each — almost even |

**Key observations from the raw data**

- Sales are evenly spread across seasons, states, shipping types, payment methods and
  purchase frequencies — no single channel dominates, which is good for analysis
  (comparisons are fair).
- Average order value is flat (~$57–$60) across all four categories and both genders,
  so revenue differences come from **volume**, not price.
- 43% of orders used a discount — large enough to measure discount impact reliably.
- `Promo Code Used` mirrors `Discount Applied` on all 3,900 rows, so it carries no
  extra information.

---

## 3. Exploratory Data Analysis Using Python

The notebook (`customer_shopping_behavior_analysis.ipynb`) works through these stages:

### 3.1 Loading and inspection
- Load the CSV with `pd.read_csv`.
- `head()`, `info()`, `describe(include='all')` to confirm 3,900 × 18, check dtypes
  (numeric: Customer ID, Age, Purchase Amount, Review Rating, Previous Purchases;
  rest categorical) and value ranges.

### 3.2 Missing-value treatment
- `isnull().sum()` finds exactly **37 missing Review Ratings** and nothing else.
- Fix: fill each null with the **median rating of its own category**
  (`groupby('Category').transform(lambda x: x.fillna(x.median()))`), which preserves
  category-level differences instead of flattening everything toward one global value.
- Re-check confirms zero nulls.

### 3.3 Column-name normalisation
- Lowercase + underscores, and `purchase_amount_(usd)` → `purchase_amount`, so the
  cleaned frame has database-friendly snake_case names.

### 3.4 Feature engineering
- **`age_group`** — age split into quartiles with `pd.qcut`, giving four balanced
  segments of ~940–1,030 customers each:
  Young_Adult (18–31), Adult (31–44), Middle-aged (44–57), Senior (57–70).
- **`purchase_frequency_days`** — the 7 text frequencies mapped to days
  (Weekly→7, Fortnightly/Bi-Weekly→14, Monthly→30, Quarterly/Every 3 Months→90,
  Annually→365) so buying frequency becomes numeric and comparable.

### 3.5 Redundancy removal
- Verified `(discount_applied == promo_code_used).all()` is `True`, then dropped
  `promo_code_used` — one fewer column, zero information lost.

### 3.6 Loading into PostgreSQL
- SQLAlchemy engine to a local `customer_behavior` database, `df.to_sql('customer',
  if_exists='replace')` — the cleaned table that every SQL query in Section 4 runs on.

---

## 4. Data Analysis Using SQL (Business Questions)

All queries run on the `customer` table in the `customer_behavior` database.
Total revenue across every query reconciles to **$233,081**.

### Q1. Total revenue: male vs. female customers

| Gender | Revenue |
|---|---|
| Male | 157,890 |
| Female | 75,191 |

**Insight:** Male customers generate ~68% of revenue — but that mirrors their ~68%
share of transactions (2,652 of 3,900). Average spend per order is nearly identical
($59.5 male vs. $60.3 female), so there is no gender pricing/spend gap; the revenue
gap is purely footfall.

### Q2. Discount users who still spent above the average order ($59.76)

839 of the 1,677 discount orders (about half) are at or above the overall average
purchase amount — e.g. customer 43 spent $100 with a discount applied.

**Insight:** Discounts are not confined to cheap baskets; high-value customers use them
too, so discount strategy reaches the customers who matter most.

### Q3. Top 5 products by average review rating

| Product | Avg rating |
|---|---|
| Gloves | 3.86 |
| Sandals | 3.84 |
| Boots | 3.82 |
| Hat | 3.80 |
| Skirt | 3.79 |

**Insight:** Ratings cluster tightly (3.79–3.86) — no runaway winner and no problem
product, with footwear/gloves slightly ahead. Quality looks consistent.

### Q4. Average order: Standard vs. Express shipping

| Shipping | Avg order |
|---|---|
| Standard | 58.46 |
| Express | 60.48 |

**Insight:** Express buyers spend ~$2 more per order — customers paying for speed also
buy slightly bigger baskets.

### Q5. Do subscribed customers spend more?

| Subscription | Customers | Avg spend | Total revenue |
|---|---|---|---|
| No | 2,847 | 59.87 | 170,436 |
| Yes | 1,053 | 59.49 | 62,645 |

**Insight:** No — average spend is virtually identical ($59.87 vs. $59.49). Subscribers
are only 27% of customers and contribute 27% of revenue, so the subscription program
currently drives reach, not higher baskets.

### Q6. Top 5 products by discount usage rate

| Product | Discount rate |
|---|---|
| Hat | 50.00% |
| Sneakers | 49.66% |
| Coat | 49.07% |
| Sweater | 48.17% |
| Pants | 47.37% |

**Insight:** Roughly every second Hat/Sneakers/Coat order uses a discount — these
categories are the most promotion-sensitive and the best candidates for targeted offers.

### Q7. Customer segmentation by previous purchases

| Segment | Definition | Customers |
|---|---|---|
| Loyal | > 10 previous purchases | 3,116 (~80%) |
| Returning | 2 – 10 | 701 (~18%) |
| New | 1 | 83 (~2%) |

**Insight:** The base is overwhelmingly loyal — retention is strong, but only 2% are
genuinely new, so acquisition (not retention) is the growth bottleneck.

### Q8. Top 3 products by orders within each category

| Category | #1 | #2 | #3 |
|---|---|---|---|
| Accessories | Jewelry (171) | Belt (161) | Sunglasses (161) |
| Clothing | Blouse (171) | Pants (171) | Shirt (169) |
| Footwear | Sandals (160) | Shoes (150) | Sneakers (145) |
| Outerwear | Jacket (163) | Coat (161) | — (only 2 products) |

**Insight:** Clear hero products per category (Jewelry, Blouse/Pants, Sandals, Jacket).
Outerwear has just two products (Jacket, Coat) — the thinnest assortment and a
candidate for range expansion.

### Q9. Are repeat buyers (> 5 previous purchases) likely to subscribe?

| Subscription | Repeat buyers |
|---|---|
| No | 2,518 |
| Yes | 958 |

**Insight:** No — of 3,476 repeat buyers, only ~28% subscribe, almost the same as the
overall 27% subscription rate. Buying often does not convert customers into
subscribers, so the subscription pitch needs its own incentive.

### Q10. Revenue contribution by age group

| Age group | Revenue |
|---|---|
| Young_Adult (18–31) | 62,143 |
| Middle-aged (44–57) | 59,197 |
| Adult (31–44) | 55,978 |
| Senior (57–70) | 55,763 |

**Insight:** Young adults lead revenue (~27%) despite similar group sizes — the
highest value-per-customer segment and the one to prioritise in marketing.

---

## Key Takeaways

1. **Volume, not price, drives revenue** — order values are flat across categories,
   genders and subscriber status.
2. **Loyal base, weak acquisition** — 80% loyal vs. 2% new customers.
3. **Subscription doesn't lift baskets** — subscribers spend the same per order as
   everyone else, and even heavy repeat buyers rarely subscribe.
4. **Discounts reach high spenders** — half of discounted orders are above-average
   value; Hat/Sneakers/Coat are the most promotion-sensitive products.
5. **Young adults are the top segment** — highest revenue of the four age groups.
