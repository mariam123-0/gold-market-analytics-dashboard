# Gold Futures Price Analysis & Prediction

An end-to-end data science project on 25 years of daily gold futures prices: data auditing and cleaning, candlestick analytics with `mplfinance`, return-based prediction modeling (Linear Regression and Random Forest, evaluated with `r2_score`), and a Power BI dashboard layer.

![Dashboard mockup](https://github.com/mariam123-0/gold-market-analytics-dashboard/blob/29c39cce59c7adb056087110cbb1f278c5ea7f27/Dashboard.png)

---

## Table of Contents

1. [Overview](#overview)
2. [Key Findings](#key-findings)
3. [Repository Structure](#repository-structure)
4. [Getting Started](#getting-started)
5. [Dataset](#dataset)
6. [Methodology](#methodology)
7. [Results](#results)
8. [Power BI Dashboard](#power-bi-dashboard)
9. [Limitations](#limitations)
10. [Future Work](#future-work)
11. [References](#references)

---

## Overview

Gold is a safe-haven asset, an inflation hedge and an actively traded instrument. This project analyses daily gold futures data from **30 August 2000 to 3 March 2026** (6,399 trading days; USD per troy ounce) and answers three questions:

1. Is the data reliable, and how should it be cleaned?
2. How has gold behaved in terms of trend, risk, volatility and drawdowns?
3. Can a regression model or a Random Forest predict gold's next-day move better than a naive benchmark?

The notebook follows a fixed structure: Introduction, Problem Questions, Objectives, Data Collection, Data Cleaning, Data Visualization, Methods and Result Analysis, Power BI Export, Discussion and Conclusion, References.

## Key Findings

| Area | Finding |
|---|---|
| **Data quality** | No missing values or duplicates, but 441 bars (almost all 2003–2011) had impossible Open/High/Low/Close combinations; these were repaired. 418 zero-volume days and 813 flat bars were flagged, not removed. |
| **Long-run behaviour** | Price rose from about $274 to $5,377 (CAGR 12.4%). Annualised volatility is 17.7%. Maximum drawdown is -44.4% (August 2011 to December 2015). |
| **Return distribution** | Not normal: excess kurtosis is about 7 and the Jarque-Bera test rejects normality, so Gaussian VaR understates tail risk. Volatility clusters in time. |
| **Seasonality** | A one-way ANOVA across calendar months is not significant (p ≈ 0.65). |
| **Prediction** | Predicting price levels gives R² ≈ 0.998 for Linear Regression, but a naive "tomorrow = today" rule scores the same. A Random Forest trained on raw prices fails on the test set (R² ≈ -0.22) because it cannot extrapolate beyond its training range. Modeling next-day **returns** fixes this and gives a fair comparison, where all models explain close to 0% of return variance. |

> **Main takeaway:** a high R² on gold prices is easy to obtain and almost meaningless. The meaningful test is next-day return R² against a naive baseline, and here the models add very little.

## Repository Structure

```
.
├── README.md
├── Gold_Price_Analysis_Rebuilt.ipynb    # Main notebook (analysis, charts, models, export)
├── finalgolddata.csv                    # Source data (daily OHLCV)
├── powerbi_data/                        # Tables exported by the notebook (section 8)
│   ├── gold_daily.csv
│   ├── gold_yearly.csv
│   ├── gold_monthly_returns.csv
│   ├── predictions_test.csv
│   ├── model_metrics.csv
│   ├── model_metrics_price_level.csv
│   ├── feature_importance.csv
│   └── risk_summary.csv
├── gold_dashboard_theme.json            # Power BI colour theme
└── Gold_Power_BI_Dashboard_Mockup.png   # Dashboard design
```

## Getting Started

### Requirements

- Python 3.9 or newer
- `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `scikit-learn`, `mplfinance`
- Jupyter Notebook, JupyterLab or Google Colab

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn mplfinance jupyter
```

### Run the notebook

1. Place `finalgolddata.csv` in the same folder as the notebook (on Colab, upload it when prompted).
2. Open `Gold_Price_Analysis_Rebuilt.ipynb` and choose **Run All**.
3. The notebook writes the Power BI tables to `powerbi_data/`.

The setup cell installs `mplfinance` automatically if it is missing.

## Dataset

Daily gold futures prices. The date range and price level are consistent with the COMEX gold futures contract, but the file carries no metadata, so confirm the data provider before citing it.

| Column | Description |
|---|---|
| `Date` | Trading day (`dd-mm-yy`) |
| `Open`, `High`, `Low`, `Close` | Daily prices in USD per troy ounce |
| `Volume` | Contracts traded |

Volume is thin and unreliable before about 2003, so it is used only as a log-scaled feature.

## Methodology

### 1. Data cleaning
Dates are parsed and sorted; missing values, duplicates and non-positive prices are checked; OHLC logic is validated and repaired (`High = max(O,H,L,C)`, `Low = min(O,H,L,C)`); zero-volume and flat bars are flagged; return outliers are reviewed and kept because they match real market events. Every step is recorded in a cleaning log.

### 2. Visualization and analytics
- **`mplfinance` charts:** candlestick with moving averages and volume, OHLC bars, hollow-and-filled candles, monthly candlesticks, and Renko.
- **Analytics:** calendar-year returns, drawdown, return distribution and tail risk (VaR/CVaR, skewness, kurtosis, Jarque-Bera), volatility clustering, monthly-returns heat-map, and volume and range.

### 3. Prediction modeling
- **Features** (all known at the close of day *t*): lagged and multi-day log-returns, distance from moving averages, rolling volatility, RSI, MACD histogram, Bollinger %B, overnight gap, daily range, candle body, log-volume and calendar terms.
- **Target:** next-day log-return, converted back to a price with `close × exp(predicted return)`.
- **Split:** chronological, first 80% for training and last 20% for testing (2021-02-11 to 2026-03-02). No shuffling.
- **Models:** Linear Regression, Ridge, Random Forest and Gradient Boosting.
- **Validation:** `TimeSeriesSplit` cross-validation (5 folds) and a Random Forest grid search on the training set only.
- **Evaluation:** `r2_score` (primary), plus RMSE, MAE, MAPE and directional accuracy, each compared against naive baselines.

The notebook first shows a **price-level** formulation to demonstrate why it is misleading, and then the **return-level** formulation used for the final comparison.

## Results

### Stage A: next-day price (test set)

| Model | R² |
|---|---|
| Linear Regression | 0.9980 |
| Naive (tomorrow = today) | 0.9980 |
| Random Forest | -0.2186 |

### Stage B: next-day return (test set)

| Model | R² (return) | R² (price) | Directional accuracy |
|---|---|---|---|
| Gradient Boosting | 0.0255 | 0.9982 | 55.3% |
| Random Forest | 0.0111 | 0.9981 | 52.4% |
| Ridge Regression | 0.0095 | 0.9981 | 53.7% |
| Linear Regression | 0.0092 | 0.9981 | 52.8% |
| Random Forest (tuned) | 0.0077 | 0.9981 | 54.7% |
| Always-long baseline | -0.0018 | 0.9981 | 55.4% |
| Naive (zero return) | -0.0060 | 0.9980 | n/a |

**Interpretation.** Gradient Boosting has the highest return R² (about 2.5%), but the result is not robust: cross-validated R² is negative for every model, and directional accuracy does not beat the 55% achieved by always predicting "up". The models do not demonstrate a reliable forecasting edge. This is consistent with the efficient-market view of liquid financial assets.

## Power BI Dashboard

The notebook exports tidy tables to `powerbi_data/`, and the dashboard is built from them. `Power_BI_Dashboard_Guide.md` contains the full data model, all DAX measures and a page-by-page layout.

**Dashboard layout (one page):**

- **KPI row:** latest close, period return, CAGR, annualised volatility, max drawdown, RSI
- **Market:** gold price with 50/200-day averages, calendar-year returns, drawdown
- **Risk and technical:** monthly heat-map, return histogram, rolling volatility, candlestick
- **Models:** R² by model, actual vs predicted price, strategy vs buy-and-hold, feature importance

A `.pbix` file is not included because it can only be authored in Power BI Desktop. Power BI has no built-in candlestick visual; install one from AppSource (*Get more visuals → "candlestick"*).

## Limitations

- A single asset and a single test window (2021 onward), which contains an unusually strong price surge.
- No macroeconomic inputs (real yields, US dollar index, inflation, ETF flows, central-bank purchases).
- The series is a continuous futures contract; roll effects are not modeled.
- Backtests ignore slippage, taxes, financing and contract rolls, and are illustrative only.
- Early-year volume data is unreliable.

## Future Work

- Add macroeconomic and sentiment features.
- Model volatility explicitly (GARCH, HAR-RV).
- Test LSTM or temporal-fusion models with walk-forward re-training.
- Produce probabilistic (quantile) forecasts rather than point estimates.
- Stress-test strategies with realistic trading costs.

## References

1. Baur, D. G., & Lucey, B. M. (2010). Is gold a hedge or a safe haven? *The Financial Review, 45*(2), 217–229.
2. Bergmeir, C., & Benítez, J. M. (2012). On the use of cross-validation for time series predictor evaluation. *Information Sciences, 191*, 192–213.
3. Breiman, L. (2001). Random forests. *Machine Learning, 45*(1), 5–32.
4. Cont, R. (2001). Empirical properties of asset returns: stylized facts and statistical issues. *Quantitative Finance, 1*(2), 223–236.
5. Fama, E. F. (1970). Efficient capital markets: A review of theory and empirical work. *The Journal of Finance, 25*(2), 383–417.
6. Friedman, J. H. (2001). Greedy function approximation: A gradient boosting machine. *Annals of Statistics, 29*(5), 1189–1232.
7. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and Practice* (3rd ed.). OTexts. https://otexts.com/fpp3/
8. Pedregosa, F., et al. (2011). Scikit-learn: Machine learning in Python. *Journal of Machine Learning Research, 12*, 2825–2830.
9. `mplfinance` contributors. *mplfinance: Matplotlib finance utilities.* https://github.com/matplotlib/mplfinance
