# Ex-Ante Risk Assessment and Attribution of a Technology Stock Portfolio

## Overview

This project performs an **ex-ante risk assessment of a concentrated U.S. technology stock portfolio** using scenario-based portfolio analysis.

The objective is to evaluate the portfolio from several complementary perspectives:

- Portfolio value and allocation
- Ex-ante return distribution
- Downside and tail risk
- Performance attribution
- Risk attribution

The analysis demonstrates how portfolio-level risk can be decomposed into its underlying stock and residual risk factors, providing insight beyond traditional portfolio weights.

---

## Portfolio

The buy-and-hold portfolio consists of five major U.S. technology companies:

| Stock | Ticker | Shares |
|---|---|---:|
| Microsoft | MSFT | 1,000 |
| Apple | AAPL | 2,000 |
| NVIDIA | NVDA | 10,000 |
| Amazon | AMZN | 2,000 |
| Alphabet | GOOGL | 2,000 |

The current portfolio value is approximately **$1,122,981**.

Although NVIDIA has the largest number of shares, portfolio exposure is determined by the **market value of each position**, rather than the number of shares held. Apple represents the largest portfolio weight at approximately **25.3%**, followed by Microsoft at approximately **21.5%**.

---

## Scenario Framework

The analysis is based on:

- **1,000 joint payoff scenarios**
- Equal scenario probabilities
- Five technology stocks
- A **one-business-week investment horizon**
- Scenario-based ex-ante portfolio returns

---

## Methodology

### 1. Value Aggregation

The current portfolio value is calculated by aggregating the market value of each position:

$V = \sum_{n=1}^{N} h_n P_n$

where:

- $h_n$ = number of shares of asset $n$
- $P_n$ = current price of asset $n$
- $V$ = total portfolio value

Portfolio weights are calculated as:

$w_n = \frac{h_nP_n}{V}$

The resulting portfolio value is approximately:

$V^{\mathbf{h}} = \$1{,}122{,}981$

---

### 2. Performance Aggregation

Joint stock return scenarios are transformed into portfolio return scenarios using the portfolio weights:

$R_p = \sum_{n=1}^{N} w_nR_n$

where $R_p$ represents the portfolio return and $R_n$ represents the return of asset $n$.

### Key Results

- **Expected one-week return:** approximately **0.37%**
- **Standard deviation:** approximately **4.80%**
- **Worst scenario:** approximately **-17.47%**
- **Best scenario:** approximately **+17.80%**

Despite the positive expected return, the relatively high volatility indicates substantial uncertainty over the one-week investment horizon.

---

### 3. Ex-Ante Risk Evaluation

Portfolio loss is defined as the negative of portfolio return:

$L = -R_p$

Several complementary risk measures are used to characterize the portfolio loss distribution.

#### Expected Loss

The expected loss is approximately:

$\mathbb{E}[L] \approx -0.37\%$

This corresponds to an expected portfolio return of approximately **+0.37%**.

#### Standard Deviation

$\sigma_L \approx 4.80\%$

The standard deviation indicates substantial dispersion around the expected outcome.

#### 99% Value-at-Risk

The portfolio's 99% Value-at-Risk is approximately:

$VaR_{99\%} \approx 10.55\%$

In dollar terms:

$0.1055 \times 1{,}122{,}981 \approx \$118{,}475$

This means there is approximately a **1% probability that the portfolio loss will exceed this level** over the one-week horizon.

#### 99% Conditional Value-at-Risk

The portfolio's 99% Conditional Value-at-Risk is approximately:

$CVaR_{99\%} \approx 12.63\%$

In dollar terms:

$0.1263 \times 1{,}122{,}981 \approx \$141{,}822$

Conditional on being in the worst 1% of scenarios, the average loss is therefore approximately **$141,822**.

The difference between VaR and CVaR highlights the presence of meaningful **tail risk**.

---

## 4. Ex-Ante Performance Attribution

The objective of performance attribution is to identify which individual stock returns best explain variation in portfolio returns.

The individual stock returns are treated as candidate risk factors:

- MSFT
- AAPL
- NVDA
- AMZN
- GOOGL

A **LASSO regression** is used to obtain a sparse factor representation of portfolio performance.

The general factor model can be written as:

$R_p = \alpha + \boldsymbol{\beta}^{\top}\mathbf{Z} + U$

where:

- $R_p$ = portfolio return
- $\alpha$ = intercept
- $\boldsymbol{\beta}$ = factor exposures
- $\mathbf{Z}$ = selected stock-return factors
- $U$ = residual component

