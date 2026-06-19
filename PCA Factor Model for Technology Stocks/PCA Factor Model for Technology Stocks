# PCA Factor Model for Technology Stocks

This project implements a **Principal Component Analysis (PCA)** factor model to identify the common sources of risk driving technology stock returns. The model combines **Flexible Probabilities (FP)** with **VIX-based volatility regime conditioning** to estimate expected returns and covariance matrices before extracting latent risk factors through PCA.

The notebook evaluates whether the covariance structure estimated from historical data remains stable out-of-sample and demonstrates how PCA can be used to construct statistical factor portfolios.

---

## Features

- Download historical prices for 14 major technology stocks using Yahoo Finance
- Estimate Flexible Probabilities using exponential time decay and VIX regime conditioning
- Compute probability-weighted expected returns and covariance matrices
- Perform Principal Component Analysis (PCA)
- Analyze explained variance and PCA loadings
- Evaluate factor stability using out-of-sample return reconstruction
- Construct and interpret PCA eigen-portfolios
- Compare PCA factor portfolios with a market-cap weighted portfolio

---

## Dataset

### Technology Stocks

- Apple (AAPL)
- Microsoft (MSFT)
- Alphabet (GOOGL)
- Amazon (AMZN)
- Meta (META)
- Cisco (CSCO)
- NVIDIA (NVDA)
- Oracle (ORCL)
- Palantir (PLTR)
- SAP
- IBM
- Salesforce (CRM)
- Shopify (SHOP)
- Uber (UBER)

### Market Indicator

- CBOE Volatility Index (VIX)

### Training Period

January 2024 – December 2024

### Test Period

January 2025 – September 2025

---

## Methodology

The project consists of the following steps:

1. Collect historical stock prices.
2. Estimate Flexible Probabilities using exponential time decay and VIX state conditioning.
3. Compute probability-weighted expected returns and covariance matrices.
4. Apply PCA to the correlation matrix.
5. Evaluate the extracted factors on an out-of-sample test dataset.
6. Construct PCA factor portfolios and compare them with a market-cap weighted portfolio.

---

## Results

- The first principal component explains approximately **39.5%** of the total variance.
- Approximately **90%** of the variance is explained by the first **10** principal components.
- The first **3** principal components are retained to construct a parsimonious factor model.
- The dominant factor represents a broad technology market factor, while the second factor captures a growth-versus-value dimension within the sector.
- Out-of-sample results indicate that the common factor structure remains reasonably stable across the test period.

---

## Technologies

- Python
- NumPy
- Pandas
- Matplotlib
- yfinance
- ARPM Toolbox

---

## Repository Structure

```
├── PCA_Factor_Model.ipynb
└── README.md
```

---

## References

- Meucci, A. *Risk and Asset Allocation*
- ARPM (Advanced Risk and Portfolio Management)
- Jolliffe, I. T. *Principal Component Analysis*
- Yahoo Finance
