# Credit Risk Modeling with Logistic Regression

A machine-learning project for estimating the **probability of credit default** using logistic regression on the German Credit dataset.

The project covers exploratory data analysis, preprocessing, feature engineering, comparison of alternative logistic-regression pipelines, regularization, ROC-AUC evaluation, threshold-based classification, out-of-sample testing, and application of the final model to a new loan applicant.

> **Target definition:** `0 = non-default / good credit`, `1 = default / bad credit`

---

## Project Overview

Credit risk modeling helps lenders estimate the likelihood that a borrower will fail to repay a loan. In this project, logistic regression is used to model the probability that a personal-loan applicant is a **bad credit risk**.

The workflow is designed around two related outputs:

1. **Probability of default (PD)** — the model's predicted probability that an applicant defaults.
2. **Point classification** — a default/non-default decision obtained by applying a probability threshold.

The primary metric used to compare candidate models is **ROC-AUC**, because it evaluates ranking/discrimination across all possible classification thresholds.

---

## Dataset

The project uses the **German Credit dataset (OpenML ID 31)**.

| Item | Description |
|---|---|
| Observations | 1,000 loan applicants |
| Input features | 20 |
| Target | `class` |
| Original target values | `good`, `bad` |
| Modeling target | `good → 0`, `bad → 1` |
| Approximate class distribution | 70% non-default, 30% default |
| Source | OpenML |

The dataset contains demographic, financial, employment, housing, repayment-history, and credit-related information.

---

## Workflow

```text
Data Collection
      ↓
Exploratory Data Analysis
      ↓
Data Preparation
      ↓
Train / Validation / Test Split
      ↓
Feature Engineering
      ↓
Logistic Regression Model Comparison
      ↓
ROC-AUC Model Selection
      ↓
Regularization
      ↓
Probability-of-Default Estimation
      ↓
Threshold-Based Classification
      ↓
Out-of-Sample Evaluation
      ↓
New Applicant Credit Decision
```

---

## Exploratory Data Analysis

The exploratory analysis identified several characteristics relevant to model development.

### Class imbalance

Approximately:

- **70%** of applicants are non-default borrowers.
- **30%** are default borrowers.

Because of this imbalance, ROC-AUC was emphasized for model comparison rather than relying only on classification accuracy.

### Numerical variables

`duration`, `credit_amount`, and `age` show right-skewed distributions.

A moderate positive correlation of approximately **0.62** was observed between:

```text
credit_amount ↔ duration
```

indicating that larger loans tend to have longer repayment periods.

### Selected categorical observations

The EDA also examined relationships between default status and:

- employment duration
- housing status
- personal status
- credit history
- loan purpose
- property ownership
- foreign-worker status
- other debtors / guarantors

These exploratory relationships were used to guide preprocessing and model interpretation rather than being treated as causal effects.

---

## Data Preparation

The target and selected binary variables were converted to numeric values.

```python
good → 0
bad  → 1
```

The following binary transformations were also applied:

```python
foreign_worker:
yes → 1
no  → 0

own_telephone:
yes  → 1
none → 0
```

Loan-purpose categories were consolidated into broader groups to reduce category fragmentation and potential multicollinearity:

```text
radio/tv
furniture/equipment
domestic appliance
        ↓
furniture/equipment/appliance


new car
used car
        ↓
vehicle


business
education
retraining
        ↓
income_investment


repairs
other
        ↓
repairs/other
```

---

## Train / Validation / Test Design

The dataset was split using **stratified sampling** so that the class distribution was preserved.

```python
# First split
85% development data
15% final test data

# Second split
development data → training + validation
```

The validation set was used for model comparison and threshold analysis, while the final test set was kept separate for out-of-sample evaluation.

---

## Feature Engineering

Different preprocessing techniques were applied according to feature type.

| Feature | Characteristic | Transformation |
|---|---|---|
| `duration` | Right-skewed | QuantileTransformer |
| `credit_amount` | Right-skewed | QuantileTransformer |
| `age` | Right-skewed | QuantileTransformer |
| `installment_commitment` | Numerical / ordinal | StandardScaler |
| `residence_since` | Numerical / ordinal | StandardScaler |
| `existing_credits` | Numerical / ordinal | StandardScaler |
| `num_dependents` | Binary / discrete | No transformation |

Categorical variables were evaluated using both **One-Hot Encoding** and **Weight of Evidence (WoE)** encoding.

---

## Candidate Models

Three logistic-regression pipelines were compared.

### Model 1 — Numerical Features

Uses numerical variables only.

```text
Right-skewed numerical features
        ↓
QuantileTransformer
        ↓
Other numerical features
        ↓
StandardScaler
        ↓
Logistic Regression
```

### Model 2 — Numerical + One-Hot Encoding

Uses both numerical and categorical information.

```text
Numerical preprocessing
        +
One-Hot Encoding
        ↓
Feature selection
        ↓
Logistic Regression
```

### Model 3 — Numerical + Weight of Evidence

Uses numerical variables together with **WoE-encoded categorical variables**.

```text
Numerical preprocessing
        +
WoE Encoding
        ↓
Logistic Regression
```

WoE is particularly useful in credit-risk modeling because categorical levels are represented according to their relationship with the log-odds of the target.

---

## Model Selection with ROC-AUC

