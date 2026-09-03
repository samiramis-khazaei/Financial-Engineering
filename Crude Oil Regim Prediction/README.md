# Crude Oil Market Regime Prediction

**Hidden Markov Models (HMMs) + Bayesian Networks (BNs) + PCA + Rolling-Window Validation**

This project develops a probabilistic framework for **classifying the next-month crude-oil market regime** as **Low, Middle, or High**.

Rather than forecasting crude-oil prices directly, the workflow first identifies latent market regimes using **Hidden Markov Models (HMMs)** and then predicts the next-month regime using **Bayesian Networks (BNs)**. A baseline categorical-HMM pipeline is compared with a **PCA-based Gaussian-HMM pipeline** under chronological rolling-window validation.

> **Portfolio project:** academic group project presented here as a technical portfolio case study.  
> **Note:** This repository is for educational and research purposes only and is not investment advice.

---

## Project Highlights

| Item | Result |
|---|---|
| **Objective** | Predict the next-month crude-oil regime: Low, Middle, or High |
| **Data** | Monthly market, macroeconomic, supply-demand, industrial, geopolitical, weather, and financial variables |
| **Period** | December 1997 – March 2024 |
| **Regime detection** | Categorical HMM and Gaussian HMM |
| **Prediction model** | Bayesian Network with MAP inference |
| **Structure learning** | Hill Climbing + BIC, Hill Climbing + BDeu, and PC |
| **Validation design** | Chronological rolling windows |
| **Best validation pipeline** | PCA-based pipeline; HC-BIC selected most often |
| **Validation accuracy** | 0.755 baseline → **0.856 PCA pipeline** |
| **Macro F1** | 0.710 baseline → **0.724 PCA pipeline** |
| **Main structural finding** | Current crude-oil regime is the dominant direct predictor of the next-month regime |
| **Final unseen period** | 12 scored months; all realized and predicted regimes were Middle |
| **Trading experiment** | Zero cumulative return because every prediction was Middle and the rule stayed flat |

---

## Problem Definition

Crude-oil markets are influenced by interacting **economic, supply-demand, financial, geopolitical, and environmental factors**.

This project asks whether:

1. HMMs can identify persistent and interpretable market regimes.
2. Bayesian Networks can predict the next-month regime out of sample.
3. A PCA-based pipeline can improve predictive performance while reducing network complexity.
4. Important direct dependencies remain stable through time.
5. Predicted regimes can be translated into a simple trading rule.

The target is the **HMM-defined crude-oil regime at month \(t+1\)**. The current regime at month \(t\) remains available as a predictor.

---

## Data

The modeling dataset contains monthly observations from **December 1997 through March 2024**.

Sources used in the original project include:

- U.S. Energy Information Administration (EIA)
- FRED Economic Data
- Yahoo Finance
- Geopolitical Risk dataset

Representative variables include:

| Category | Representative Variables |
|---|---|
| **Supply** | OPEC production, non-OPEC production |
| **Demand** | OECD and non-OECD petroleum consumption |
| **Industrial / Macro** | Industrial production indices for the U.S., OECD, China, India, and Russia |
| **U.S. Oil Industry** | Crude-oil industrial production, petroleum/coal inventories, capacity utilization |
| **Financial** | VIX, U.S. Dollar Index, NASDAQ Composite, U.S. Treasury yield |
| **Other** | Global temperature anomaly, Geopolitical Risk Index |
| **Target Input** | Crude-oil futures (`CL=F`), transformed into HMM regimes |

---

## Data Preparation

Key preprocessing principles:

- Preserve **time-series ordering**; random cross-validation is not used.
- Assess stationarity using **ADF** and **KPSS** diagnostics.
- Apply transformations where required.
- Treat missing values before model construction.
- Retain economically meaningful extreme observations rather than automatically treating them as errors.
- Fit scaling, quantile transformation, and PCA **only on the training window**, then apply them to validation data to reduce look-ahead bias.

---

## Methodology

### Pipeline 1 — Baseline Categorical-HMM

1. Transform and discretize continuous variables using training-derived thresholds.
2. Fit a **three-state Categorical HMM** separately to each modeled variable.
3. Map hidden states to ordered labels: **Low, Middle, High**.
4. Use the resulting regime variables as nodes in a discrete Bayesian Network.

### Pipeline 2 — PCA + Gaussian-HMM

