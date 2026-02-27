# Factor Registry — 32 因子定義

## Data Loading

```python
d = {}
d['close'] = data.get('price:收盤價')
d['adj_close'] = data.get('etl:adj_close')
d['volume'] = data.get('price:成交股數')
d['rev'] = data.get('monthly_revenue:當月營收')
d['rev_yoy_pct'] = data.get('monthly_revenue:去年同月增減(%)')
d['pb'] = data.get('price_earning_ratio:股價淨值比')
d['pe'] = data.get('price_earning_ratio:本益比')
d['yield_ratio'] = data.get('price_earning_ratio:殖利率(%)')
d['market_val'] = data.get('etl:market_value')
d['total_assets'] = data.get('financial_statement:資產總額')
d['gross_profit'] = data.get('financial_statement:營業毛利')
d['roe'] = data.get('fundamental_features:ROE稅後')
d['roa'] = data.get('fundamental_features:ROA稅後息前')
d['op_margin'] = data.get('fundamental_features:營業利益率')
d['gross_margin'] = data.get('fundamental_features:營業毛利率')
d['debt_ratio'] = data.get('fundamental_features:負債比率')
d['current_ratio'] = data.get('fundamental_features:流動比率')
d['asset_turnover'] = data.get('fundamental_features:總資產週轉次數')
d['dir_holding'] = data.get('internal_equity_changes:董監持有股數占比')

# Optional (may fail)
d['op_cashflow'] = data.get('fundamental_features:營運現金流')       # may raise
d['noncurrent_debt'] = data.get('financial_statement:非流動負債')     # may raise
d['ebitda'] = data.get('fundamental_features:EBITDA')                # may raise
d['margin_usage'] = data.get('margin_transactions:融資使用率')       # may raise

# Computed
d['boss_ratio'] = get_inventory_holder(start_level=9)
d['rsv'] = get_rsv(60)
d['gpa'] = get_gpa()
d['entry_vol'] = get_entry_volatility()
d['broker_buy_sell'] = get_broker_buy_sell()
broker_bsr = d['broker_buy_sell']['buy'] / d['broker_buy_sell']['sell'].replace(0, np.nan)
broker_bsr_std = broker_bsr.rolling(20).std().replace(0, np.nan)
d['broker_bsr_sharpe'] = broker_bsr.rolling(20).mean() / broker_bsr_std
d['rci_26'] = calc_rci(d['adj_close'], period=26)
```

## Factor Definitions

### Value (6)

| Name | Display | Compute | Direction |
|------|---------|---------|-----------|
| `PB_inv` | PB反轉 | `-d['pb']` | Lower PB = higher score |
| `PE_inv` | PE反轉 | `-d['pe']` | Lower PE = higher score |
| `MktRev_inv` | 市值營收比反轉 | `-(d['market_val'] / d['rev'].rolling(4).sum())` | Lower mkt/rev = higher score |
| `DivYield` | 殖利率 | `d['yield_ratio']` | Higher yield = higher score |
| `EV_EBITDA_inv` | EV/EBITDA反轉 | `-(d['market_val'] + d['noncurrent_debt']) / d['ebitda']` | Lower EV/EBITDA = higher score |
| `FCF_Yield` | FCF收益率 | `d['op_cashflow'] / d['market_val']` | Higher FCF yield = higher score |

### Momentum (8)

| Name | Display | Compute | Direction |
|------|---------|---------|-----------|
| `RSV_60` | RSV(60) | `get_rsv(60)` | Higher RSV = stronger momentum |
| `RevMom_3M` | 營收月增 | `rev.average(3) / rev.average(3).shift()` | MoM revenue growth |
| `RevYoY_2M` | 營收年增 | `rev.average(2) / rev.average(2).shift(12)` | YoY revenue growth |
| `PriceMom_60` | 價格動能60 | `adj_close / adj_close.shift(60) - 1` | 60-day price return |
| `PriceMom_120` | 價格動能120 | `adj_close / adj_close.shift(120) - 1` | 120-day price return |
| `PriceMom_240` | 價格動能240 | `adj_close / adj_close.shift(240) - 1` | 240-day price return |
| `VolRatio_5_20` | 量比 | `volume.average(5) / volume.average(20)` | Short-term volume surge |
| `Pos_52W` | 52週位置 | `(close - 240d_low) / (240d_high - 240d_low)` | Position in 52-week range |

