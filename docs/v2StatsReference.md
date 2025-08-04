---
stoplight-id: xvigwkviktlog
---

# V2 Stats API - Statistical Methodology Reference

Technical reference for statistical calculations and methodologies used in the V2 Stats API. This guide explains what each metric represents, how it's calculated, and the data format returned.

## API Response Structure

Statistical data is organized into these main categories in the response:

```json
{
  "periods": {
    "1y": {
      "returns": { /* return calculations */ },
      "risk": {
        "volatility": { /* volatility measures */ },
        "market_risk": { /* benchmark-relative metrics */ },
        "risk_adjusted_performance": { /* ratio calculations */ }
      },
      "maxdraw": { /* drawdown analysis */ }
    }
  }
}
```

---

## Return Calculations

### Total Return
**API Field**: `returns.total`  
**Data Type**: Number (decimal)  
**Formula**: `(Ending Value / Starting Value) - 1`  
**Example Response**: `0.307` (representing 30.7% return)  
**Range**: Unlimited (can be negative for losses)

### Annualized Market Return  
**API Field**: `returns.annualized_market`  
**Data Type**: Number (decimal)  
**Formula**: `((Ending Value / Starting Value)^(252.25 / Trading Days)) - 1`  
**Trading Days**: 252.25 market days per year (industry standard)  
**Example Response**: `0.2616` (representing 26.16% annualized)

### Annualized Calendar Return
**API Field**: `returns.annualized_calendar`  
**Data Type**: Number (decimal)  
**Formula**: `((Ending Value / Starting Value)^(365.25 / Calendar Days)) - 1`  
**Calendar Days**: 365.25 days per year (accounts for leap years)  
**Example Response**: `0.25931` (representing 25.93% annualized)

---

## Volatility Measures

### Standard Deviation (Monthly)
**API Field**: `risk.volatility.sd_monthly`  
**Data Type**: Number (decimal)  
**Calculation**: `√(Daily Price Change Variance) × √21.25`  
**Basis**: 21.25 average market days per month  
**Example Response**: `0.053` (representing 5.3% monthly volatility)

### Standard Deviation (Annualized)
**API Field**: `risk.volatility.sd_annualized`  
**Data Type**: Number (decimal)  
**Calculation**: `Daily SD × √252.25`  
**Basis**: 252.25 market days per year  
**Example Response**: `0.185` (representing 18.5% annual volatility)

### Downside Deviation
**API Field**: `risk.volatility.downside_deviation`  
**Data Type**: Number (decimal)  
**Calculation**: `√[(1/N) × Σ[max(0, Ri – Threshold)²]]`  
**Parameters**: 
- `Ri` = daily return
- `N` = number of trading days  
- `Threshold` = minimum acceptable return (typically 0)
**Example Response**: `0.042` (representing 4.2% downside deviation)

### Ulcer Index
**API Field**: `risk.volatility.ulcer_index`  
**Data Type**: Number (decimal)  
**Calculation**: `√((Sum of R²) / N)` where R = % below highest previous value  
**Measures**: Depth and duration of drawdowns combined  
**Example Response**: `5.03`

---

## Market Risk Measures

### Beta
**API Field**: `risk.market_risk.beta`  
**Data Type**: Number (decimal)  
**Calculation**: `Covariance(Asset, Benchmark) / Variance(Benchmark)`  
**Requirements**: Calculated using correlation period (default 21 days)  
**Example Response**: `1.08` (8% more volatile than benchmark)  
**Special Values**: Can be negative (inverse relationship)

### Alpha  
**API Field**: `risk.market_risk.alpha`  
**Data Type**: Number (decimal)  
**Calculation**: `Asset Return - (Risk-free Rate + Beta × (Benchmark Return - Risk-free Rate))`  
**Units**: Decimal format (0.021 = 2.1% annual alpha)  
**Dependencies**: Requires valid beta calculation  
**Example Response**: `0.021` (2.1% annual excess return)

### Correlation
**API Field**: `risk.market_risk.correlation`  
**Data Type**: Number (decimal)  
**Range**: -1.0 to +1.0  
**Calculation**: Based on daily price changes over correlation period  
**Formula**: `Pearson correlation coefficient between asset and benchmark returns`  
**Example Response**: `0.89` (strong positive correlation)

### R-Squared
**API Field**: `risk.market_risk.r_squared`  
**Data Type**: Number (decimal)  
**Calculation**: `Correlation²`  
**Range**: 0.0 to 1.0  
**Interpretation**: Percentage of variance explained by benchmark  
**Example Response**: `0.79` (79% of movements explained by benchmark)

---

## Risk-Adjusted Performance

### Sharpe Ratio
**API Field**: `risk.risk_adjusted_performance.sharpe_ratio`  
**Data Type**: Number (decimal)  
**Calculation**: `(Asset Return - Benchmark Return) / (Monthly SD × √12)`  
**Note**: Uses benchmark return instead of risk-free rate  
**Example Response**: `1.25`