1. Fit a **QuantileTransformer** on the training window.
2. Apply PCA to grouped industrial, developing-country, and financial-market variables.
3. Retain two principal components per group.
4. Fit a **three-state Gaussian HMM** to processed variables/components.
5. Map hidden states to **Low, Middle, High**.
6. Pass the regime variables into the Bayesian Network.

   
<img width="998" height="665" alt="image" src="https://github.com/user-attachments/assets/755173d2-4182-4fca-a0ee-8009687d7bea" />


### Bayesian Network Learning

Three structure-learning approaches are compared inside each rolling window:

- **Hill Climbing + BIC**
- **Hill Climbing + BDeu**
- **PC algorithm**

The candidate with the highest **validation Macro F1** is retained.

Prediction uses **MAP inference** for the next-month regime.

> Directed edges are interpreted as **conditional-dependency structure**, not as established causal effects.

---

## Validation Design

The project uses **chronological, non-anchored rolling windows**:

- **15 years** for training
- **2 years** for validation
- A forward **12-month** segment is also allocated in the window construction
- A separate final unseen period from **March 2023 to March 2024** is used for final regime prediction and the trading experiment

This design preserves temporal ordering and avoids random shuffling.
<img width="998" height="708" alt="image" src="https://github.com/user-attachments/assets/e5eab940-6388-4626-a8a4-ef95e95f14af" />


---

## Validation Results

| Metric | Baseline | PCA Pipeline | Difference |
|---|---:|---:|---:|
| **Accuracy** | 0.755 | **0.856** | +0.101 |
| **Macro F1** | 0.710 | **0.724** | +0.014 |
| **Weighted F1** | 0.753 | **0.844** | +0.091 |
| **Micro F1** | 0.755 | **0.856** | +0.101 |

### Interpretation

The PCA-based pipeline shows substantially higher average:

- Accuracy
- Micro F1
- Weighted F1

However, the improvement in **Macro F1 is small**, indicating that most of the gain comes from overall or majority-weighted classification rather than balanced improvement across all three regimes.

The PCA pipeline also shows a larger train-validation gap, which is consistent with **higher variance / overfitting risk**.

---

## Model-Selection Frequency

| Pipeline | Saved Windows | HC-BIC | HC-BDeu | PC |
|---|---:|---:|---:|---:|
| Baseline | 101 | **89 (88.1%)** | 11 (10.9%) | 1 (1.0%) |
| PCA Pipeline | 97 | **86 (88.7%)** | 6 (6.2%) | 5 (5.2%) |

**HC-BIC** is the dominant selected structure-learning configuration in both pipelines.

> The saved baseline and PCA outputs contain different numbers of windows, so the aggregate comparisons are not strictly paired over identical windows.

---

## Network Complexity and Stability

| Measure | Baseline | PCA Pipeline |
|---|---:|---:|
| Mean number of edges | 19.47 | **10.58** |
| Edge-count standard deviation | 5.35 | **3.76** |
| Minimum edges | 9 | **3** |
| Maximum edges | 37 | **25** |
| Mean consecutive-network Jaccard similarity | 0.198 | 0.155 |

The PCA-based pipeline produces **substantially smaller Bayesian Networks**, but its lower Jaccard similarity shows that exact edge configurations are **not more stable through time**.

This highlights an important distinction:

> **Parsimony and structural stability are not the same property.**

---

## Most Persistent Predictive Relationship

The strongest structural result is **regime persistence**.

- In the baseline pipeline, the current crude-oil regime (`cl_f`) is a direct parent of the next-month target in **all 101 rolling windows**.
- In the PCA pipeline, it appears in **92 of 97 windows (94.8%)**.

Other industrial or market factors appear as direct target parents only occasionally.

This suggests that a simple **persistence benchmark** — predicting next month to remain in the current regime — should be included in future versions of the project.

---

## Final Unseen Test

The PCA + HC-BIC framework is fitted using the 15 years preceding March 2023 and evaluated over the subsequent unseen period.

| Final-Test Statistic | Result |
|---|---:|
| Accuracy | 1.00 |
| Macro F1 | 1.00 |
| Weighted F1 | 1.00 |
| Micro F1 | 1.00 |
| Predicted class in all 12 months | Middle |
| Actual class in all 12 months | Middle |

The numerical scores are perfect, but the final test contains **no Low or High target regimes**.

Therefore, the result should be interpreted as:

> Correct classification of a persistent **Middle-regime episode**, not evidence of general 100% multiclass forecasting accuracy.

---

## Trading Experiment

Predicted regimes are mapped to positions as follows:

- **High** → `+1` (long)
- **Middle** → `0` (flat)
- **Low** → `-1` (short)

Because every final-period prediction is **Middle**, the strategy remains flat during the entire test period and generates:

