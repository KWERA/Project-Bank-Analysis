# Bank Marketing: Predicting Term-Deposit Subscription

**DSC8307 Data Mining, Modeling and Analytics, Project 1**

This project builds and evaluates models that predict whether a bank customer will subscribe to a term deposit after a telephone marketing call. The goal is to help the bank choose which customers to call in future campaigns, so that more subscribers are reached with fewer wasted calls.

## Dataset
- **Size:** 41,188 calls, 20 input features, 1 target
- **Features:** client data (age, job, marital status, education, default, housing loan, personal loan), current campaign contact (contact type, month, day of week, number of calls), previous campaigns (pdays, previous, poutcome) and economic indicators (employment variation rate, consumer price index, consumer confidence index, 3-month Euribor rate, number of employees)
- **Target:** `y`, whether the customer subscribed (yes or no). Only 11.3% of calls led to a subscription, so the classes are imbalanced.

## Repository structure

```
.
├── README.md
├── requirements.txt
├── data/
│   └── bank-additional-full.csv
├── notebooks/
│   └── banking_analysis_final.ipynb
├── figures/                      (created when the notebook runs)
└── reports/
    └── Project_1_Bank_Analysis_Report.pdf
```

## How to run

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
pip install -r requirements.txt
jupyter notebook notebooks/banking_analysis_final.ipynb
```

Run all cells from top to bottom. The notebook reads the data from `data/bank-additional-full.csv` (semicolon-separated) and saves its charts to a `figures/` folder.

## Method

1. **Exploration.** Checked duplicates, "unknown" values, the `pdays = 999` placeholder, the target balance and how subscription rates vary by previous outcome, contact type, month, age and number of calls.
2. **Preparation.**
   - Removed 12 exact duplicate rows (41,176 rows remained).
   - Dropped `duration`, because call length is only known after the call. Keeping it would cause target leakage.
   - Replaced `pdays` with a yes/no flag, `prev_contacted`, because 999 is a code rather than a number of days.
   - Kept "unknown" as its own category, since the missing information may itself be a signal.
   - One-hot encoded categorical features and standardised numerical features for logistic regression, all inside pipelines to prevent leakage.
   - Made a stratified 80/20 train/test split (32,940 train, 8,236 test).
3. **Baseline.** A majority-class model that always predicts "no".
4. **Models.** Logistic regression, random forest and gradient boosting, scored with 5-fold stratified cross-validation and once on the test set. PR-AUC was the main metric because it focuses on the rare subscribers.
5. **Check on later data.** Trained on the first 80% of calls and tested on the last 20%, to see how the models hold up as economic conditions change.
6. **Decision threshold.** Compared thresholds using cross-validated predictions and an example cost (5 units per call, 50 units per subscriber).

## Results

| Model | CV ROC-AUC | CV PR-AUC | Test ROC-AUC | Test PR-AUC | ROC-AUC on later data |
| --- | --- | --- | --- | --- | --- |
| Gradient boosting | 0.800 | 0.465 | 0.812 | 0.491 | 0.632 |
| **Random forest** | **0.795** | **0.467** | **0.805** | **0.484** | **0.739** |
| Logistic regression | 0.790 | 0.451 | 0.801 | 0.453 | 0.748 |
| Majority baseline | 0.500 | 0.113 | 0.500 | 0.113 | |

**Key findings**

- All three models clearly beat the baseline, with PR-AUC about four times higher.
- **Random forest is the recommended model.** It is nearly as accurate as gradient boosting on the random split and holds up much better on later data. Logistic regression is a simpler, easy-to-explain alternative.
- Customers whose previous campaign succeeded subscribed at 65%, against 9% for customers never contacted before. Mobile contacts converted at 15% and landline contacts at 5%.
- **Recommended threshold: 0.15.** On the test set, the bank calls 17% of customers and reaches 64% of subscribers, with 42% of calls succeeding compared with 11% when calling at random.

**Limitations**

- The cost figures are assumptions. The threshold should be recalculated with the bank's real costs.
- Performance is weaker on later data, so the model should be retrained and monitored over time.
- The data only covers customers the bank chose to call.
- The patterns found are predictive, not causal.


## Author

Kwera Stella 
