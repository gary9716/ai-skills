# Quant Reader Examples

## Example 1: Factor Paper — Full Extraction

**User:** `/quant-reader ~/papers/value-and-momentum-everywhere.pdf`

**Output:**

---

# Value and Momentum Everywhere

## Metadata
- **Authors**: Asness, Moskowitz, Pedersen
- **Date**: 2013
- **Publication**: Journal of Finance, Vol. 68, No. 3
- **DOI**: 10.1111/jofi.12021
- **Pages**: 44

## Abstract / Core Thesis
> Value and momentum returns are positively correlated across asset classes (equities, bonds, currencies, commodities). A diversified portfolio combining value and momentum across all markets produces large, significant risk-adjusted returns that cannot be explained by standard risk factors. The negative correlation between value and momentum within each asset class provides natural hedging benefits.

## Methodology

### Model Specification
Time-series and cross-sectional factor regressions across 8 markets spanning 4 asset classes.

### Estimation Approach
- Portfolio sorts: assets ranked into terciles by value/momentum signals
- Long-short portfolios: long top tercile, short bottom tercile
- Fama-MacBeth cross-sectional regressions for risk premium estimation
- Time-series regressions against global factors

### Key Assumptions
- Sufficient liquidity in all markets for long-short implementation
- Transaction costs do not eliminate returns
- Short selling is feasible across all asset classes

### Robustness Checks
- Subperiod analysis (pre-1990 vs post-1990)
- Excluding individual markets one at a time
- Alternative value and momentum signal definitions
- Controlling for macroeconomic variables (GDP growth, inflation)

## Key Formulas

| # | Formula | Variables | Context |
|---|---------|-----------|---------|
| 1 | $VAL_t = \frac{P_{t-48}}{P_t}$ (equities variant) | $P$ = price level, 48-month lookback | Value signal |
| 2 | $MOM_t = r_{t-12,t-1}$ | 12-month return skipping most recent month | Momentum signal |
| 3 | $r_{i,t} = \alpha + \beta_{MKT} MKT_t + \beta_{VAL} VAL_t + \beta_{MOM} MOM_t + \epsilon$ | Standard factor regression | Spanning test |

## Data Sources

| Dataset | Period | Universe | Frequency |
|---------|--------|----------|-----------|
| CRSP/Compustat | 1972-2011 | US equities | Monthly |
| MSCI | 1972-2011 | Global equity indices (18 countries) | Monthly |
| Barclays | 1972-2011 | Government bonds (10 countries) | Monthly |
| Interbank rates | 1972-2011 | Currencies (10 pairs) | Monthly |
| GSCI | 1972-2011 | Commodity futures (27 contracts) | Monthly |

## Key Findings

1. **Value works everywhere**: Positive returns in equities (t=3.3), bonds (t=2.1), currencies (t=2.6), commodities (t=2.0)
2. **Momentum works everywhere**: Positive returns in equities (t=4.4), bonds (t=2.5), currencies (t=3.1), commodities (t=3.6)
3. **Negative correlation within**: Value and momentum are negatively correlated within each asset class (avg ρ ≈ -0.5)
4. **Positive correlation across**: Value in one asset class is correlated with value in others; same for momentum
5. **Diversified combo outperforms**: Global diversified value+momentum portfolio: Sharpe ~1.0

## Backtesting Results

| Metric | Value (global) | Momentum (global) | 50/50 Combo |
|--------|---------------|-------------------|-------------|
| Annualized Return | 5.1% | 7.3% | 6.2% |
| Annualized Volatility | 6.5% | 8.2% | 5.1% |
| Sharpe Ratio | 0.78 | 0.89 | 1.22 |
| Max Drawdown | -23% | -45% | -18% |

## Implementation Notes

- **Data requirements**: Global equity indices, government bond yields, FX rates, commodity futures — available from Bloomberg or Refinitiv
- **Signal construction**: Simple — rank assets by value (book-to-market or yield) and momentum (past 12-1 month return)
- **Rebalancing**: Monthly
- **Universe**: 4 asset classes, ~50+ instruments minimum
- **Capacity**: Very high — uses liquid futures and country indices
- **Complexity**: Simple
- **Key risks**: Momentum crashes (2009), value drawdowns during rate-hiking cycles

## Limitations & Caveats
- Transaction costs not fully modeled (assumed low for liquid instruments)
- Some commodities may have limited capacity
- Value signal definition varies across asset classes (not truly universal)

### Red Flags
- [ ] Data mining concerns — results span 4 asset classes, reducing p-hacking risk
- [ ] Survivorship bias — uses broad indices, less susceptible
- [x] No transaction cost analysis — gross returns only

## References of Interest
- **Jegadeesh & Titman (1993)** — Returns to Buying Winners — Original momentum paper
- **Fama & French (1998)** — Value Premium in International Returns — International value evidence
- **Barroso & Santa-Clara (2015)** — Momentum has its moments — Momentum crash risk management