The LASSO procedure selects:

- **NVDA**
- **AMZN**
- **GOOGL**

as the most relevant explanatory factors, while the coefficients for MSFT and AAPL are shrunk to zero.

These coefficients should not be interpreted as portfolio weights. Portfolio weights describe capital allocation, while attribution coefficients describe how strongly each factor explains changes in portfolio performance.

### Key Insight

**NVIDIA emerges as the dominant driver of portfolio return variability.**

Although Apple and Microsoft have relatively large portfolio weights, they provide less incremental explanatory information after the selected risk factors are included.

---

## 5. Ex-Ante Risk Attribution

Risk attribution is performed using an **Euler decomposition**, allowing total portfolio risk to be decomposed into contributions from the selected factors and the residual component.

For a positively homogeneous risk measure $\rho$, Euler allocation can be expressed as:

$\rho(\mathbf{x})
=
\sum_{i=1}^{N}
x_i
\frac{\partial \rho(\mathbf{x})}{\partial x_i}$

Each term

$RC_i =x_i\frac{\partial \rho(\mathbf{x})}{\partial x_i}$

represents the risk contribution of factor $i$.

The analysis shows that portfolio risk is not distributed proportionally to investment weights.

In particular, **NVIDIA and the residual factor dominate portfolio volatility and tail risk**.

The residual component contributes approximately:

- **39% of portfolio variance**
- **65% of VaR**
- **45% of CVaR**

This disproportionately large contribution to tail risk suggests that the residual component plays an important role in extreme portfolio losses.

Possible explanations include:

- Heavy-tailed residual behavior
- Nonlinear dependencies
- Omitted systematic risk factors
- Risk that cannot be fully captured by the selected three-factor representation

---

## Key Findings

1. **Portfolio weights and risk contributions are not the same.**  
   A stock can have a moderate portfolio weight while contributing disproportionately to total portfolio risk.

2. **Positive expected return does not imply low risk.**  
   The portfolio has a positive expected one-week return but is exposed to substantial volatility and tail losses.

3. **VaR alone is insufficient for tail-risk analysis.**  
   CVaR provides information about the severity of losses after the VaR threshold has been exceeded.

4. **A small number of factors explain much of portfolio variation.**  
   LASSO identifies NVDA, AMZN, and GOOGL as the most informative explanatory factors.

5. **Residual risk matters.**  
   The residual component contributes disproportionately to extreme portfolio losses.

6. **Sector concentration creates diversification risk.**  
   Because all five holdings are large U.S. technology companies, correlated adverse market movements can significantly affect the entire portfolio.

---

## Workflow

```text
Joint Stock Payoff Scenarios
          |
          v
   Portfolio Valuation
          |
          v
 Individual Stock Returns
          |
          v
 Portfolio Return Distribution
          |
          v
   Portfolio Loss Distribution
          |
          +-- Expected Loss
          +-- Standard Deviation
          +-- VaR
          +-- CVaR
          |
          v
 Performance Attribution
     (LASSO Regression)
          |
          v
 Selected Risk Factors
 NVDA + AMZN + GOOGL
          |
          v
    Risk Attribution
   (Euler Decomposition)
```

---

## Techniques Used

- Scenario-based portfolio analysis
- Portfolio value aggregation
- Portfolio return aggregation
- Loss distribution modeling
- Value-at-Risk (VaR)
- Conditional Value-at-Risk (CVaR / Expected Shortfall)
- Correlation analysis
- LASSO regression
- Sparse factor selection
- Ex-ante performance attribution
- Euler risk decomposition
- Tail-risk analysis

---

## Tools

The analysis is implemented in **Python** using a Jupyter Notebook / Google Colab environment.

Core functionality includes:

- Numerical computation
- Data manipulation
- Statistical analysis
- Machine-learning-based factor selection
- Data visualization
- Portfolio risk analytics

---

## Conclusion

This project demonstrates a scenario-based framework for evaluating the **ex-ante risk profile of an equity portfolio**.

The results show that apparent diversification across five technology stocks does not necessarily translate into diversified risk. In particular, NVIDIA and the unexplained residual component contribute disproportionately to portfolio variability and extreme losses.

The analysis illustrates why portfolio risk management should move beyond holdings and weights toward identifying the **underlying drivers of performance, volatility, and tail risk**.

---

## Disclaimer

This project is intended for educational and portfolio-demonstration purposes only. The analysis and results should not be interpreted as investment advice.
