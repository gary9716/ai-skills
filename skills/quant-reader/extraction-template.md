# Extraction Template

Use this template to structure the output when analyzing a quant paper or book chapter.

---

## Output Template

```markdown
# [Paper Title]

## Metadata
- **Authors**: [Names]
- **Date**: [Publication date]
- **Publication**: [Journal / working paper / arXiv]
- **DOI/Link**: [DOI or URL]
- **Pages**: [Total page count]

## Abstract / Core Thesis
> [2-3 sentence summary of the paper's main claim and contribution]

## Methodology

### Model Specification
[Core model or framework used]

### Estimation Approach
[How parameters are estimated - OLS, GMM, MLE, Fama-MacBeth, etc.]

### Key Assumptions
- [Assumption 1]
- [Assumption 2]

### Robustness Checks
- [Alternative specification 1]
- [Alternative specification 2]

## Key Formulas

| # | Formula | Variables | Context |
|---|---------|-----------|---------|
| 1 | $r_{i,t} = \alpha_i + \beta_i f_t + \epsilon_{i,t}$ | $r$ = excess return, $f$ = factor, $\epsilon$ = residual | Main model |
| 2 | ... | ... | ... |

## Data Sources

| Dataset | Period | Universe | Frequency | Filters |
|---------|--------|----------|-----------|---------|
| CRSP | 1963-2019 | US equities | Monthly | Exclude micro-caps |
| ... | ... | ... | ... | ... |

## Key Findings

1. **[Finding 1]**: [Description] (t-stat = X.XX, p < 0.0X)
2. **[Finding 2]**: [Description] (t-stat = X.XX)
3. ...

### In-Sample vs Out-of-Sample
- **In-sample period**: [dates] — [summary of results]
- **Out-of-sample period**: [dates] — [summary of results]

## Tables & Figures

### Table X: [Title]
[Extracted table data or description of content and key takeaways]

### Figure Y: [Title]
[Description of what the figure shows and main insight]

## Backtesting Results

| Metric | Long | Short | Long-Short | Benchmark |
|--------|------|-------|------------|-----------|
| Annualized Return | | | | |
| Annualized Volatility | | | | |
| Sharpe Ratio | | | | |
| Information Ratio | | | | |
| Max Drawdown | | | | |
| Annualized Turnover | | | | |
| Transaction Costs (est.) | | | | |

## Implementation Notes

- **Data requirements**: [What data feeds/vendors are needed]
- **Signal construction**: [Steps to build the trading signal]
- **Rebalancing**: [Frequency and methodology]
- **Universe**: [What assets to trade]
- **Capacity**: [Estimated strategy capacity]
- **Decay**: [How quickly does the signal decay]
- **Transaction costs**: [Impact on net returns]
- **Complexity**: [Simple / Moderate / Complex]
- **Key risks**: [Main implementation risks]

## Limitations & Caveats
- [Limitation 1]
- [Limitation 2]

### Red Flags
- [ ] Data mining / p-hacking concerns
- [ ] Survivorship bias
- [ ] Lookahead bias
- [ ] Unrealistic assumptions
- [ ] Narrow sample period
- [ ] No out-of-sample test
- [ ] No transaction cost analysis

## References of Interest
- **[Author (Year)]** — [Title] — [Why it's relevant]
- ...
```

---

## Section Definitions

### Metadata
Extract from the first page and PDF metadata. For arXiv papers, note the arXiv ID. For journal papers, note the journal name and volume.

### Abstract / Core Thesis
The paper's central claim in 2-3 sentences. Not a copy of the abstract — a distillation of what the paper contributes and why it matters.

### Methodology
The technical approach. Identify:
- **Study type**: cross-sectional, time-series, panel, event study, simulation
- **Model**: regression, factor model, optimization, machine learning
- **Estimation**: OLS, GLS, GMM, MLE, Bayesian, Fama-MacBeth, portfolio sorts
- **Standard errors**: Newey-West, clustered, bootstrapped, White robust

### Key Formulas
Every important equation. Always define variables with units. Note equation numbers from the paper for cross-referencing.

### Data Sources
Be specific about:
- Exact dataset name (not just "stock data")
- Start and end dates
- Geographic scope
- Asset class and universe filters
- Data frequency (daily, monthly, quarterly)
- Any exclusions (penny stocks, financials, utilities)

### Key Findings
Quantitative results with statistical evidence. Always note:
- Point estimates with t-statistics or confidence intervals
- Economic significance (not just statistical significance)
- Whether findings hold in subperiods
- Monotonicity across quantile portfolios (for factor papers)

