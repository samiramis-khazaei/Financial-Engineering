# LendingClub Credit Risk Scorecard

## Project Overview

This project develops an interpretable **Probability of Default (PD) model and credit scorecard** using LendingClub loan data. The objective is to estimate borrower default risk using information available around loan origination, rank borrowers from higher to lower risk, and translate a logistic-regression model into an operational score.

The project follows a traditional credit-risk workflow:

**Raw loan data → leakage control → cleaning and feature engineering → temporal development/test split → coarse classing → WoE/IV → fine classing → feature selection → logistic regression → challenger model → out-of-time validation → gains/lift → calibration → scorecard**

The binary target is:

- `1 = Default / Charged Off`
- `0 = Fully Paid / Non-default`

---

## Business Objective

A lending model should answer two related questions:

1. **Risk ranking:** Which borrowers are more likely to default?
2. **Probability estimation:** How likely is each borrower to default?

The model is therefore evaluated on both:

- **Discrimination:** whether risky borrowers receive higher predicted PDs than safer borrowers.
- **Calibration:** whether predicted PDs agree with observed default frequencies.

The final scorecard is designed so that:

- higher PD = higher risk
- lower score = higher risk
- higher score = lower risk

---

## Dataset

The dataset contains LendingClub loans issued between 2007 and 2014.

After removing all-null, constant, identifier, high-cardinality, and post-origination leakage variables, the working dataset contains approximately **466,285 loans and 44 variables**.

### Target population

Loans with a resolved outcome are used for model development:

- Fully Paid → `0`
- Does not meet credit policy: Fully Paid → `0`
- Charged Off → `1`
- Default → `1`
- Does not meet credit policy: Charged Off → `1`

Loans still active or unresolved are separated from the development sample:

- Current
- Late (16–30 days)
- Late (31–120 days)
- In Grace Period

The resolved development population contains approximately:

- **80.9% non-defaults**
- **19.1% defaults**

---

## Leakage Control

The model is intended to estimate risk using information available at or near origination. Variables that reveal loan performance after origination are therefore excluded.

Examples include:

- outstanding principal
- total payments received
- principal and interest collected
- late fees
- recoveries and recovery fees
- last payment date and amount
- next payment date
- latest credit-pull date

Removing these variables prevents the model from using information that would not have been available when the lending decision was made.

---

## Data Cleaning and Missing Values

High-cardinality identifiers and text fields are removed because they do not provide stable, generalizable borrower risk information.

Examples include:

- loan ID
- member ID
- employer title
- URL
- free-text description
- borrower-provided title

Missing-value treatment is based on the meaning of each variable.

For "months since" variables, missingness can itself contain information. The workflow therefore creates missing indicators and replaces missing numeric values with a value beyond the observed range. This lets the model distinguish "no recorded event / unavailable record" from a recent event.

For example:

- `no_recorded_delinq`
- `no_record`
- `no_major_derog`

A remaining improvement is to label missing `emp_length` as **Missing/Unknown** rather than automatically treating it as unemployed unless the data dictionary supports that assumption.

---

## Feature Engineering

### Credit history length

`earliest_cr_line` is converted into a borrower credit-history duration:

$CreditHistoryMonths =IssueDate - EarliestCreditLine$

This transforms two dates into a more interpretable risk variable: the number of months the borrower has had recorded credit history.

### Term

Loan term is converted into a binary representation distinguishing 36-month and 60-month loans.

### Rare categories

Rare categorical levels are consolidated where appropriate to avoid unstable estimates from very small groups.

---

## Exploratory Data Analysis

EDA is used to understand:

- class imbalance
- missingness
- numerical distributions
- skewness
- outliers
- category frequencies
- relationships among candidate predictors

Extreme values are inspected rather than automatically deleted. Because the scorecard later bins continuous predictors, extreme observations can often be absorbed into interpretable tail bins.

The purpose of EDA in this project is not simply visualization; it informs cleaning, binning, missing-value treatment, and feature-selection decisions.

---

## Temporal Development and Out-of-Time Test

The model uses a chronological split:

- **Development sample:** loans issued before 2014
- **Out-of-time test sample:** loans issued in 2014