---

## Example 2: Book Chapter — Page Range Extraction

**User:** `/quant-reader ~/books/advances-financial-ml.pdf pages 45-65`

**Output:**

---

# Advances in Financial Machine Learning — Chapter 3: Bars

## Metadata
- **Authors**: Marcos López de Prado
- **Date**: 2018
- **Publication**: Wiley
- **Pages**: 45-65 (Chapter 3 of 25 chapters)

## Abstract / Core Thesis
> Standard time bars (OHLCV at fixed intervals) are a poor choice for financial ML because they oversample low-activity periods and undersample high-activity periods. Alternative bar types — tick bars, volume bars, and dollar bars — produce more normally distributed returns and better statistical properties for ML applications.

## Methodology

### Approach
Empirical comparison of bar types using tick-level data. Statistical tests for normality (Jarque-Bera) and serial correlation applied to returns from each bar type.

### Bar Types Analyzed
1. **Time bars**: Fixed time intervals (1min, 5min, 1hr, daily)
2. **Tick bars**: Fixed number of ticks per bar
3. **Volume bars**: Fixed volume per bar
4. **Dollar bars**: Fixed dollar volume per bar
5. **Information-driven bars**: Imbalance bars, run bars

## Key Formulas

| # | Formula | Variables | Context |
|---|---------|-----------|---------|
| 1 | $T = \{t : V_t \geq \bar{V}\}$ | $V_t$ = cumulative volume, $\bar{V}$ = threshold | Volume bar sampling |
| 2 | $T = \{t : D_t \geq \bar{D}\}$ | $D_t$ = cumulative dollar volume, $\bar{D}$ = threshold | Dollar bar sampling |
| 3 | $\theta_T = \sum_{t=1}^{T} b_t v_t$ | $b_t$ = tick direction (+1/-1), $v_t$ = volume | Tick imbalance |

## Key Findings

1. **Dollar bars produce most normal returns**: Jarque-Bera statistic is lowest for dollar bars across all tested securities
2. **Time bars oversample**: 80%+ of time bars during US trading hours contain minimal information
3. **Volume bars adjust for activity**: Naturally produce more bars during high-volatility periods
4. **Serial correlation reduced**: Tick and volume bars show significantly less serial autocorrelation than time bars

## Implementation Notes

- **Data requirements**: Tick-level trade data (TAQ, exchange feeds, or crypto websocket)
- **Signal construction**: Replace time-based OHLCV with dollar bars before any feature engineering
- **Rebalancing**: N/A — this is a data preprocessing technique, not a strategy
- **Complexity**: Moderate — requires tick data infrastructure and bar generation code
- **Key risks**: Threshold selection is somewhat arbitrary; need to recalibrate as dollar volume changes over time

## Limitations & Caveats
- All analysis done on limited number of securities (E-mini S&P 500 futures)
- Bar threshold selection not rigorously optimized
- No analysis of impact on downstream ML model performance (addressed in later chapters)

---

## Example 3: Section-Specific — Methodology Deep Dive

**User:** `/quant-reader ~/papers/black-litterman.pdf methodology`

**Output:**

---

# Methodology: Black-Litterman Model

## Study Type
Portfolio optimization framework combining equilibrium (CAPM) with investor views.

## Model Specification

The Black-Litterman model solves the problem of unstable mean-variance optimization by using the market equilibrium as a starting point and blending it with investor views.

### Step 1: Implied Equilibrium Returns
$$\Pi = \delta \Sigma w_{mkt}$$

| Variable | Definition |
|----------|-----------|
| $\Pi$ | Implied equilibrium excess returns (N×1 vector) |
| $\delta$ | Risk aversion coefficient (typically $\delta = \frac{E[r_m - r_f]}{\sigma_m^2}$) |
| $\Sigma$ | Covariance matrix of returns (N×N) |
| $w_{mkt}$ | Market capitalization weights (N×1 vector) |

### Step 2: Express Views
$$P \cdot \mu = Q + \epsilon, \quad \epsilon \sim N(0, \Omega)$$

| Variable | Definition |
|----------|-----------|
| $P$ | Pick matrix linking views to assets (K×N) |
| $Q$ | View returns (K×1 vector) |
| $\Omega$ | Uncertainty of views (K×K diagonal matrix) |
| $K$ | Number of views |

### Step 3: Posterior Combined Return
$$E[R] = [(\tau\Sigma)^{-1} + P'\Omega^{-1}P]^{-1} [(\tau\Sigma)^{-1}\Pi + P'\Omega^{-1}Q]$$

| Variable | Definition |
|----------|-----------|
| $\tau$ | Scalar indicating uncertainty in equilibrium (typically 0.025-0.05) |
| $E[R]$ | Blended expected returns combining equilibrium and views |

