---
name: factor-lab
description: Factor analysis and mining framework for Taiwan stock quantitative strategies. Use when the user wants to analyze factors, compute IC, build scorecards, test incremental factor contributions, check factor decay, or explore the factor registry. Triggers on "factor analysis", "IC analysis", "factor mining", "factor scorecard", "factor decay", "incremental test", "因子分析", "因子挖掘", "因子衰退".
argument-hint: <command> [args] - e.g., "ic boss_ratio", "scorecard", "incremental s3v4 ROE", "correlation", "decay", "registry"
user-invocable: true
allowed-tools: Bash(uv run python *), Bash(cd *), Bash(open *), Bash(ls *), Bash(cat *), Read, Grep, Glob, Write, Edit
---

# Factor Lab — 因子分析與挖掘框架

## Execution Philosophy

**You are an executor, not a tutorial.**

When the user asks for factor analysis, produce results on screen — tables, metrics, charts. Write a Python script, execute it with `uv run python`, and show the output. After generating charts, run `open <filepath>` to display them.

## Environment Setup

Every script MUST start with this boilerplate:

```python
import sys
sys.path.insert(0, '/Users/gary/ai-investment/finlabtwstock')

from finlab_util import (auto_login_with_config, use_file_storage,
                         get_inventory_holder, get_rsv, get_gpa,
                         get_broker_buy_sell, calc_rci, get_entry_volatility,
                         apply_stocks_filter, get_candle_volatility_revert_signal,
                         get_daily_index_to_begin_of_next_month, form_position)
from finlab import data, backtest
from finlab.dataframe import FinlabDataFrame
import numpy as np
import pandas as pd
import warnings
warnings.filterwarnings('ignore')

auto_login_with_config()
use_file_storage()
data.prefer_local_if_exists = True

import finlab.data.data as _dd
_dd._role = 'vip'

# numpy 2.x monkey-patch (REQUIRED)
import finlab.backtest as _bt
_orig_arguments = _bt.arguments
def _writable_arguments(*args, **kwargs):
    result = _orig_arguments(*args, **kwargs)
    return [a.copy() if isinstance(a, np.ndarray) and not a.flags.writeable else a for a in result]
_bt.arguments = _writable_arguments
```

**Critical rules:**
- Use `uv run python` to execute (NOT `python3`)
- Working directory: `/Users/gary/ai-investment/finlabtwstock`
- Output directory: `/Users/gary/ai-investment/finlabtwstock/experiment/`
- Set `upload=False` in `backtest.sim()` always
- Strategy names in `backtest.sim()` must be ≤20 chars, English-only or Chinese-first

## Argument Parsing

Parse `$ARGUMENTS` to determine which sub-command to run:

| Pattern | Command | Description |
|---------|---------|-------------|
| `ic <factor>` | IC Analysis | Compute Rank IC for one or more factors |
| `ic all` | Full IC Scan | Rank IC for all 32 factors |
| `scorecard` | Scorecard | Full 6-phase factor mining pipeline |
| `incremental <strategy> [factors]` | Incremental Test | Add factors one-at-a-time to a base strategy |
| `correlation` | Correlation | IC correlation matrix + hierarchical clustering |
| `decay` | Decay Scan | Rolling IC trend + decay diagnosis for all factors |
| `registry` | Registry | List all 32 factors by category |
| (no args) | Help | Show available commands |

## Factor Registry

See [factor-registry.md](factor-registry.md) for the complete 32-factor registry with definitions and compute functions.

The registry covers 6 categories:
- **Value (6):** PB_inv, PE_inv, MktRev_inv, DivYield, EV_EBITDA_inv, FCF_Yield
- **Momentum (8):** RSV_60, RevMom_3M, RevYoY_2M, PriceMom_60/120/240, VolRatio_5_20, Pos_52W
- **Quality (9):** GPA, ROE, ROA, OpMargin, GrossMargin, OpCF_Assets, DebtRatio_inv, CurrentRatio, AssetTurnover
- **LowVol (3):** EntryVol_inv, RetStd60_inv, Beta_inv
- **Size (1):** MktCap_inv
- **TW_Specific (5):** BossRatio, BrokerBSR_Sharpe, RCI26_inv, DirHolding, MarginUsage_inv