### Backtesting Results
Performance metrics in a standardized table. Convert to annual figures when the paper reports monthly. Flag if gross or net of transaction costs.

### Implementation Notes
The "so what" — translate academic findings into practical trading considerations. This is the highest-value section for practitioners.

### Red Flags
Check each item. A paper with multiple red flags warrants skepticism about real-world applicability.

---

## Quant Finance Glossary

Terms to recognize and extract with precision:

### Return Metrics
- **Alpha ($\alpha$)**: Excess return above benchmark or factor model
- **Beta ($\beta$)**: Sensitivity to a factor or market return
- **Sharpe Ratio**: $SR = \frac{E[R - R_f]}{\sigma}$ — risk-adjusted return
- **Information Ratio**: $IR = \frac{\alpha}{\sigma_\epsilon}$ — alpha per unit of tracking error
- **Max Drawdown**: Largest peak-to-trough decline
- **Calmar Ratio**: Annualized return / max drawdown
- **Sortino Ratio**: Return / downside deviation

### Factor Models
- **CAPM**: $r_i - r_f = \alpha + \beta(r_m - r_f) + \epsilon$
- **Fama-French 3-Factor**: Market, SMB (size), HML (value)
- **Carhart 4-Factor**: + UMD (momentum)
- **Fama-French 5-Factor**: + RMW (profitability), CMA (investment)
- **Factor loading**: Regression coefficient on a factor
- **Factor premium**: Average return of long-short factor portfolio

### Statistical Methods
- **Fama-MacBeth regression**: Two-pass cross-sectional regression
- **Newey-West standard errors**: HAC-consistent standard errors
- **GRS test**: Joint test of multiple alphas
- **t-statistic threshold**: Harvey, Liu, Zhu (2016) suggest t > 3.0 for new factors
- **Bonferroni correction**: Multiple testing adjustment
- **Walk-forward validation**: Rolling out-of-sample testing

### Portfolio Construction
- **Long-short portfolio**: Long top quantile, short bottom quantile
- **Decile sorts**: Sort into 10 groups by signal
- **Value-weighted**: Weighted by market capitalization
- **Equal-weighted**: Equal allocation across holdings
- **Turnover**: Fraction of portfolio replaced per period
- **Rebalancing frequency**: How often the portfolio is reconstructed

### Risk Measures
- **VaR (Value at Risk)**: Loss threshold at a confidence level
- **CVaR / Expected Shortfall**: Average loss beyond VaR
- **Tracking error**: Standard deviation of returns vs benchmark
- **Idiosyncratic risk**: Risk not explained by factors

---

## Common Data Sources

| Source | Coverage | Typical Use |
|--------|----------|-------------|
| **CRSP** | US equities, 1926-present | Returns, prices, shares outstanding |
| **Compustat** | US/global fundamentals | Accounting data, financial statements |
| **IBES** | Analyst forecasts | Earnings estimates, recommendations |
| **TAQ** | US tick data | Intraday prices, trades, quotes |
| **OptionMetrics** | US options | Implied volatility, option prices |
| **Bloomberg** | Global multi-asset | Prices, fundamentals, fixed income |
| **Refinitiv/LSEG** | Global multi-asset | Prices, fundamentals, ESG |
| **Kenneth French Library** | Factor returns | Fama-French factors, industry portfolios |
| **AQR Data Library** | Factor returns | HML Devil, BAB, QMJ, time-series momentum |
| **FRED** | Macro data | Interest rates, GDP, inflation |
| **TEJ** | Taiwan equities | Prices, fundamentals, corporate actions |
| **WRDS** | Research platform | Aggregates CRSP, Compustat, IBES, TAQ |

---

## Formula Notation Guide

Use LaTeX-style notation in markdown:

| Concept | Notation |
|---------|----------|
| Subscripts | `$r_{i,t}$` → $r_{i,t}$ |
| Superscripts | `$R^2$` → $R^2$ |
| Greek letters | `$\alpha, \beta, \gamma, \sigma, \mu, \epsilon$` |
| Summation | `$\sum_{i=1}^{N}$` |
| Expectation | `$E[X]$` or `$\mathbb{E}[X]$` |
| Matrix | `$\Sigma$` for covariance matrix |
| Absolute value | `$|r_t|$` |
| Hat (estimate) | `$\hat{\beta}$` |
| Bar (average) | `$\bar{r}$` |
| Fraction | `$\frac{a}{b}$` |