**Cumulative return = 0**

Actual monthly crude-oil returns within the Middle regime still range roughly from **-10.4% to +9.4%**, showing that the three-state regime definition is too coarse to produce differentiated trading signals in this test episode.

---

## Temporal Evolution of the Bayesian Network

The PCA + HC-BIC specification is also estimated over three overlapping 15-year periods.

| Period | Edges | Direct Parent of Current Oil Regime |
|---|---:|---|
| Pre-COVID (2004–2018) | 13 | Temperature anomaly → current oil regime |
| COVID-inclusive (2006–2020) | 13 | Developing-country industrial factor → current oil regime |
| Later period (2010–2024) | 9 | Supply → current oil regime |

The surrounding dependency structure changes substantially across periods.

Pairwise Jaccard similarities are very low:

- 0.040
- 0.100
- 0.048

However, one forecasting relationship remains stable:

> The **current crude-oil regime remains the only direct parent of the next-month regime** in all three period-specific networks.

<img width="930" height="639" alt="image" src="https://github.com/user-attachments/assets/5019ce45-67a1-4082-add7-e2f90e1fe6a9" />

<img width="930" height="639" alt="image" src="https://github.com/user-attachments/assets/0f1b02c2-1a15-4013-92c7-d8a7de878abf" />

<img width="930" height="639" alt="image" src="https://github.com/user-attachments/assets/ecb9ca71-54ce-4bc6-ae96-ba8b06929d30" />




---

## Key Takeaways

- HMMs provide a structured way to represent crude-oil market conditions as latent regimes.
- Bayesian Networks capture changing conditional dependencies among macro-financial and market variables.
- The PCA-based pipeline improves average accuracy and weighted F1 while producing a more compact network.
- The improvement cannot be attributed to PCA alone because the two pipelines also differ in preprocessing and HMM specification.
- The most robust predictive relationship is **current-regime → next-month-regime persistence**.
- The final unseen test is not sufficiently diverse to establish general multiclass performance.
- The trading rule does not generate returns in the final test because all predicted regimes are Middle.

---
## Future Improvements

Planned extensions include:

- Add persistence / Markov-transition benchmarks.
- Compare against multinomial logistic regression and gradient boosting.
- Use identical rolling dates for all candidate pipelines.
- Report paired differences with confidence intervals or bootstrap uncertainty.
- Run a controlled PCA ablation.
- Report per-class precision, recall, F1, and confusion matrices.
- Evaluate posterior regime probabilities using log loss and Brier score.
- Redesign the trading layer using probabilities or return forecasts.
- Include transaction costs and risk-adjusted benchmarks.
- Reserve a longer final test containing multiple regime transitions.

---

## Technology Stack

| Component | Tools |
|---|---|
| Language | Python |
| Core Data Tools | pandas, NumPy |
| Time Series / Diagnostics | statsmodels, arch |
| Hidden Markov Models | hmmlearn |
| Bayesian Networks | pgmpy |
| Machine Learning / PCA | scikit-learn |
| Graph Analysis | NetworkX |
| Visualization | Matplotlib |
| Market Data | Yahoo Finance / yfinance |

---

## References

Selected references used in the project include:

- Alvi, D. A. (2018). *Application of Probabilistic Graphical Models in Forecasting Crude Oil Price*. University College London.
- Caldara, D., & Iacoviello, M. (2022). *Measuring Geopolitical Risk*. American Economic Review, 112(4), 1194–1225.
- Hamilton, J. D. (1989). *A New Approach to the Economic Analysis of Nonstationary Time Series and the Business Cycle*. Econometrica, 57(2), 357–384.
- Jolliffe, I. T., & Cadima, J. (2016). *Principal Component Analysis: A Review and Recent Developments*. Philosophical Transactions of the Royal Society A, 374.
- Koller, D., & Friedman, N. (2009). *Probabilistic Graphical Models: Principles and Techniques*. MIT Press.
- Rabiner, L. R. (1989). *A Tutorial on Hidden Markov Models and Selected Applications in Speech Recognition*. Proceedings of the IEEE, 77(2), 257–286.
- Federal Reserve Bank of St. Louis — FRED Economic Data.
- U.S. Energy Information Administration — Short-Term Energy Outlook / Open Data.
- Yahoo Finance — Historical Market Data.
- `pgmpy` documentation.
- `hmmlearn` documentation.

---

## Disclaimer

This repository is a portfolio presentation of an academic project and saved notebook outputs. It is intended for educational and research purposes only and does not constitute investment advice.