## S3v4 Base Strategy Reference

When doing incremental tests against S3v4, use these parameters:

```python
S3V4_WEIGHTS = {
    'boss_ratio': 2.67, 'rsv': 0.65, 'rev_mom': 0.20, 'rev_yoy': 2.25,
    'gpa': 1.42, 'market_rev_ratio': 0.72, 'pb': 1.03,
    'vol_ratio': 0.30, 'broker_bsr_sharpe': 0.20,
}
S3V4_NSTOCKS = 4
S3V4_SL = 0.0667
S3V4_TS = 0.1095
S3V4_TP = 0.5917
```

S3v4 baseline: CAGR 85.1%, MDD -17.0%, Calmar 5.01, Sharpe 2.42

---

## Command: `registry`

Simply print the 32-factor table grouped by category. No data loading needed — output the table from [factor-registry.md](factor-registry.md) directly.

---

## Command: `ic <factor>`

Compute Rank IC for one or more factors.

### Workflow

1. **Load data**: price, adj_close, revenue, and the requested factor(s)
2. **Compute monthly returns**: `adj_close.resample('ME').last().pct_change().shift(-1)` → excess return (subtract market mean)
3. **Compute monthly IC**: For each month, Spearman correlation between factor cross-section and next-month excess return (min 30 valid stocks)
4. **Statistics**:
   - Full-period: Mean IC, Std IC, IR (=Mean/Std), IC Win Rate (% months IC>0)
   - Recent 2Y: Mean IC for last 24 months
   - Recent 1Y: Mean IC for last 12 months
   - Decay diagnosis: compare recent vs full period
5. **Rolling IC chart**: 12-month rolling mean IC with trend line
6. **Output**: Terminal table + PNG chart

### IC Computation Reference

```python
from scipy.stats import spearmanr

# Monthly alignment
monthly_close = adj_close.resample('ME').last()
monthly_return = monthly_close.pct_change().shift(-1)  # forward return
market_return = monthly_return.mean(axis=1)
excess_return = monthly_return.sub(market_return, axis=0)

# Factor → monthly
monthly_factor = factor_daily.reindex(close.index, method='ffill').resample('ME').last()

# IC per month
for dt in common_dates:
    ret_row = excess_return.loc[dt].dropna()
    fac_row = monthly_factor.loc[dt].dropna()
    valid = fac_row.index.intersection(ret_row.index)
    if len(valid) >= 30:
        ic = spearmanr(fac_row[valid], ret_row[valid])[0]
```

### Decay Thresholds

| Recent IC / Full IC | Status |
|---------------------|--------|
| > 120% | Strengthening |
| 80-120% | Stable |
| 50-80% | Mild Decay |
| 20-50% | Significant Decay |
| < 20% | Severe Decay |

---

## Command: `scorecard`

Run the full 6-phase factor mining pipeline. This is the most comprehensive analysis.

### Phase 1: IC/IR Quadrant Analysis
- Compute Mean IC, Std IC, IR for all 32 factors
- Bootstrap CI for IR (5000 resamples)
- Classify into 4 quadrants:
  - High IC + High IR → **Direct Winners**
  - Low IC + High IR → **Hidden Gems** (consistent but weak signal)
  - High IC + Low IR → **Unstable** (noisy)
  - Low IC + Low IR → **Discard**
- Output: Scatter plot (IC vs IR) with quadrant labels

### Phase 2: Mutual Information (Nonlinear)
- Compute MI between each factor and forward return using `sklearn.feature_selection.mutual_info_regression`
- Compare MI rank vs IC rank → large rank difference = nonlinear hidden factor
- Output: IC vs MI scatter plot