### Step 4: Posterior Covariance
$$\bar{\Sigma} = \Sigma + [(\tau\Sigma)^{-1} + P'\Omega^{-1}P]^{-1}$$

### Key Assumptions
1. Market is in equilibrium — current market-cap weights are optimal for the "average" investor
2. Views are independent (diagonal $\Omega$)
3. Returns are normally distributed
4. $\tau$ is small — equilibrium is a strong prior
5. Covariance matrix $\Sigma$ is known with certainty (separate from $\tau$ uncertainty on means)

### Estimation Procedure
1. Compute market-cap weights from current prices
2. Estimate $\Sigma$ from historical returns (recommend shrinkage estimator)
3. Choose $\delta$ from market risk premium and variance
4. Compute equilibrium returns $\Pi$
5. Formulate views as $(P, Q, \Omega)$ triplet
6. Compute posterior $E[R]$ using the master formula
7. Run mean-variance optimization with $E[R]$ and $\bar{\Sigma}$

### Comparison to Alternatives
| Approach | Advantage | Disadvantage |
|----------|-----------|--------------|
| Mean-variance (Markowitz) | Theoretically optimal | Unstable, extreme weights |
| Black-Litterman | Stable, intuitive | Requires view specification |
| Risk parity | No return estimates needed | Ignores expected returns |
| 1/N (equal weight) | Simplest, robust | Ignores all information |

---

## Example 4: Dense Table Extraction with pdfplumber Fallback

**User:** `/quant-reader ~/papers/factor-returns-2023.pdf tables`

**Execution flow:**

```
Phase 1: Read pages 1-3 to identify structure
  → Paper has 8 tables, most contain dense numerical data

Phase 2: Read full paper to understand context for each table

Phase 3: Read tool shows garbled table alignment for Table 3 (correlation matrix)
  → Fall back to pdfplumber:

$ python3 -c "
import pdfplumber
with pdfplumber.open('factor-returns-2023.pdf') as pdf:
    page = pdf.pages[11]  # Table 3 is on page 12
    tables = page.extract_tables()
    for row in tables[0]:
        print(' | '.join(str(c or '') for c in row))
"
```

**Output:**

---

# Tables: Factor Returns 2023

## Table 1: Summary Statistics (p. 5)

| Factor | Mean (%) | Std (%) | Sharpe | Skew | Kurt | t-stat |
|--------|----------|---------|--------|------|------|--------|
| MKT | 8.2 | 15.3 | 0.54 | -0.62 | 3.45 | 3.21 |
| SMB | 2.1 | 10.8 | 0.19 | 0.31 | 4.12 | 1.17 |
| HML | 3.4 | 9.7 | 0.35 | 0.18 | 3.89 | 2.10 |
| UMD | 6.8 | 14.5 | 0.47 | -1.42 | 8.76 | 2.81 |
| RMW | 3.1 | 7.2 | 0.43 | -0.22 | 4.56 | 2.58 |
| CMA | 2.3 | 6.8 | 0.34 | 0.15 | 3.21 | 2.03 |

**Takeaway**: MKT and UMD deliver highest absolute returns; RMW has best risk-adjusted performance per unit volatility. UMD shows extreme negative skew and kurtosis (momentum crash risk).

## Table 3: Factor Correlation Matrix (p. 12)

*Extracted via pdfplumber — Read tool output was misaligned*

| | MKT | SMB | HML | UMD | RMW | CMA |
|-----|------|------|------|------|------|------|
| MKT | 1.00 | 0.28 | -0.27 | -0.14 | -0.23 | -0.38 |
| SMB | 0.28 | 1.00 | -0.10 | -0.02 | -0.37 | -0.05 |
| HML | -0.27 | -0.10 | 1.00 | -0.53 | 0.08 | 0.70 |
| UMD | -0.14 | -0.02 | -0.53 | 1.00 | 0.11 | -0.03 |
| RMW | -0.23 | -0.37 | 0.08 | 0.11 | 1.00 | 0.07 |
| CMA | -0.38 | -0.05 | 0.70 | -0.03 | 0.07 | 1.00 |

**Takeaway**: Strong negative correlation between HML and UMD (-0.53) confirms the value-momentum tension. HML and CMA highly correlated (0.70) — adding both to a model may cause multicollinearity. MKT negatively correlated with most factors — diversification benefit.

## Table 6: Subperiod Analysis (p. 18)

| Period | HML Return | UMD Return | Notes |
|--------|-----------|-----------|-------|
| 1963-1980 | 5.8% | 8.1% | Both strong |
| 1981-2000 | 4.2% | 9.3% | Momentum peak era |
| 2001-2010 | 1.1% | 2.4% | Value struggles, momentum crash (2009) |
| 2011-2023 | -0.3% | 5.2% | Value drought, growth dominance |

**Takeaway**: HML (value) returns have declined monotonically across subperiods. The 2011-2023 period shows negative value returns — driven by growth stock dominance (FAANG era). Momentum remains more stable but vulnerable to sharp reversals.
