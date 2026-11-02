# Interview Questions — Online Retail Customer Churn Prediction & Analytics

Use these as practice prompts. Keep answers consistent with what the notebook actually implements; distinguish current work from future improvements.

## 1. Project overview

1. **Explain your project in 60–90 seconds.**  
   Structure: business problem → dataset → cleaning and analysis → churn definition/features → models and evaluation → how risk scores could support retention prioritization.

2. **Why did you choose this project?**  
   It connects transaction analytics with a practical customer-retention use case and demonstrates SQL, Python, feature engineering, and model evaluation.

3. **Who would use the output?**  
   A retention/CRM or business analytics team could review high-risk customers and combine risk with customer value before deciding on outreach.

4. **What is the business problem?**  
   Identify customers who are likely to stop purchasing, so a business can investigate and prioritize retention actions.

5. **What are the main limitations?**  
   Limited historical period, project-specific churn definition, returns/cancellations, no causal retention experiment, and the need for time-based validation and probability calibration.

## 2. Dataset and data cleaning

6. **What is the grain of the raw dataset?**  
   A transaction line/item record, not one row per customer or necessarily one row per order.

7. **What are the important columns?**  
   `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, and `Country`.

8. **Why remove rows with missing CustomerID?**  
   Customer-level aggregation and churn labels require a customer identifier. Removing them makes the customer analysis possible, but it can introduce selection bias because anonymous transactions are excluded.

9. **Why remove duplicates?**  
   Exact duplicate rows can inflate revenue, quantities, and order/product counts. First verify what counts as a duplicate and whether repeated lines could be legitimate.

10. **What do negative quantities mean?**  
    They may represent returns or cancellations. They should not automatically be treated as ordinary sales; define whether the analysis uses gross sales, net sales, or excludes cancellations.

11. **How is revenue calculated?**  
    `Revenue = Quantity × UnitPrice`. Explain that this is line-item revenue and that returns can make it negative.

12. **Why inspect zero UnitPrice values?**  
    They can represent free items or data-quality/business exceptions and may affect revenue-per-unit calculations.

13. **How would you prevent data-quality issues from silently changing results?**  
    Add assertions, document row counts before/after each filter, inspect distributions, and keep a repeatable cleaning pipeline.

## 3. Exploratory data analysis and business metrics

14. **What is AOV?**  
    Average order value = total revenue / number of distinct orders for the same period and scope.

15. **Why use `nunique()` for InvoiceNo?**  
    An invoice can have multiple line items, so counting rows would overcount orders.

16. **How would you calculate monthly revenue in SQL?**  
    Group by a month key derived from `InvoiceDate`, then sum `Revenue`; count distinct invoices for orders.

17. **Why compare revenue and revenue per customer by country?**  
    Total revenue reflects market size; revenue per customer gives a complementary view of customer economics. Small customer counts can make per-customer values volatile.

18. **How do new and returning customers differ?**  
    New customers make their first observed purchase in the relevant period; returning customers purchased earlier and purchase again in the period. Explain how “first observed” is constrained by the dataset's date range.

19. **What is RFM analysis?**  
    Recency is time since the most recent purchase, Frequency is purchase/order frequency, and Monetary is customer spend. These features summarize different dimensions of customer behavior.

20. **What business actions might follow from the EDA?**  
    Investigate high-value customers whose recency is rising, identify products driving revenue, and compare markets using both total and per-customer measures. Treat these as hypotheses to test, not proven causal effects.

## 4. Churn definition and feature engineering

21. **How did you define churn?**  
    A customer is labeled churned if they have no observed order in the 90-day period after the feature cutoff.

22. **Why use a future inactivity window?**  
    It makes the target operational and allows features to be computed only from information available before the prediction cutoff.

23. **What are the model features?**  
    Recency, Frequency, Monetary, AvgOrderValue, TotalUnits, ActiveDays, ProductVariety, OrdersPerActiveDay, and RevenuePerUnit.

24. **What is data leakage? Give an example from this project.**  
    Leakage occurs when information unavailable at prediction time is used as a feature. `FutureOrders` is used only to create the label and must not be included in `X`.

25. **Why exclude CustomerID from the features?**  
    It is an identifier rather than a behavioral measure and may encourage memorization without useful generalization.

26. **Why use a 90-day window?**  
    It is a chosen business rule for this project. In a real deployment, the window should reflect the typical repurchase cycle and business requirements.

27. **What is a weakness of this churn label?**  
    A customer may purchase after the observed window or be seasonal; no purchase in 90 days does not prove permanent churn.

28. **Why is the dataset period important?**  
    It limits the available feature and label windows and may include seasonal effects. The model may not generalize to other time periods.

## 5. Machine learning and evaluation

29. **Why start with Logistic Regression?**  
    It is a strong, interpretable baseline, trains quickly, and provides coefficients and probabilities.

30. **Why scale features for Logistic Regression?**  
    Features have different units and ranges. Scaling helps optimization and makes coefficient magnitudes more comparable, though coefficients still need careful interpretation.

31. **Why use median imputation?**  
    It fills missing numeric feature values with a robust central value. Fit the imputer only on training data; the pipeline does this.

32. **Why stratify the train/test split?**  
    It helps preserve the churn/non-churn class proportions in both partitions.

33. **Why compare Random Forest and Gradient Boosting?**  
    They can capture nonlinear relationships and feature interactions that a linear model may miss.

34. **What does ROC-AUC = 0.773 mean?**  
    On this test set, the model has about a 0.773 probability of ranking a randomly selected churn-positive customer above a randomly selected non-churn customer. It is not 77.3% accuracy.

35. **Why is accuracy not enough?**  
    It can hide poor performance on one class, especially with imbalance. Precision, recall, F1, ROC-AUC, the confusion matrix, and business costs give a fuller view.

36. **What is precision?**  
    Of customers predicted as churned, the fraction who are actually labeled churned.

37. **What is recall?**  
    Of customers actually labeled churned, the fraction the model identifies.

38. **Which error is more costly: false positive or false negative?**  
    It depends on campaign cost and customer value. False positives waste outreach; false negatives miss customers who may need intervention. The business should choose thresholds using these costs and campaign capacity.

39. **Which model performed best in the current notebook?**  
    Logistic Regression had the highest recorded ROC-AUC (0.773), compared with 0.743 for Random Forest and 0.761 for Gradient Boosting. This is a result from the current experiment, not proof it will always be best.

40. **How would you improve model validation?**  
    Use a time-based holdout, cross-validation where appropriate, compare with a simple baseline, tune hyperparameters, inspect calibration, and test on a later period.

41. **Why should the imputer/scaler be inside a Pipeline?**  
    The transformations are fitted on training data and applied consistently to test/new data, reducing preprocessing leakage.

42. **How do you interpret Logistic Regression coefficients?**  
    A positive coefficient increases the log-odds of the positive class and a negative coefficient decreases them, holding other features constant. With scaled features, the coefficient refers to a one-standard-deviation feature change.

## 6. Risk scoring and business application

43. **How are Low/Medium/High risk groups created?**  
    The notebook bins predicted churn probabilities at 0.40 and 0.70. These are chosen thresholds, not optimized or calibrated business cutoffs.

44. **What does RevenueAtRisk mean in your notebook?**  
    It is computed as historical Monetary value multiplied by predicted churn probability. It is a prioritization proxy—not a reliable forecast of revenue loss or revenue recoverable through a campaign.

45. **How would you prioritize customers?**  
    Consider both churn risk and customer value, then account for contact cost, eligibility, campaign capacity, and likely treatment effectiveness.

46. **How would you prove a retention campaign works?**  
    Run a randomized holdout/A-B test among eligible customers and compare incremental outcomes such as repeat purchase rate, revenue, or margin—not just model accuracy.

47. **How would you monitor the model after deployment?**  
    Track feature/data drift, score distributions, precision/recall and calibration once labels mature, campaign outcomes, and performance by customer segment/time.

## 7. SQL, Python, and implementation

48. **Why use SQL if Pandas can do the same analysis?**  
    SQL is useful when data lives in a relational database and filtering/aggregation should happen close to the data. Pandas is useful for flexible in-memory analysis and modeling. The tools complement each other.

49. **Why use `COUNT(DISTINCT InvoiceNo)` instead of `COUNT(*)`?**  
    The data is at line-item level, so one order may occupy multiple rows.

50. **What is the purpose of SQLite in this notebook?**  
    It demonstrates the same analytical questions using SQL on a local in-memory table. It does not demonstrate querying a production database.

51. **How would you make this project production-ready?**  
    Separate ingestion, cleaning, feature creation, training, scoring, and reporting into modules; add tests and logging; version data/model artifacts; schedule scoring; and define monitoring and retraining criteria.

52. **What would you do if the model's performance dropped?**  
    Check data/schema changes, missingness, feature and score drift, label definition changes, and segment-level performance before retraining or changing thresholds.

## Interview honesty checklist

- Say **“the notebook reports ROC-AUC of 0.773”**, not “77.3% accuracy.”
- State that the notebook uses a **stratified random customer-level split**, and describe time-based validation as an improvement unless you implement it.
- Explain that `RevenueAtRisk` is a **proxy** based on historical Monetary value and predicted probability.
- Do not claim a deployed retention system or measured revenue uplift unless you have actually implemented and measured it.
- Be ready to explain how negative quantities, cancellations, and missing CustomerIDs affect the analysis.