### Phase 3: Category Diversity Check
- Ensure min 2 factors per category are in the candidate pool
- Fill gaps using IR ranking within each category

### Phase 4: XGBoost Walk-Forward + RFE
- Walk-forward: 36-month train, 12-month test
- XGBoost feature importance (gain-based)
- Recursive Feature Elimination curve
- Output: Importance bar chart + RFE curve

### Phase 5: Bootstrap Stability
- Split data into 3 time periods
- Bootstrap IR within each period (1000 resamples, 60% sample ratio)
- Stability score = number of periods where IR > 0.3

### Phase 6: Composite Scorecard
- Normalize all metrics to 0-1 scale
- Composite = 0.3×IC_IR + 0.15×MI + 0.2×XGB_importance + 0.2×Bootstrap_stability + 0.15×RFE_persistence
- Output: Ranked table + radar charts for top factors
- Save to: `experiment/factor_mining_scorecard.csv`

### Reference Implementation
Based on: `/Users/gary/ai-investment/finlabtwstock/experiment/factor_mining_engine.py`

---

## Command: `incremental <strategy> [factors]`

Test adding factors one-at-a-time to an existing strategy.

### Arguments
- `<strategy>`: Strategy name (e.g., `s3v4`)
- `[factors]`: Optional comma-separated list of factors to test. Default: top 10 non-strategy factors by composite score.

### Workflow

1. **Baseline backtest**: Run the base strategy, record CAGR/MDD/Calmar/Sharpe
2. **For each candidate factor**:
   - Test weights: [0.3, 0.6, 1.0, 1.5, 2.0]
   - Add factor to base strategy with each weight
   - Record best weight by Calmar
3. **Rank by ΔCalmar** (improvement over baseline)
4. **Combination test**: If ≥2 factors show positive ΔCalmar, test Top-2, Top-3, Top-5 combinations
5. **Output**: Ranked table + bar chart (ΔCalmar) + scatter (CAGR vs MDD)

### Backtest Function Reference

```python
def run_backtest(weights, ranked, nstocks, sl, ts, tp, name):
    score = sum(weights[f] * ranked[f] for f in weights) * stock_filter
    position = score[score > 0].is_largest(nstocks).astype(int)
    position = apply_stocks_filter(position, ky=False, disposal=False, noticed=False, win=10)
    position = position.reindex(rev.index_str_to_date().index, method='ffill').fillna(0)

    ri = resample_index
    buy = position.reindex(ri, method='ffill').fillna(0) > 0
    sell_sig = (cvr_sell | atr_sell).reindex(ri, method='ffill').fillna(False)
    pos_final = form_position(buy, sell_sig)

    report = backtest.sim(pos_final.copy(), name=name[:20],
                          stop_loss=sl, trail_stop=ts, take_profit=tp,
                          fee_ratio=1.425/1000, tax_ratio=3/1000,
                          position_limit=1.0/nstocks, upload=False)
    return report
```

### Stock Filter (S3v4)

```python
rev_yoy_growth_pos = (rev_yoy_growth > 0).astype(int)
price_new_high = (close.rolling(240, min_periods=120).max() == close).astype(int)
vol_cond = (vol.average(10) > 500 * 1000).astype(int)
stock_filter = (price_new_high * rev_yoy_growth_pos * vol_cond).astype(int)
```

### Exit Signals (S3v4)

```python
resample_index = get_daily_index_to_begin_of_next_month()
cvr_sell = get_candle_volatility_revert_signal(short_period=20, vwap_period=60)
atr = data.indicator("ATR", adjust_price=False, resample="D", timeperiod=14)
atr_sell = (close < (close.shift(1) - 3.0 * atr))
```

### Reference Implementation
Based on: `/Users/gary/ai-investment/finlabtwstock/experiment/s3v4_incremental_factor_test.py`

---

## Command: `correlation`

Analyze IC cross-correlations and factor redundancy.

