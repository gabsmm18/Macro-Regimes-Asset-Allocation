# Macroeconomic Regimes and Tactical Asset Allocation

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/gabsmm18/macro-regimes-asset-allocation/blob/main/macro_regimes_asset_allocation.ipynb)

**Author:** Mattia Mariani

## Research question

Does the macroeconomic regime (growth and inflation accelerating or decelerating) help explain, and exploit, the returns of different asset classes?

The project tests the classic four-regime framework, the idea behind Merrill Lynch's "Investment Clock" and Bridgewater's "All Weather" portfolio, on US data from 2007 to today, and asks whether macro information can improve a standard 60/40 allocation.

## Framework

Each month is classified by the **direction** of growth and inflation relative to their trailing 12-month average:

| | Inflation ↓ | Inflation ↑ |
|---|---|---|
| **Growth ↑** | Goldilocks | Reflation |
| **Growth ↓** | Deflation | Stagflation |

Theory suggests equities do best in Goldilocks, commodities in Reflation, gold in Stagflation and government bonds in Deflation. The project checks whether the data agree.

## Data

| Source | Series | Role |
|---|---|---|
| FRED | Industrial production (YoY) | Growth |
| FRED | CPI (YoY) | Inflation |
| FRED | Unemployment rate | Labour-market cycle |
| FRED | Effective Fed Funds rate | Monetary policy |
| FRED | 10Y – 2Y Treasury spread | Yield-curve slope |
| FRED | BAA – 10Y spread | Credit risk |
| Yahoo Finance | SPY, IEF, GLD, DBC, UUP | US equities, 7-10Y Treasuries, gold, commodities, US dollar |

All data are free and downloaded automatically by the notebook. No API key is required.

## Methodology

1. **Avoiding look-ahead bias.** Monthly macro releases are lagged by one month to reflect publication delays (e.g. January CPI is released in mid-February). Market-based spreads are known at month end and are not lagged.
2. **Regime analysis.** The regime known at the end of month *t* is matched with asset returns in month *t+1*. For each asset-regime pair the notebook reports annualised return, volatility and the t-statistic of the mean.
3. **Rule-based strategy.** Portfolio weights for each regime are fixed ex ante from economic theory, not fitted to the data, then backtested net of 10 bp transaction costs against a 60/40, an equal-weight portfolio and equities only. A sensitivity analysis varies costs from 0 to 50 bp.
4. **Machine learning.** Logistic regression and a random forest predict whether equities will beat Treasuries in the following month, using macro levels, 3-month changes and 12-month relative momentum. Models are evaluated in an **expanding-window walk-forward** test (re-trained every month on past data only) and compared with a naive "most frequent class" benchmark.

## Results

![Regime timeline](figures/02_regime_timeline.png)

> **Summary:** the macro regime helps less to earn more than to lose less. A simple strategy based on growth and inflation halved the maximum drawdown of a 60/40 portfolio at the same return, while machine learning added no value over a naive benchmark.

### 1. Asset returns by regime

![Regime heatmap](figures/03_regime_heatmap.png)

Annualised return in the month after the regime signal (* = |t| > 2):

| Regime | Equities | Treasuries | Gold | Commodities |
|---|---|---|---|---|
| Goldilocks | +21%* | +2% | +21%* | +17%* |
| Reflation | +12%* | +5% | +10% | +9% |
| Stagflation | −12% | +6% | +5% | −12% |
| Deflation | +20%* | +1% | +7% | +3% |

- **Stagflation is the clearest result.** It is the only regime in which equities and commodities lose money, while Treasuries and gold hold up, as the theory predicts. Equity volatility almost doubles (21% vs 11-15% in the other regimes).
- **Goldilocks is good for almost everything**: equities, gold and commodities all deliver significant returns.
- **Growth regimes favour equities**, with statistically significant returns in Goldilocks and Reflation.
- **Deflation is the surprise.** Theory points to Treasuries, but equities performed best. A plausible explanation is the post-2008 policy reaction: in slowdowns the Fed cut rates and ran QE, and the liquidity supported equities above all.

### 2. Regime strategy vs benchmarks

![Strategy backtest](figures/04_strategy_backtest.png)

| Strategy | Annual return | Volatility | Sharpe | Max drawdown |
|---|---|---|---|---|
| **Macro regimes** | 7.9% | 8.9% | 0.90 | **−12.4%** |
| 60/40 | 8.0% | 9.5% | 0.86 | −29.5% |
| Equal weight | 6.2% | 6.7% | 0.93 | −18.0% |
| Equities only | 10.7% | 15.4% | 0.74 | −50.8% |

*Net of 10 bp transaction costs. Sharpe ratio computed without the risk-free rate.*

The regime strategy earns about the same as a 60/40 (7.9% vs 8.0% a year) with **less than half its maximum drawdown** (−12% vs −29%). Its value lies in avoiding crashes, mainly by reducing equity exposure when Stagflation arrives, rather than in higher returns.

The Sharpe ratio advantage is small (0.90 vs 0.86), lower than a simple equal-weight portfolio (0.93), and disappears with transaction costs above about 20 bp (Sharpe 0.81 at 30 bp). **The robust result is the lower drawdown, not a better risk-adjusted return.**

### 3. Machine learning (out-of-sample)

![ML backtest](figures/05_ml_backtest.png)

| Model | Out-of-sample accuracy |
|---|---|
| Naive (most frequent class) | 65.6% |
| Logistic regression | 62.6% |
| Random forest | 64.4% |

| Strategy (Apr 2013 - Sep 2026) | Annual return | Volatility | Sharpe | Max drawdown |
|---|---|---|---|---|
| ML: Logistic | 9.0% | 9.9% | 0.93 | −17.3% |
| ML: Random forest | 10.4% | 11.1% | 0.95 | −26.4% |
| Macro regimes | 7.6% | 8.4% | 0.91 | **−10.9%** |
| 60/40 | 9.1% | 9.2% | **0.99** | −20.5% |

Equities beat Treasuries in 63% of months, so a naive model that always predicts "equities" is right 66% of the time. **Neither ML model beats this benchmark.** The random forest earns the most in the backtest, but with the largest drawdown, and neither model beats the Sharpe ratio of a passive 60/40. Over this period, dominated by a long equity bull market, the 60/40 has the best risk-adjusted return, while the macro rules again have the smallest drawdown. With few monthly observations and a low signal-to-noise ratio, the models fit relationships that do not persist out of sample, while simple theory-driven rules proved more robust.

## Limitations

- **Data revisions:** FRED provides revised data, not the real-time vintages markets observed. A natural extension is to use ALFRED vintages.
- **Missing data:** October 2025 CPI was never published because of the US government shutdown. The missing month is linearly interpolated.
- **Hindsight in the rules:** portfolio weights were set from economic theory, but the theory itself is partly shaped by past episodes in the sample (e.g. 2008, 2022).
- **Short sample:** about 19 years split across four regimes leaves limited observations per regime.
- **Researcher choices:** the regime threshold, portfolio weights and model parameters are discretionary, which creates a risk of data snooping.
- **Simplified costs:** drift rebalancing, variable bid-ask spreads and taxes are ignored.
- **US only:** results may not generalise to other markets.

## Possible extensions

- Macro **surprises** (actual minus consensus) instead of levels
- **Monetary-policy regimes** (hiking vs cutting cycles)
- Euro-area replication with ECB / Eurostat data
- Statistical regime detection with a **Hidden Markov Model**

