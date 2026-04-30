[README.md](https://github.com/user-attachments/files/27248997/README.md)
# 📈 Options Pricing & Volatility Surface Model

This project is a complete Python framework for **options data analysis, implied volatility surface construction, and derivative pricing** using both the **Black-Scholes model** and a **Calibrated Local Volatility (CEV) model**.

It fetches real market data, builds an implied volatility surface, calibrates a stochastic model, and compares theoretical vs market prices.

---

## 🚀 Features

- 📊 Fetch real-time options data from Yahoo Finance
- 📉 Clean and filter option chains (moneyness, liquidity, maturity filters)
- 📈 Build and visualize implied volatility surface (3D)
- 🧮 Interpolate interest rate curve using US Treasury yields (FRED data)
- 📦 Price European options using:
  - Black-Scholes model
  - Constant Elasticity of Variance (CEV) model
- 🎯 Calibrate model parameters (σ, β) via optimization (MSE minimization)
- 🔍 Compute implied volatilities via interpolation

---

## 🧠 Models Implemented

### 1. Black-Scholes Model
Classic closed-form solution for European call/put pricing.

### 2. Implied Volatility Surface
Constructed using scattered interpolation over:
- Moneyness
- Time to maturity

### 3. Yield Curve Model
- US Treasury data (FRED)
- Nelson-Siegel-Svensson calibration for smooth interpolation

### 4. CEV (Local Volatility) Model
- Captures volatility skew via elasticity parameter β
- Calibrated to market option prices

---

## 📦 Installation

Install dependencies:

```bash
pip install numpy pandas scipy matplotlib scikit-learn
pip install yfinance pandas_datareader
pip install nelson_siegel_svensson
