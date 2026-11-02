# Online Retail Customer Churn Prediction & Analytics

An end-to-end exploratory analytics and machine-learning project using the UCI Online Retail transaction dataset. The notebook moves from transaction-level data quality checks to business KPIs, customer behavior analysis, churn-label construction, model evaluation, and customer risk prioritization.

## Project objectives

- Understand sales, orders, customers, products, and geography.
- Measure monthly revenue, order volume, average order value (AOV), and new-versus-returning customer activity.
- Build customer-level behavioral features, including Recency, Frequency, Monetary value (RFM), Active Days, and Product Variety.
- Define churn as **no purchase during the next 90-day observation window**.
- Compare Logistic Regression, Random Forest, and Gradient Boosting.
- Produce churn-risk segments and a revenue-at-risk proxy to help prioritize retention analysis.
- Reproduce selected analytical questions using SQL (SQLite) as well as Pandas.

## Dataset

This project uses the **Online Retail** dataset from the UCI Machine Learning Repository:

- Dataset page: https://archive.ics.uci.edu/dataset/352/online+retail
- File expected by the notebook: `Online_Retail.xlsx`

The dataset is not included in this repository. Download it from UCI and place `Online_Retail.xlsx` in the same directory as the notebook. Review the dataset's terms and attribution guidance before redistribution.

## Repository structure

```text
online-retail-customer-churn/
├── online_retail_customer_churn.ipynb
├── README.md
├── requirements.txt
├── .gitignore
└── INTERVIEW_QUESTIONS.md
```

## Run locally

Recommended: Python 3.10 or 3.11.

```bash
python -m venv .venv
source .venv/bin/activate        # macOS/Linux
# .venv\Scripts\activate         # Windows

python -m pip install --upgrade pip
pip install -r requirements.txt
jupyter notebook
```

Open `online_retail_customer_churn.ipynb` and run the cells from top to bottom after placing `Online_Retail.xlsx` next to it.

## Workflow

1. **Load and inspect** — examine columns, data types, nulls, duplicates, and summary statistics.
2. **Clean data** — remove rows without a CustomerID and duplicate transactions; inspect negative quantities and zero prices.
3. **Feature engineering** — calculate line-item revenue as `Quantity × UnitPrice` and derive date/customer/product features.
4. **Business analytics** — analyze revenue, orders, AOV, monthly patterns, new and returning customers, RFM, products, and countries.
5. **Churn-label creation** — compute features only from transactions before a cutoff; label a customer as churned when no purchase is observed in the following 90 days.
6. **SQL validation** — reproduce selected top-customer, monthly-sales, and country-performance queries in SQLite.
7. **Modeling** — use a stratified 80/20 train/test split and compare Logistic Regression, Random Forest, and Gradient Boosting.
8. **Evaluation and interpretation** — report precision, recall, F1, ROC-AUC, confusion matrices, and model coefficients/feature importance.
9. **Risk prioritization** — group predicted probabilities into Low/Medium/High risk bands and rank customers for further review.

## Results recorded in the notebook

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Logistic Regression | 0.703 | 0.651 | 0.690 | 0.670 | **0.773** |
| Random Forest | 0.663 | 0.613 | 0.619 | 0.616 | 0.743 |
| Gradient Boosting | 0.675 | 0.626 | 0.633 | 0.629 | 0.761 |

These are the metrics stored in the current notebook run, not a guarantee that every environment or dataset version will reproduce them exactly. Logistic Regression has the highest recorded ROC-AUC in this experiment.

## Important modeling choices and limitations

- **Operational churn definition:** no observed purchase in the next 90 days. This is a project-specific definition, not a universal definition of churn.
- **Observation window:** the dataset covers a limited historical period. A 90-day label window near the end of the data may not generalize to other businesses or seasons.
- **Returns and cancellations:** negative quantities exist in the source data. The notebook inspects them, but the business interpretation of net revenue and returns should be documented carefully before production use.
- **Validation strategy:** the current notebook uses a stratified random customer-level train/test split. For stronger evidence, use a time-based validation design and compare performance across later cohorts.
- **Risk probabilities:** the notebook uses predicted probabilities to rank customers. Probability calibration should be checked before treating these values as true churn likelihoods.
- **Revenue at risk:** `Monetary × ChurnProbability` is a prioritization proxy, not a causal estimate of revenue that will be lost or saved.
- **Risk thresholds:** Low/Medium/High bands are rule-based thresholds (0.40 and 0.70), not business-optimized cutoffs.
- **Reproducibility:** this is a notebook-first project. The notebook should be rerun from a clean kernel before final publication.

## Possible next steps

- Add a time-based holdout and cross-validation.
- Tune classification thresholds against retention-team capacity and the cost of false positives/false negatives.
- Calibrate predicted probabilities and evaluate calibration curves/Brier score.
- Compare against a simple baseline and report class balance.
- Add a small dashboard or a separate executive summary with charts and business implications.
- Refactor reusable feature engineering and evaluation into Python modules and add tests.

## Author

Maintained by Manu Pradeep Kumar. See the profile linked from the GitHub account for contact and related projects.
