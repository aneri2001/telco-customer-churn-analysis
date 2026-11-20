# Customer Churn Analysis: Telco One

End-to-End Business Analytics, Carnegie Mellon University (Sep – Oct 2026). Solo project. Telco One is a simulated client: the course framed the case as a real engagement, but no company commissioned it.

## The problem

Telco One's executives each wanted something different from a churn program:

| Stakeholder | Goal |
|---|---|
| CEO | Grow and protect the subscriber base against rival carriers |
| CFO | Maximize revenue while retaining churning users |
| CMO | Protect brand loyalty and reduce the churn rate |

The common goal was to **identify at-risk subscribers early and act on them, at a cost the business can justify**. That meant three questions: who churns, what drives it, and how many people to contact.

## How I approached it

**1. Understand the customers before modelling them** (`notebooks/01_eda_pca_clustering.ipynb`, `decks/update-1-personas.pdf`)
- Ran EDA on 7,043 subscribers and 20 attributes. Tenure and contract type showed the clearest churn signal.
- Used PCA to see which dimensions separate customers (spend and service usage), then K-means to group them into 4 personas:

| Persona | Share | Churn rate |
|---|---|---|
| Legacy phone subscribers | 21.6% | 7.4% |
| Fiber newcomers (price-sensitive) | 30.6% | 53.2% |
| Bundle DSL users | 23.9% | 25.3% |
| Loyalists | 23.8% | 11.0% |

**2. Build a model the business can act on** (`notebooks/02_churn_models.ipynb`, `decks/update-2-models.pdf`)
- Removed redundant or collinear features using VIF: TotalCharges (tenure × monthly charges), MonthlyCharges and NumServices, and one dummy level per category to avoid the dummy-variable trap.
- Compared logistic regression, a decision tree (depth tuned on a validation set) and a random forest. I also tested down-sampling for the 26.5% class imbalance.

| Model | Accuracy | Precision | Recall | AUC |
|---|---|---|---|---|
| Logistic regression | 0.798 | 0.636 | 0.562 | 0.842 |
| Random forest | 0.793 | 0.678 | 0.417 | 0.844 |
| Decision tree | 0.795 | 0.620 | 0.594 | 0.828 |
| Down-sampled tree | 0.734 | 0.500 | 0.773 | 0.829 |

- **Decision:** logistic regression. Its AUC is within 0.002 of the random forest, its coefficients explain *why* a customer is at risk, and its probabilities feed the cost model directly.

**3. Turn probabilities into a decision** (`decks/update-3-cutoff-and-roi.pdf`)
- Valued each flagged customer. Revenue saved = 30% take-up × 12 months × monthly bill. Cost = $10 call + 30% × $75 offer.
- Used Excel Solver to find the cutoff that best met each executive's goal: most customers saved (CEO), highest net value (CFO), highest precision with at least 75% recall (CMO).

## Result

Recommended **logistic regression with a 0.2 cutoff**. On the 1,407-customer test set it projected:
- $68.7K net value
- 4.1× ROI
- 88% of churners caught
- projected churn lowered from 26.6% to 19.6%

Sensitivity analysis on take-up: break-even is about 2.7%, and 0.2 stays the best cutoff for any take-up from 15% to 50%.

## Risks and what I'd do next

- **Take-up is assumed.** Pilot with a random contact group and a held-out control group to measure the real take-up rate.
- **The cutoff was tuned on the test set, so the results are likely slightly optimistic.** Re-tune on finer steps (0.15 to 0.25) with fresh data.
- **Target by persona.** Start the pilot with fiber newcomers, the highest-churn persona; for example, offer them a contract lock.
- **Monitor monthly.** Track churn, take-up and cost per saved customer, and retrain as behavior shifts.

## Data

The notebooks use the public IBM Telco Customer Churn dataset (7,043 customers), in a course-prepared version (`telco-churn-kpi.csv`). The dataset is not included in this repo. To run the notebooks, download it from [Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) and save it next to the notebooks.

**Tools:** Python (pandas, scikit-learn, statsmodels, matplotlib), Excel Solver