The class mix changes slightly across time:

- pre-2014 development sample: approximately **18.3% defaults**
- 2014 test sample: approximately **21.1% defaults**

This split is preferable to a purely random final holdout because it tests whether a model trained on earlier loan vintages generalizes to a later period.

The 2014 sample should not be used to learn bin boundaries, WoE values, feature selection, or model parameters.

---

## Coarse Classing

Continuous variables are initially divided into bins using a supervised decision-tree procedure.

The tree uses entropy to identify cut points that separate defaults from non-defaults while limiting the number of terminal bins and enforcing a minimum bin size.

The objective of coarse classing is to:

- capture nonlinear risk relationships
- reduce sensitivity to extreme observations
- create interpretable intervals
- prepare variables for WoE transformation
- provide a starting point for manual fine classing

The bin boundaries are learned on the development sample and then applied unchanged to the out-of-time test set.

---

## Weight of Evidence (WoE)

Each bin is transformed using Weight of Evidence:

$
WoE_i =
\ln
\left(
\frac{\text{Distribution of Defaults}_i}
{\text{Distribution of Non-defaults}_i}
\right)
$

Under this project's convention:

- **positive WoE** → relatively greater concentration of defaults
- **negative WoE** → relatively greater concentration of non-defaults

WoE allows categorical and binned numerical predictors to enter logistic regression on a common risk scale.

---

## Information Value (IV)

Information Value measures the univariate discriminatory strength of each variable:

$
IV =
\sum_i
(\text{DefaultDist}_i-\text{NonDefaultDist}_i)\times WoE_i
$

The strongest initial predictors include:

| Variable | IV |
|---|---:|
| Interest rate | ~0.397 |
| Sub-grade | ~0.392 |
| Grade | ~0.363 |
| Term | ~0.168 |
| DTI | ~0.065 |
| Revolving utilization | ~0.063 |
| Annual income | ~0.050 |

The project initially retains variables with:

$
IV \ge 0.02
$

IV is treated as a screening tool rather than the only feature-selection criterion. Business meaning, correlation, stability, and leakage risk are also considered.

---

## Fine Classing

The coarse bins are reviewed and manually consolidated.

Bins are candidates for merging when they have:

- very low frequency
- similar WoE
- similar observed default rates
- noisy or unstable patterns
- little business distinction

Fine classing improves stability and interpretability while reducing unnecessary fragmentation.

Examples include merging:

- LendingClub Grades F and G
- adjacent DTI intervals
- adjacent revolving-utilization intervals
- similar annual-income ranges
- noisy loan-amount ranges
- similar loan-purpose categories

Exactly the same final mappings are applied to the test data.

After fine classing, WoE and IV are recalculated because changing bins changes the distribution of defaults and non-defaults.

---

## Correlation and Redundancy

After IV screening, correlation is reviewed among WoE-transformed features.

Highly redundant variables are removed to avoid allowing the same underlying risk signal to enter the logistic model multiple times.

The project removes:

- `funded_amnt`
- `funded_amnt_inv`
- `sub_grade`
- `int_rate`

This leaves a more compact and interpretable scorecard while retaining `grade` as the principal LendingClub risk-grade variable.

The final scorecard feature set is approximately:

- grade
- term
- debt-to-income ratio
- revolving utilization
- annual income
- total current balance
- verification status
- loan amount
- loan purpose
- inquiries in the last six months
- total revolving credit limit

---

## Champion and Challenger Models

### Logistic Regression — Champion

Logistic regression is selected as the scorecard model because it provides:

- transparent coefficients
- direct PD estimates
- compatibility with WoE variables
- straightforward score scaling
- easy explanation and governance

The final specification uses L1 regularization:

```python
LogisticRegression(
    solver="liblinear",
    penalty="l1",
    C=0.1,
    max_iter=5000,
    random_state=42
)
```

### XGBoost — Challenger

XGBoost is included as a nonlinear challenger to test whether additional model flexibility materially improves discrimination.

The comparison uses cross-validation metrics including AUC, KS, Average Precision, Brier Score, Log Loss, Accuracy, Precision, Recall, and F1.

For a credit scorecard, a small gain in AUC from a black-box model may not justify sacrificing interpretability and deployability.