### Quality (9)

| Name | Display | Compute | Direction |
|------|---------|---------|-----------|
| `GPA` | GPA | `get_gpa()` | Gross Profitability to Assets |
| `ROE` | ROE | `d['roe']` | Return on Equity |
| `ROA` | ROA | `d['roa']` | Return on Assets |
| `OpMargin` | 營業利益率 | `d['op_margin']` | Operating Margin |
| `GrossMargin` | 毛利率 | `d['gross_margin']` | Gross Margin |
| `OpCF_Assets` | 營運現金/資產 | `d['op_cashflow'] / d['total_assets']` | Operating CF / Total Assets |
| `DebtRatio_inv` | 負債比反轉 | `-d['debt_ratio']` | Lower debt = higher score |
| `CurrentRatio` | 流動比率 | `d['current_ratio']` | Current Ratio |
| `AssetTurnover` | 資產週轉 | `d['asset_turnover']` | Asset Turnover |

### LowVol (3)

| Name | Display | Compute | Direction |
|------|---------|---------|-----------|
| `EntryVol_inv` | 波動率反轉 | `-get_entry_volatility()` | Lower vol = higher score |
| `RetStd60_inv` | 報酬標準差反轉 | `-adj_close.pct_change().rolling(60).std()` | Lower return std = higher score |
| `Beta_inv` | Beta反轉 | `-rolling_beta(12M)` | Lower beta = higher score |

### Size (1)

| Name | Display | Compute | Direction |
|------|---------|---------|-----------|
| `MktCap_inv` | 市值反轉 | `-d['market_val']` | Small cap preference |

### TW_Specific (5)

| Name | Display | Compute | Direction |
|------|---------|---------|-----------|
| `BossRatio` | 大戶持股 | `get_inventory_holder(start_level=9)` | Higher = more institutional holding |
| `BrokerBSR_Sharpe` | 券商BSR_Sharpe | `broker_bsr.rolling(20).mean() / broker_bsr.rolling(20).std()` | Higher = consistent broker buying |
| `RCI26_inv` | RCI26反轉 | `-calc_rci(adj_close, period=26)` | Lower RCI = mean reversion signal |
| `DirHolding` | 董監持股 | `d['dir_holding']` | Higher director holding |
| `MarginUsage_inv` | 融資使用率反轉 | `-d['margin_usage']` | Lower margin usage = less retail speculation |

## S3v4 Factor Subset (9 factors)

The current production strategy S3v4 uses these 9 factors:

| Factor | Registry Name | S3v4 Weight | Category |
|--------|--------------|-------------|----------|
| boss_ratio | BossRatio | 2.67 | TW_Specific |
| rev_yoy | RevYoY_2M | 2.25 | Momentum |
| gpa | GPA | 1.42 | Quality |
| pb | PB_inv | 1.03 | Value |
| market_rev_ratio | MktRev_inv | 0.72 | Value |
| rsv | RSV_60 | 0.65 | Momentum |
| vol_ratio | VolRatio_5_20 | 0.30 | Momentum |
| rev_mom | RevMom_3M | 0.20 | Momentum |
| broker_bsr_sharpe | BrokerBSR_Sharpe | 0.20 | TW_Specific |

## Category Color Scheme

```python
CATEGORY_COLORS = {
    'Value': '#e74c3c',
    'Momentum': '#3498db',
    'Quality': '#2ecc71',
    'LowVol': '#9b59b6',
    'Size': '#f39c12',
    'TW_Specific': '#1abc9c',
}
```