### Sortino Ratio  
**API Field**: `risk.risk_adjusted_performance.sortino_ratio`  
**Data Type**: Number (decimal)  
**Calculation**: `(Return - MAR) / Downside Deviation`  
**MAR**: Minimum Acceptable Return (typically benchmark return)  
**Example Response**: `2.76`

### Treynor Ratio
**API Field**: `risk.risk_adjusted_performance.treynor_ratio`  
**Data Type**: Number (decimal)  
**Calculation**: `(Security Return - Risk-Free Return) / Beta`  
**Dependencies**: Requires valid beta calculation  
**Example Response**: `702.96`

### Ulcer Performance Index
**API Field**: `risk.risk_adjusted_performance.ulcer_performance_index`  
**Data Type**: Number (decimal)  
**Calculation**: `(Security Return - Low Risk Base Return) / Ulcer Index`  
**Example Response**: `1.89`

### FastTrack Alpha (FTAlpha)
**API Field**: `risk.risk_adjusted_performance.ftalpha`  
**Data Type**: Number (decimal)  
**Calculation**: Proprietary algorithm combining multiple risk and return factors  
**Characteristics**: Higher values indicate better risk-adjusted performance  
**Example Response**: `2790.49`

---

## Drawdown Analysis

### Maximum Drawdown Value
**API Field**: `maxdraw.value`  
**Data Type**: Number (decimal, negative)  
**Calculation**: `(Peak Value - Trough Value) / Peak Value`  
**Example Response**: `-0.152` (representing -15.2% drawdown)

### Peak Date
**API Field**: `maxdraw.peak_date`  
**Data Type**: String (ISO date format)  
**Description**: Date when maximum drawdown period began  
**Example Response**: `"2024-03-15"`

### Valley Date  
**API Field**: `maxdraw.valley_date`  
**Data Type**: String (ISO date format)  
**Description**: Date of lowest point during maximum drawdown  
**Example Response**: `"2024-06-22"`

### Duration Days
**API Field**: `maxdraw.duration_days`  
**Data Type**: Integer  
**Calculation**: Number of calendar days from peak to valley  
**Example Response**: `99`

### Recovery Days
**API Field**: `maxdraw.recovery_days`  
**Data Type**: Integer  
**Calculation**: Days from valley back to peak value  
**Special Value**: `-1` if not yet recovered  
**Example Response**: `82`

---

## Calculation Parameters

### Default Settings
- **Correlation Period**: 21 days (configurable via `advanced.correlation_period`)
- **Risk-Free Rate**: Derived from benchmark calculations
- **Trading Days Per Year**: 252.25
- **Calendar Days Per Year**: 365.25
- **Market Days Per Month**: 21.25

### Benchmark Configuration
- **Default Benchmark**: SPY (S&P 500)
- **Customizable**: Via `benchmark` request parameter
- **Usage**: Required for all market risk and risk-adjusted performance calculations

### Period Requirements
- **Risk Calculations**: Available for periods ≥ 1 month
- **Drawdown Analysis**: Available for periods ≥ 1 month  
- **Return Calculations**: Available for all periods
- **Short Periods (1d, 5d)**: Only return calculations provided

---

## Error Handling and Special Values

### Error Codes in Responses
- **-99**: Standard calculation error or insufficient data
- **-9999**: System calculation error
- **-1**: Missing or invalid data point
- **null**: Metric not calculated for this period/asset type

### Data Validation
- **Correlation Requirement**: Beta calculations require correlation ≥ 0.90 for validity
- **Minimum Data Points**: Risk calculations need sufficient data points for statistical validity
- **Asset Type Limitations**: Some metrics only available for certain asset types

### Response Precision  
All numerical values are rounded to 5 decimal places using the `RoundValue()` function, except:
- Error values (-99, -9999, -1) pass through unchanged
- Null values indicate the metric was not calculated

---

## Asset Type Behavior

### Individual Securities
- **All metrics available** (subject to period requirements)
- **Data Sources**: Market data, dividend data, volume data
- **Benchmark Comparison**: Full market risk analysis

### Portfolio Models  
- **Return Calculations**: Based on calculated portfolio values
- **Risk Metrics**: Calculated from portfolio price series
- **Limitations**: No volume data, dividend data comes from underlying holdings

### Custom Data (FNU)
- **Return Calculations**: Based on provided price series  
- **Risk Metrics**: Available if sufficient data points
- **Limitations**: No fundamental data (dividends, volume, etc.)

---

## Technical Implementation Notes

### Calculation Engine
- Uses FastTrack calculation engine (`ftshell`) for all statistical computations
- Maintains compatibility with existing FastTrack methodologies
- All calculations use market-day adjusted periods (excludes weekends/holidays)

### Data Precision
- Calculations performed in single-precision floating point
- Final values rounded to 5 decimal places for consistency
- Error values preserved exactly as returned by calculation engine

### Performance Considerations
- Risk calculations require additional processing time
- Complex portfolio models require more computational resources  
- Caching used for frequently requested calculations