---

## Model Validation

The final logistic model is evaluated on both the development sample and the untouched 2014 out-of-time test sample.

### Accuracy

$
Accuracy=\frac{TP+TN}{TP+TN+FP+FN}
$

Measures the fraction of all classifications that are correct.

Because the dataset is imbalanced, accuracy is not used as the primary metric.

### Precision

$
Precision=\frac{TP}{TP+FP}
$

Among borrowers predicted to default, the fraction that actually default.

### Recall

$
Recall=\frac{TP}{TP+FN}
$

Among all actual defaults, the fraction identified by the model.

### F1 Score

$
F1 =
2\frac{Precision\times Recall}{Precision+Recall}
$

Balances precision and recall.

### ROC-AUC

ROC-AUC measures overall ranking ability across all thresholds.

It can be interpreted as the probability that a randomly selected defaulter receives a higher predicted risk than a randomly selected non-defaulter.

- `0.50` = random ranking
- `1.00` = perfect ranking

### Gini

$
Gini = 2\times AUC-1
$

Gini contains the same ranking information as AUC and is commonly reported in credit-risk modeling.

### Kolmogorov–Smirnov (KS)

$
KS =
\max_t
\left|
F_{Default}(t)-F_{NonDefault}(t)
\right|
$

KS is the maximum separation between the cumulative score distributions of defaults and non-defaults.

Higher KS indicates stronger separation.

### Average Precision

Average Precision summarizes the precision-recall curve.

It is especially informative when defaults are less frequent than non-defaults.

### Brier Score

$
Brier =
\frac{1}{N}\sum_i(p_i-y_i)^2
$

Measures squared probability error.

**Lower is better.**

### Log Loss

$
LogLoss =
-\frac{1}{N}
\sum_i
[y_i\ln(p_i)+(1-y_i)\ln(1-p_i)]
$

Penalizes confident but incorrect probability forecasts.

**Lower is better.**

**Results**

| Metric        | Train    | Test     |
|---------------|----------|----------|
| Accuracy      | 0.818150 | 0.791256 |
| Precision     | 0.524085 | 0.543619 |
| Recall        | 0.036989 | 0.051722 |
| F1            | 0.069101 | 0.094457 |
| AUC           | 0.699164 | 0.700243 |
| Gini          | 0.398327 | 0.400485 |
| Brier         | 0.137459 | 0.152101 |
| LogLoss       | 0.436751 | 0.471957 |
| KS            | 0.289238 | 0.292068 |

---

## ROC Curve

Train and test ROC curves are plotted together.

The plot is used to assess both:

- absolute discrimination
- generalization from development to the later loan vintage

A large gap between training and test AUC would indicate instability or overfitting. Similar curves suggest more stable ranking performance.

---

## Confusion Matrix

For:

- `0 = Non-default`
- `1 = Default`

the confusion matrix is:

| | Predicted Non-default | Predicted Default |
|---|---:|---:|
| **Actual Non-default** | True Negative | False Positive |
| **Actual Default** | False Negative | True Positive |

A **false negative** is a borrower who actually defaults but is classified as non-default.

The confusion matrix depends on the classification threshold, so the default 0.50 cutoff should not automatically be interpreted as the optimal lending decision threshold.

---

## Gains and Lift

Predicted PDs are ranked from highest to lowest and divided into ten approximately equal-sized deciles.

### Cumulative Gain

$
Gain_k =
\frac{\text{Cumulative defaults captured through decile }k}
{\text{Total defaults}}
$

This answers:

> If the lender focuses on the riskiest X% of borrowers, what percentage of all defaults are captured?

### Lift

$
Lift_k =
\frac{CumulativeGain_k}{CumulativePopulation_k}
$

Lift compares the model with random selection.

For example, if the riskiest 20% of borrowers contain 40% of all defaults:

$
Lift=\frac{0.40}{0.20}=2
$

The model is concentrating defaults at twice the rate of random selection.

---

## Probability Calibration

Discrimination and calibration answer different questions:

- **AUC / Gini / KS:** Is the ordering of borrower risk correct?
- **Calibration:** Are the predicted PD values numerically realistic?

