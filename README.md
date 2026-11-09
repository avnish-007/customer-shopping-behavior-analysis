# Customer Shopping Behavior Analysis

Exploratory data analysis of 3,900 retail transactions — data cleaning, feature
engineering and insights on purchasing patterns, discounts and subscriptions.

## Dataset

`customer_shopping_behavior.csv` — 3,900 rows × 18 columns of retail transactions.

| Column | Description |
|---|---|
| Customer ID | Unique customer identifier |
| Age | Customer age |
| Gender | Male / Female |
| Item Purchased | Product name |
| Category | Clothing, Accessories, Footwear, Outerwear |
| Purchase Amount (USD) | Transaction value in USD |
| Location | US state |
| Size, Color | Product attributes |
| Season | Spring, Summer, Fall, Winter |
| Review Rating | Rating out of 5 |
| Subscription Status | Whether the customer has a subscription |
| Shipping Type | Delivery method chosen |
| Discount Applied, Promo Code Used | Whether a discount/promo was used |
| Previous Purchases | Count of prior purchases |
| Payment Method | Payment instrument |
| Frequency of Purchases | How often the customer buys |

## What the notebook does

`customer_shopping_behavior_analysis.ipynb`

1. Load the CSV with pandas
2. Inspect shape, dtypes, summary statistics and missing values
3. Fill the 37 missing `Review Rating` values with the per-category median
4. Normalise column names to snake_case (`purchase_amount_(usd)` → `purchase_amount`)
5. Engineer `age_group` from age quartiles (Young_Adult, Adult, Middle-aged, Senior)
6. Engineer `purchase_frequency_days` by mapping purchase frequency to days
7. Verify `discount_applied` equals `promo_code_used` for every row and drop the
   redundant `promo_code_used` column

## Getting started

```bash
git clone https://github.com/avnish-007/customer-shopping-behavior-analysis.git
cd customer-shopping-behavior-analysis
pip install pandas jupyter
jupyter notebook customer_shopping_behavior_analysis.ipynb
```

## Requirements

- Python 3.9+
- pandas, Jupyter

## Status

The notebook currently covers data loading and cleaning. Next steps: revenue by
category/season/state, subscription vs non-subscription behaviour, discount impact
and customer segmentation.

## License

Released under the [MIT License](LICENSE).

## Acknowledgements

This is a personal learning project. The code and analysis were written while
following data analysis tutorials on YouTube, and the dataset is a public dataset
used for practice — it is not my original work or a commercial product.