### Workflow

1. **Compute monthly IC** for all 32 factors
2. **IC correlation matrix**: Pairwise Pearson correlation of IC time series
3. **Hierarchical clustering**: distance = 1 - |corr|, average linkage
4. **Identify clusters** at |corr| > 0.6 threshold
5. **Output**:
   - Correlation heatmap (lower triangle, ordered by cluster)
   - Dendrogram
   - Cluster membership table (which factors are redundant)

### Key Thresholds

| |corr| Range | Interpretation |
|-------------|---------------|
| > 0.8 | Highly redundant — keep only one |
| 0.6 - 0.8 | Moderately redundant — consider removing weaker |
| 0.3 - 0.6 | Low redundancy — safe to combine |
| < 0.3 | Independent — ideal for diversification |

### Reference Implementation
Based on: Step 1 of `/Users/gary/ai-investment/finlabtwstock/experiment/s3_factor_rebuild_pipeline.py`

---

## Command: `decay`

Scan all factors for IC decay — identify factors losing predictive power.

### Workflow

1. **Compute rolling 12-month IC** for all 32 factors
2. **For each factor**, compare:
   - Full-period Mean IC
   - Recent 2Y Mean IC
   - Recent 1Y Mean IC
3. **Decay score**: (Recent 1Y IC - Full IC) / |Full IC| × 100%
4. **Categorize**: Strengthening / Stable / Mild Decay / Significant Decay / Severe Decay
5. **Output**:
   - Sorted table by decay severity
   - Rolling IC chart for top decayed factors
   - Recommendations for weight adjustment

### Reference Implementation
Based on: Part 1 of `/Users/gary/ai-investment/finlabtwstock/experiment/s3_factor_refresh.py`

---

## Visualization Standards

All charts must follow these conventions:

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
plt.rcParams['font.sans-serif'] = ['Arial Unicode MS', 'Heiti TC', 'PingFang TC', 'SimHei']
plt.rcParams['axes.unicode_minus'] = False
```

- Save to: `/Users/gary/ai-investment/finlabtwstock/experiment/factor_lab_*.png`
- After saving, run `open <filepath>` to display
- Use consistent color scheme:
  - Value: `#e74c3c`, Momentum: `#3498db`, Quality: `#2ecc71`
  - LowVol: `#9b59b6`, Size: `#f39c12`, TW_Specific: `#1abc9c`

## Output Templates

See [templates.md](templates.md) for standardized output formats for each command.

## Factor Alignment Utilities

### String index to DatetimeIndex + daily alignment

```python
def _align_and_rank(df):
    ff = FinlabDataFrame(df)
    if not isinstance(ff.index, pd.DatetimeIndex):
        try:
            ff = ff.index_str_to_date()
        except Exception:
            ff.index = pd.to_datetime(ff.index, errors='coerce')
            ff = ff[ff.index.notna()]
    return ff.reindex(close.index, method='ffill').rank(axis=1, pct=True)
```

## Previous Research Results

- S3v4 incremental test (2026-02-27): Only ROE showed +ΔCalmar (+0.21), at cost of -5.3% CAGR
- Orthogonalized 12-factor "new S3" underperformed (Calmar 2.22 vs 5.01)
- Key insight: IC composite score does NOT equal incremental strategy value
- Full results: `/Users/gary/ai-investment/finlabtwstock/experiment/s3v4_incremental_results.json`
- Factor scorecard: `/Users/gary/ai-investment/finlabtwstock/experiment/factor_mining_scorecard.csv`

## Error Handling

| Error | Response |
|-------|----------|
| Factor not found | List similar factor names from registry, suggest corrections |
| Data loading fails | Check finlab login, retry with `auto_login_with_config()` |
| Backtest fails | Check numpy monkey-patch applied, ensure `upload=False` |
| Too few stocks | Lower `min_valid` threshold or report sparse data warning |
| Chart font issues | Verify matplotlib font config, fall back to English labels |