The calibration plot groups borrowers by predicted probability and compares:

$
Mean\ Predicted\ PD
$

with:

$
Observed\ Default\ Rate
$

Perfect calibration lies on the 45-degree line:

$
Observed\ Default\ Rate = Predicted\ PD
$

The out-of-time calibration curve in this project lies very close to the reference line across the observed PD range, indicating that the logistic model's probabilities align closely with realized default rates in the test sample.

---

## Credit Scorecard

The fitted logistic model is:

$
logit(PD)=\beta_0+\sum_j \beta_j WoE_j
$

The score is defined as:

$
Score = Offset-Factor\times logit(PD)
$

The project uses:

- **Base Score:** 1000
- **Bad:Good base odds:** 0.10 = 1:10
- **PDO:** 200

Therefore a score of 1000 corresponds to Bad:Good odds of 1:10:

$
PD=\frac{0.1}{1+0.1}\approx9.09\%
$

The scaling factor is:

$
Factor=\frac{PDO}{\ln(2)}
$

and:

$
Offset=BaseScore+Factor\ln(BaseOdds)
$

Because score decreases as default odds increase:

- higher score = lower risk
- lower score = higher risk

A doubling of Bad:Good odds decreases the score by one PDO, or 200 points.

---

## Scorecard Validation

The scorecard is validated by proving that its score transformation reproduces the logistic-regression PD.

From:

$
Score=Offset-Factor\times logit(PD)
$

we obtain:
$
logit(PD)=\frac{Offset-Score}{Factor}
$

and:

$
PD=
\frac{1}
{1+\exp[-(Offset-Score)/Factor]}
$

The PD reconstructed from the score should match `LogisticRegression.predict_proba()` up to floating-point precision.

This validates that the scorecard is mathematically equivalent to the fitted logistic model rather than a separate approximation.

Because score is a monotonic decreasing transformation of PD, ranking metrics calculated using `-Score` should reproduce the model's AUC, Gini, KS, gains, and lift.

---

## Metric Framework

| Question | Metrics |
|---|---|
| Can the model rank risk? | ROC-AUC, Gini, KS, Average Precision |
| Are the PDs numerically reliable? | Calibration, Brier Score, Log Loss |
| How does a specific cutoff perform? | Confusion Matrix, Accuracy, Precision, Recall, F1 |
| Does the ranking concentrate defaults? | Gains, Lift |
| Does the model generalize over time? | Out-of-time test, train-test AUC/KS gaps |

No single metric is sufficient for model validation.

---

## Modeling Limitations

### Supervised preprocessing before cross-validation

Coarse binning, WoE estimation, IV screening, and manual fine classing are currently developed using the full development sample before model cross-validation.

Because these steps use the target, cross-validation results can be optimistic. A production-grade implementation would fit supervised preprocessing separately within each training fold or use nested / out-of-fold preprocessing.

The 2014 out-of-time test remains the most important validation set provided it is not used to learn these preprocessing decisions.

### Resolved-loan selection and maturity

Using only resolved loans can introduce maturity or right-censoring bias if later loan vintages have not had enough observation time to reach a final outcome.

A production model would ideally define default over a fixed performance horizon such as 12 or 24 months.

### Decision threshold

The default logistic-regression cutoff of 0.50 is not necessarily the economically optimal lending threshold.

A practical cutoff should reflect:

- expected loss
- false-negative cost
- false-positive / rejected-good cost
- pricing
- risk appetite
- capital requirements

---

## Conclusion

This project demonstrates an end-to-end, interpretable credit-risk modeling workflow using traditional scorecard methodology and modern validation techniques.

The main strength of the project is the combination of:

- origination-time leakage control
- temporal out-of-time testing
- supervised binning and fine classing
- WoE / IV feature engineering
- interpretable logistic regression
- nonlinear challenger modeling
- AUC / Gini / KS validation
- gains and lift analysis
- probability calibration
- operational credit-score scaling

The result is not only a classifier, but a transparent framework for ranking borrower risk, estimating PD, and converting model output into a practical credit score.

---

## Disclaimer

This project is for educational and portfolio purposes only. It is not a production underwriting system and should not be used to make real lending decisions.