Candidate models were compared using the **Area Under the Receiver Operating Characteristic Curve (ROC-AUC)**.

The ROC curve evaluates:


$TPR = \frac{TP}{TP+FN}$


against:

$FPR = \frac{FP}{FP+TN}$


across all possible thresholds.

AUC summarizes the model's discrimination ability into a single value:

- `0.50` → approximately random ranking
- closer to `1.00` → stronger discrimination

### Validation AUC

| Model | Validation AUC |
|---|---:|
| Model 1 — Numerical only | 0.715 |
| Model 2 — One-Hot Encoding | 0.724 |
| **Model 3 — WoE Encoding** | **0.802** |

Model 3 was selected because it achieved the strongest validation discrimination.

---

## Final Logistic Regression Model

After selecting the WoE pipeline, regularization was introduced.

The final logistic-regression specification uses:

```text
Penalty      : L1
Solver       : liblinear
Class weight : balanced
C            : 13.89495
```

The regularized model achieved approximately:

```text
Validation AUC = 0.803
```

---

## Classification Threshold

The model produces a probability:


$P(\text{default}\mid X)$

A classification threshold of:


$\boxed{0.55}$


was then used for the point classifier:

```python
prediction = 1 if PD >= 0.55 else 0
```

where:

```text
1 = predicted default
0 = predicted non-default
```

### Important distinction

**ROC-AUC was used to evaluate and select the model.**

AUC itself is **threshold-independent**, so a single AUC value does not mathematically determine the classification threshold. The `0.55` operating threshold was considered separately on the validation data after the probabilistic model had been selected.

This distinction is important in credit-risk applications:

```text
Model selection     → ROC-AUC
Business decision   → probability threshold
```

---

## Out-of-Sample Performance

The selected logistic-regression model achieved:

| Dataset | ROC-AUC |
|---|---:|
| Training | **0.81** |
| Test | **0.76** |

The reduction from training to test AUC suggests some overfitting, but the model retains useful discriminatory ability on unseen observations.

The result also indicates that the relationship between borrower characteristics and default may not be fully linear. More flexible models could potentially capture additional nonlinear relationships and interactions.

---

## Credit-Risk Interpretation

For this project:

```text
Positive class = Default
Negative class = Non-default
```

Therefore:

| Outcome | Credit Interpretation |
|---|---|
| True Positive | Defaulting borrower correctly identified |
| True Negative | Creditworthy borrower correctly identified |
| False Positive | Creditworthy borrower incorrectly rejected |
| False Negative | Defaulting borrower incorrectly approved |

In real lending applications, a **false negative** can be substantially more costly than a false positive because it can lead directly to credit losses.

For that reason, a production threshold should ultimately incorporate:

- expected loss
- probability of default
- exposure at default
- loss given default
- opportunity cost
- risk appetite

rather than relying only on statistical classification performance.

---

## Example: New Applicant

The final model was applied to a new applicant.

Estimated probability of default:


$P(\text{default}) = 0.412$


Using the project threshold:

$0.412 < 0.55$


the applicant is classified as:

```text
Predicted class: Non-default
Credit decision: Approve
```

The model therefore recommends extending credit under the project's decision rule.

---

## Key Findings

- Logistic regression provides an interpretable framework for estimating borrower default probability.
- The dataset is moderately imbalanced, with approximately 30% default observations.
- `duration`, `credit_amount`, and `age` are right-skewed and benefit from distributional transformation.
- Including categorical borrower information materially improves discrimination relative to a numerical-only model.
- **WoE encoding produced the strongest candidate model**, with validation AUC of approximately **0.802**.
- The regularized final model achieved approximately **0.81 training AUC** and **0.76 test AUC**.
- The train-test gap indicates some generalization loss.
- A probability threshold converts PD estimates into operational lending decisions, but the economically optimal threshold should ideally reflect asymmetric credit-loss costs.

---

## Limitations

This project uses a simplified academic credit dataset and should not be interpreted as a production banking model.

Important limitations include:

- only 1,000 observations
- simplified borrower information
- static modeling assumptions
- no explicit PD calibration analysis
- no LGD or EAD modeling
- no expected-loss optimization
- no fairness or regulatory assessment
- limited investigation of nonlinear effects
- threshold not optimized using an explicit monetary cost function

---

## Potential Improvements

Future extensions could include:

- comparing Logistic Regression with Random Forest, Gradient Boosting, and XGBoost
- probability calibration
- cross-validated hyperparameter tuning
- cost-sensitive threshold optimization
- Precision-Recall analysis
- KS statistic
- Brier score
- log loss
- SHAP or other model-explanation techniques
- PD, LGD, and EAD integration
- Expected Credit Loss modeling


---

## Technology Stack

| Component | Tools |
|---|---|
| Language | Python |
| Data manipulation | pandas, NumPy |
| Machine learning | scikit-learn |
| Statistical transformation | QuantileTransformer, StandardScaler |
| Categorical encoding | OneHotEncoder, custom WoE encoder |
| Model | LogisticRegression |
| Evaluation | ROC curve, AUC, confusion matrix, precision, recall |
| Visualization | Matplotlib, Seaborn |
| Data source | OpenML |

---



## Disclaimer

This project was developed for educational and portfolio purposes. It is a simplified credit-risk modeling exercise and is **not intended for real lending decisions or investment advice**.
