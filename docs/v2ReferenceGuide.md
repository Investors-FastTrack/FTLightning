---
stoplight-id: q1qm3qbjxdfhx
---
# Data API Reference Guide

## Asset Type Compatibility 

Different data types are available depending on the asset type you're requesting data for:

| Data Type | Individual Stocks | ETFs | Mutual Funds | Portfolio Models | Custom Data |
|-----------|-------------------|------|--------------|------------------|-------------|
| `prices`    | ✅ Available | ✅ Available | ✅ Available | ✅ Available | ✅ Available |
| `volumes`   | ✅ Available | ✅ Available | ❌ Not Available | ❌ Not Available | ❌ Not Available |
| `ohlv`      | ✅ Available | ✅ Available | ❌ Not Available | ❌ Not Available | ❌ Not Available |
| `dividends` | ✅ Available | ✅ Available | ✅ Available | ❌ Not Available | ❌ Not Available |
| `yield`     | ✅ Available | ✅ Available | ✅ Available | ❌ Not Available | ❌ Not Available |
| `info`      | ✅ Available | ✅ Available | ✅ Available | ✅ Available | ✅ Available |

### Notes:
- **Individual Stocks**: All data types available
- **ETFs/Mutual Funds**: No intraday OHLV or volume data, but prices, dividends, yield, and info are available
- **Portfolio Models** (Static/Dynamic/Momentum): Only calculated prices and basic info available - no underlying security data like dividends or yields
- **Custom Data**: Only user-provided prices and basic metadata available

### Asset Type Detection:
- Individual tickers (e.g., "AAPL", "MSFT") are automatically classified
- Portfolio models are specified with `type` property ("static", "dynamic", "momentum")
- Custom data uses `type: "fnu"`

If you request incompatible data types, those fields will be `null` or omitted from the response without generating an error.

## Data Frequency 

The `frequency` parameter controls which trading days are included in your data response:

### Frequency Types

| Frequency | Description | Performance |
|-----------|-------------|-------------|
| `daily` | All trading days in the date range | Standard processing |
| `weekly` | Friday closes only (or last trading day of week if Friday is holiday) | Cached - faster |
| `monthly` | Last trading day of each month only | Cached - faster |
| `quarterly` | Last trading day of each quarter (March, June, September, December) | Cached - faster |

### Examples

For a request from `2024-01-01` to `2024-03-31`:

**Daily frequency** returns ~65 trading days:
```
["2024-01-02", "2024-01-03", "2024-01-04", ..., "2024-03-28", "2024-03-29"]
```

**Weekly frequency** returns ~13 Fridays:
```
["2024-01-05", "2024-01-12", "2024-01-19", "2024-01-26", "2024-02-02", "2024-02-09", ...]
```

**Monthly frequency** returns 3 month-ends:
```
["2024-01-31", "2024-02-29", "2024-03-28"]
```

**Quarterly frequency** returns 1 quarter-end:
```
["2024-03-28"]
```

### Important Notes

- **Time Series Alignment**: All data arrays (prices, volumes, etc.) will have the same length as the `metadata.dates` array
- **Holiday Handling**: If the target day (Friday, month-end, quarter-end) falls on a market holiday, the last trading day before that date is used
- **Performance**: Weekly, monthly, and quarterly frequencies use pre-calculated caches for faster response times
- **Date Range**: The frequency setting affects which dates are included, but the overall date range is still controlled by `start_date` and `end_date`

## Null Value Handling 

Time series arrays may contain `null` values in specific situations:

### When Nulls Occur

| Scenario | Behavior | Example |
|----------|----------|---------|
| **Before Asset Inception** | `null` for dates before the security started trading | Stock IPO'd in 2020, requesting data from 2019 |
| **Market Holidays** | No nulls - holiday dates are excluded from results | July 4th simply won't appear in dates array |
| **Weekends** | No nulls - weekend dates are excluded from results | Saturdays/Sundays won't appear in dates array |
| **Data Unavailable** | `null` for that specific data point | Volume data missing for a particular day |
| **Incompatible Asset Type** | Entire array is `null` or omitted | Requesting OHLV data for a mutual fund |

### Array Consistency

- **All time series arrays** (prices, volumes, OHLV) will have the same length as `metadata.dates`
- **Null positions align** across all arrays for the same date
- **Array indices correspond** to the same date position across all data types

### Example Response with Nulls

```json
{
  "metadata": {
    "dates": ["2024-01-02", "2024-01-03", "2024-01-04"]
  },
  "results": [{
    "prices": {
      "adjusted": [null, 150.25, 152.10]
    },
    "volumes": [null, 45123000, 52341000]
  }]
}
```

In this example, the first date has no data available, so all arrays have `null` in position 0.

## Dividend Types

Dividend data includes both a human-readable type and a single-character code for each dividend payment:

### Dividend Type Mapping

| Type Code | Type Name | Description | Tax Treatment |
|-----------|-----------|-------------|---------------|
| `D` | Income | Regular dividend distributions from company earnings | Qualified dividends (lower tax rates for most investors) |
| `G` | Short Term | Short-term capital gains distributions | Taxed as ordinary income |
| `C` | Long Term | Long-term capital gains distributions | Taxed at capital gains rates |
| `Z` | Split | Stock splits (not actual cash payments) | No immediate tax implications |
| (Other) | Other | Any dividend type not covered above | Varies by specific type |

### Example Dividend Data

```json
{
  "dividends": [
    {
      "date": "2024-02-15",
      "amount": 0.24,
      "type": "Income",
      "type_code": "D"
    },
    {
      "date": "2024-01-15", 
      "amount": 0.15,
      "type": "Long Term",
      "type_code": "C"
    },
    {
      "date": "2023-08-01",
      "amount": 2.0,
      "type": "Split", 
      "type_code": "Z"
    }
  ]
}
```

### Important Notes

- **Ex-dividend dates**: The `date` field represents the ex-dividend date (when the stock begins trading without the dividend)
- **Stock splits**: When `type_code` is "Z", the `amount` represents the split ratio (e.g., 2.0 = 2-for-1 split)
- **Tax implications**: Dividend types affect tax treatment - consult tax professionals for specific guidance
- **Historical data**: Dividend history is available for individual stocks and ETFs, but not for portfolio models
- **Currency**: All dividend amounts are in USD regardless of the underlying security's home currency

## Price Adjustments 

The API provides three types of price data, each with different adjustment methodologies:

### Price Type Comparison

| Price Type | Stock Splits | Dividends | Use Cases |
|------------|--------------|-----------|-----------|
| **Adjusted** | ✅ Adjusted | ✅ Adjusted | Performance analysis, returns calculation, backtesting |
| **Semi-Adjusted** | ✅ Adjusted | ❌ Not Adjusted | Technical analysis, chart patterns, volume analysis |
| **Unadjusted** | ❌ Not Adjusted | ❌ Not Adjusted | Historical trading prices, order book analysis |

### When to Use Each Type

#### Adjusted Prices (Default)
- **Best for**: Performance calculations, portfolio analysis, returns comparison
- **Why**: Provides continuous price series that reflects true economic value changes
- **Example**: A stock trading at $100 pays a $2 dividend. Adjusted price drops to $98 on ex-dividend date to reflect the economic transfer

#### Semi-Adjusted Prices
- **Best for**: Technical analysis, chart patterns, moving averages
- **Why**: Maintains dividend gaps that technical analysts expect to see
- **Example**: Same $100 stock shows price drop from $100 to $98 on ex-dividend date, preserving the gap for technical analysis

#### Unadjusted Prices
- **Best for**: Historical research, order execution analysis, actual trading prices
- **Why**: Shows exactly what prices were quoted and traded at each date
- **Example**: Shows the actual $100 → $98 price movement without any adjustments

### Stock Split Example

For a 2-for-1 stock split on a stock trading at $200:

```json
{
  "dates": ["2024-01-30", "2024-01-31", "2024-02-01"],
  "prices": {
    "adjusted": [100.00, 100.00, 101.00],
    "semi_adjusted": [100.00, 100.00, 101.00], 
    "unadjusted": [200.00, 200.00, 101.00]
  }
}
```

- **Adjusted & Semi-Adjusted**: Both show split-adjusted prices for continuity
- **Unadjusted**: Shows actual trading prices ($200 before split, $101 after)

### Performance Notes

- **Default behavior**: Only adjusted prices are returned unless specifically requested
- **Additional cost**: Requesting unadjusted/semi-adjusted prices requires additional data retrieval
- **Asset limitations**: Only available for individual securities (not portfolio models or custom data)

## Supported Asset Types 

The Data API supports the same asset types as the Stats API, allowing you to extract raw data from complex portfolio models:

### Simple Tickers

The most common asset type - individual securities:

```json
{
  "assets": ["AAPL", "MSFT", "VTI"]
}
```

**Available data**: All data types (prices, volumes, OHLV, dividends, yield, info)

### Static Portfolio Models

Fixed allocation portfolios that rebalance to target weights:

```json
{
  "assets": [{
    "type": "static",
    "name": "Conservative 60/40",
    "ticker": "CONS_6040",
    "holdings": [
      {"ticker": "VTI", "weight": 60},
      {"ticker": "BND", "weight": 40}
    ],
    "rebalance": "quarter",
    "start": "2020-01-01",
    "seed": 100000
  }]
}
```

**Available data**: prices, info (no volumes, OHLV, dividends, or yield)

### Dynamic Portfolio Models  

Portfolios that change allocation over time:

```json
{
  "assets": [{
    "type": "dynamic",
    "name": "Risk On/Off Strategy",
    "ticker": "RISK_TACTICAL",
    "seed": 75000,
    "models": [
      {
        "start": "2020-01-01",
        "rebalance": "month",
        "holdings": [
          {"ticker": "QQQ", "weight": 80},
          {"ticker": "TLT", "weight": 20}
        ]
      },
      {
        "start": "2022-01-01", 
        "rebalance": "quarter",
        "holdings": [
          {"ticker": "VTI", "weight": 60},
          {"ticker": "BND", "weight": 40}
        ]
      }
    ]
  }]
}
```

**Available data**: prices, info (no volumes, OHLV, dividends, or yield)

### Momentum Models

Strategies that rank securities and hold top performers:

```json
{
  "assets": [{
    "type": "momentum",
    "name": "Top 3 Tech Momentum",
    "ticker": "TECH_MOM_3",
    "seed": 50000,
    "start": "2021-01-01",
    "rebalance": "month",
    "model": {
      "universe": ["AAPL", "MSFT", "GOOGL", "AMZN", "TSLA"],
      "number_holdings": "3",
      "ranking_system": {
        "type": "return",
        "parameters": {
          "lookback_period": "quarter"
        }
      }
    }
  }]
}
```

**Available data**: prices, info (no volumes, OHLV, dividends, or yield)

### Custom Data

User-provided price series:

```json
{
  "assets": [{
    "type": "fnu",
    "name": "Custom Strategy",
    "data": {
      "ticker": "CUSTOM_001",
      "prices": [100, 102, 98, 105, 110, 108, 115]
    }
  }]
}
```

**Available data**: prices, info (no other data types)

### Data Extraction Considerations

#### Portfolio Model Pricing
- Portfolio models return **calculated portfolio values** over time
- Prices reflect the combined performance of underlying holdings
- Rebalancing frequency affects how often allocations are adjusted
- Start date determines when the portfolio begins tracking

#### Mixed Asset Requests
You can mix different asset types in a single request:

```json
{
  "assets": [
    "AAPL",
    {
      "type": "static",
      "name": "60/40 Portfolio", 
      "holdings": [
        {"ticker": "VTI", "weight": 60},
        {"ticker": "BND", "weight": 40}
      ]
    }
  ]
}
```

#### Performance Notes
- **Simple tickers**: Fastest data retrieval
- **Portfolio models**: Require calculation, slower response times
- **Large universes**: Momentum models with many securities take longer to process

## Performance Considerations 

Understanding API performance characteristics helps you optimize request patterns and set appropriate expectations for response times.

### Response Time Expectations

| Request Type | Typical Response Time | Factors Affecting Speed |
|--------------|----------------------|------------------------|
| **Single ticker, daily data** | 50-200ms | Data range length, include options |
| **Multiple tickers (5-10)** | 200-800ms | Number of securities, data types requested |
| **Static portfolio model** | 300-1000ms | Portfolio complexity, date range, rebalancing frequency |
| **Dynamic portfolio model** | 500-2000ms | Number of model changes, underlying securities |
| **Momentum model** | 1000-5000ms | Universe size, ranking calculations, rebalancing frequency |

### Performance Optimization Tips

#### Choose Appropriate Frequency
```json
// Faster - cached dates
{"frequency": "monthly"}  // Pre-calculated month-ends

// Slower - full processing  
{"frequency": "daily"}    // All trading days calculated
```

**Monthly/Quarterly/Weekly frequencies use cached date calculations for 3-5x faster response times.**

#### Limit Data Types
```json
// Faster - minimal data
{"include": ["prices"]}

// Slower - comprehensive data
{"include": ["prices", "volumes", "ohlv", "dividends", "yield", "info"]}
```

**Each additional data type requires separate database queries and processing.**

#### Optimize Date Ranges
```json
// Faster - shorter periods
{"start_date": "2024-01-01", "end_date": "2024-03-31"}

// Slower - longer periods  
{"start_date": "2020-01-01", "end_date": "2024-07-29"}
```

**Shorter date ranges reduce data processing and transfer time.**

#### Batch Related Requests
```json
// Efficient - single request
{"assets": ["AAPL", "MSFT", "GOOGL"]}

// Less efficient - multiple requests
// Three separate API calls for individual tickers
```

**Batch requests share processing overhead and reduce total response time.**

### Portfolio Model Performance

#### Static Models
- **Fast**: Simple allocations with few holdings
- **Moderate**: Monthly/quarterly rebalancing
- **Slower**: Daily rebalancing with many holdings

#### Dynamic Models  
- **Performance scales with**: Number of model transitions, underlying securities
- **Optimization**: Fewer model changes = faster processing

#### Momentum Models
- **Most resource-intensive**: Require ranking calculations at each rebalance
- **Major factors**: Universe size, rebalancing frequency, ranking complexity
- **Tip**: Use quarterly rebalancing vs. daily for 10x speed improvement

### Caching and Data Freshness

#### Frequency Caches
- **Weekly/Monthly/Quarterly**: Pre-calculated date arrays cached for years 2000-present
- **Cache refresh**: Updated daily with new market data
- **Performance boost**: 3-5x faster than daily frequency requests

#### Market Data Updates
- **Intraday**: Data updated throughout trading day
- **End-of-day**: Final data available ~1 hour after market close
- **Historical**: Data older than 1 year is heavily cached

### Request Size Limits

#### Practical Limits
- **Assets per request**: 50+ assets supported, but response time increases linearly
- **Date range**: 10+ years supported, but consider pagination for very large datasets
- **Data volume**: Large requests may timeout - consider breaking into smaller chunks

#### Best Practices
- **Large portfolios**: Break into multiple requests by asset type or date range
- **Historical analysis**: Use appropriate frequency (monthly vs daily) for the analysis timeframe
- **Real-time needs**: Use minimal include options for fastest response

### Error Handling and Timeouts

#### Timeout Behavior
- **Typical timeout**: 30 seconds for complex portfolio calculations
- **Fallback strategy**: Retry with smaller date ranges or fewer assets
- **Partial success**: API returns available data with error details for failed assets

#### Optimization for Reliability
- **Conservative requests**: Start with smaller requests and scale up
- **Error monitoring**: Check metadata.calculation_time_ms to track performance trends
- **Retry logic**: Implement exponential backoff for failed requests

## Period Calculation Behavior 

Different statistical calculations are available depending on the time period length. This ensures statistical validity and prevents misleading metrics for very short time frames.

### Calculation Matrix by Period Length

| Period | Returns | Risk Metrics | Drawdown Analysis | Notes |
|--------|---------|--------------|-------------------|--------|
| **1d, 5d** | ✅ Available | ❌ Not Available | ❌ Not Available | Too short for meaningful risk statistics |
| **1m+** | ✅ Available | ✅ Available | ✅ Available | Full statistical analysis |
| **Custom Periods** | ✅ Available | ✅ Available* | ✅ Available* | *Depends on actual date range length |
| **Historical Periods** | ✅ Available | ✅ Available | ✅ Available | Full analysis for all rollup periods |

### What's Always Calculated

**Returns Data** (available for all periods):
- `total`: Cumulative return over the period
- `annualized_market`: Return annualized using market days (252.25/year)
- `annualized_calendar`: Return annualized using calendar days (365.25/year)

### What Requires Longer Periods

**Risk Metrics** (excluded for 1d, 5d periods):
- **Volatility**: Standard deviation, downside deviation, ulcer index
- **Market Risk**: Beta, alpha, correlation, R-squared vs benchmark
- **Risk-Adjusted Performance**: Sharpe ratio, Sortino ratio, Treynor ratio, etc.

**Drawdown Analysis** (excluded for 1d, 5d periods):
- Maximum drawdown value and dates
- Peak-to-valley analysis
- Recovery time calculations

### Why Short Periods Are Limited

**Statistical Validity**: 
- Risk metrics require sufficient data points for meaningful calculation
- Single-day or 5-day periods don't provide enough variation to calculate reliable volatility or correlation

**Benchmark Comparison**:
- Risk-adjusted metrics (Sharpe, alpha, beta) need adequate time to show relationship with benchmark
- Short periods can show extreme or misleading ratios

**Drawdown Analysis**:
- Meaningful drawdown analysis requires time for market cycles
- Very short periods may not capture significant price movements

### Custom Period Behavior

Custom periods are evaluated based on their actual date range:

```json
{
  "time_periods": [
    {
      "custom": {
        "label": "short_test",
        "from": "2024-07-25",  
        "to": "2024-07-29"     // 4 days - gets limited calculations
      }
    },
    {
      "custom": {
        "label": "covid_crash",
        "from": "2020-02-01",
        "to": "2020-04-01"     // 2 months - gets full calculations
      }
    }
  ]
}
```

### Response Structure Differences

**Limited Calculations** (1d, 5d periods):
```json
{
  "periods": {
    "1d": {
      "returns": { /* returns data only */ }
      // No risk or maxdraw objects
    }
  }
}
```

**Full Calculations** (1m+ periods):
```json
{
  "periods": {
    "1y": {
      "returns": { /* returns data */ },
      "risk": { /* complete risk analysis */ },
      "maxdraw": { /* drawdown analysis */ }
    }
  }
}
```

### Recommendations

- **Use 1m or longer periods** for comprehensive statistical analysis
- **Short periods (1d, 5d)** are useful for recent performance checking only
- **Custom periods should span at least 30 days** for meaningful risk metrics
- **Historical analysis** automatically uses appropriate period lengths

## Response Structure Differences 

The Stats API has two endpoints that return fundamentally different response structures. Understanding these differences is crucial for proper integration.

### Snapshot Mode (`/stats/stats2`)

**Data Organization**: Statistics organized by **lookback periods** from a single as-of date
**Use Case**: Current analysis with various lookback periods

#### Response Structure
```json
{
  "results": [{
    "ticker": "AAPL",
    "name": "Apple Inc",
    "periods": {
      "1m": { /* 1-month lookback stats */ },
      "3m": { /* 3-month lookback stats */ },
      "1y": { /* 1-year lookback stats */ }
    },
    "shared_data": {
      "pricing": { /* current pricing info */ },
      "yield": { /* current yield data */ }
    }
  }]
}
```

#### Key Characteristics
- **`periods` object**: Contains lookback periods (`1m`, `3m`, `1y`, etc.)
- **`shared_data`**: Current pricing and yield data shared across all periods
- **Single point-in-time**: All calculations are as of the specified `as_of` date
- **Lookback logic**: Each period looks back from the as-of date

### Historical Mode (`/stats/stats2/historical`)

**Data Organization**: Statistics organized by **calendar periods** across multiple time ranges
**Use Case**: Time-series analysis and historical trend analysis

#### Response Structure
```json
{
  "results": [{
    "ticker": "AAPL", 
    "name": "Apple Inc",
    "yearly": {
      "2022": { 
        "returns": { /* 2022 full-year stats */ },
        "pricing": { /* end-of-2022 pricing */ },
        "yield": { /* end-of-2022 yield */ }
      },
      "2023": { /* 2023 full-year stats */ }
    },
    "quarterly": {
      "2023-Q1": { /* Q1 2023 stats */ },
      "2023-Q2": { /* Q2 2023 stats */ }
    },
    "monthly": {
      "2023-01": { /* January 2023 stats */ },
      "2023-02": { /* February 2023 stats */ }
    }
  }]
}
```

#### Key Characteristics
- **`yearly/quarterly/monthly` objects**: Contains discrete calendar periods
- **Period-specific data**: Each period includes its own pricing and yield data
- **Multiple time points**: Each period represents a complete discrete time range
- **No shared data**: All data is specific to each calendar period

### Comparison Examples

#### Same Asset, Different Structures

**Snapshot Request**:
```json
{
  "assets": ["AAPL"],
  "time_periods": ["1y", "3y"],
  "as_of": "2024-07-29"
}
```

**Snapshot Response Structure**:
```json
{
  "results": [{
    "periods": {
      "1y": { /* July 2023 → July 2024 */ },
      "3y": { /* July 2021 → July 2024 */ }
    },
    "shared_data": {
      "pricing": { /* July 29, 2024 pricing */ }
    }
  }]
}
```

**Historical Request**:
```json
{
  "assets": ["AAPL"],
  "rollups": ["year"],
  "years": ["2022", "2023", "2024"]
}
```

**Historical Response Structure**:
```json
{
  "results": [{
    "yearly": {
      "2022": { 
        "returns": { /* Jan 1 → Dec 31, 2022 */ },
        "pricing": { /* Dec 31, 2022 pricing */ }
      },
      "2023": { 
        "returns": { /* Jan 1 → Dec 31, 2023 */ },
        "pricing": { /* Dec 31, 2023 pricing */ }
      },
      "2024": { 
        "returns": { /* Jan 1 → Current date */ },
        "pricing": { /* Current pricing */ }
      }
    }
  }]
}
```

### When to Use Each Mode

#### Snapshot Mode Best For:
- **Current performance analysis**: "How has this stock performed over various recent periods?"
- **Comparative analysis**: Comparing 1-month vs 1-year performance as of today
- **Risk assessment**: Current risk metrics over different lookback periods
- **Portfolio review**: Regular portfolio performance checking

#### Historical Mode Best For:
- **Trend analysis**: "How did performance change year over year?"
- **Seasonal analysis**: Quarterly or monthly performance patterns
- **Historical comparison**: Comparing specific calendar periods
- **Time-series charting**: Building historical performance charts
- **Backtesting**: Analyzing how strategies would have performed in specific periods

### Data Availability Differences

| Feature | Snapshot Mode | Historical Mode |
|---------|---------------|-----------------|
| **Pricing Data** | Current (in shared_data) | Period-specific (in each period) |
| **Yield Data** | Current (in shared_data) | Period-specific (in each period) |
| **Risk Metrics** | Lookback-based calculations | Period-specific calculations |
| **Custom Periods** | ✅ Supported | ❌ Not supported |
| **Multiple Rollups** | ❌ Single as-of date | ✅ Multiple rollup types |

### Integration Tips

#### For Snapshot Mode:
- Access period data via `result.periods["1y"]`
- Use `shared_data` for current pricing information
- All periods share the same end date (as_of)

#### For Historical Mode:
- Access data via `result.yearly["2023"]`, `result.quarterly["2023-Q1"]`, etc.
- Each period contains its own pricing/yield data
- Periods represent complete calendar ranges

#### Error Handling:
Both modes use the same error structure in `result.error` when issues occur.

## Include Options Behavior

The `include` parameter controls which optional data is calculated and returned. Each option has specific requirements and behaviors depending on asset type and request settings.

### Include Options Matrix

| Option | Individual Stocks | ETFs/Mutual Funds | Portfolio Models | Period Requirements | Notes |
|--------|-------------------|-------------------|------------------|---------------------|--------|
| `drawdown_analysis` | ✅ Available | ✅ Available | ✅ Available | 1m+ periods only | Excluded from 1d, 5d periods |
| `moving_averages` | ✅ Available | ✅ Available | ✅ Available | Any period | Uses current as-of date |
| `volume_data` | ✅ Available | ❌ Not Available | ❌ Not Available | Any period | Individual stocks only |
| `position_details` | ❌ Not Applicable | ❌ Not Applicable | ✅ Available | Any period | Portfolio models only |
| `security_info` | ✅ Available | ✅ Available | ✅ Available | Any period | Always recommended |

### Detailed Option Behavior

#### `drawdown_analysis`
**What it does**: Calculates maximum drawdown analysis for each time period
**Adds to response**: `maxdraw` object in each period with peak/valley dates and recovery analysis

```json
{
  "periods": {
    "1y": {
      "maxdraw": {
        "value": -0.152,
        "peak_date": "2024-03-15",
        "valley_date": "2024-06-22", 
        "duration_days": 99,
        "recovery_days": 82
      }
    }
  }
}
```

**Requirements**:
- **Period length**: Only calculated for periods 1m and longer
- **Asset types**: All asset types supported
- **Performance impact**: Moderate - requires additional peak/valley calculations

#### `moving_averages`
**What it does**: Calculates moving averages as of the current date
**Adds to response**: `moving_averages` array at result level

```json
{
  "moving_averages": [
    {"period": 50, "value": 172.45},
    {"period": 200, "value": 168.32}
  ]
}
```

**Configuration**:
- **Default periods**: 50-day and 200-day moving averages
- **Custom periods**: Set via `advanced.moving_average_periods`
- **Date basis**: Always calculated as of the `as_of` date
- **Performance impact**: Low - simple price averaging

#### `volume_data`
**What it does**: Calculates current and historical volume statistics
**Adds to response**: `volume_data` object at result level

```json
{
  "volume_data": {
    "current": 45123000,
    "average_30d": 52341000,
    "average_90d": 48567000
  }
}
```

**Restrictions**:
- **Individual stocks only**: ETFs, mutual funds, and portfolio models don't have meaningful volume data
- **Current data**: Reflects volume as of the `as_of` date
- **Performance impact**: Low - simple volume calculations

#### `position_details`
**What it does**: Provides detailed holdings information for portfolio models
**Adds to response**: Position details embedded in portfolio model results

```json
{
  "info": {
    "positions": [
      {
        "ticker": "VTI",
        "weight": 60.0,
        "value": 60000,
        "shares": 245.5
      }
    ]
  }
}
```

**Restrictions**:
- **Portfolio models only**: Only meaningful for static, dynamic, and momentum models
- **Individual securities**: Not applicable to simple ticker requests
- **Performance impact**: None - data generated during model calculation

#### `security_info`
**What it does**: Includes comprehensive security information and current pricing/yield data
**Adds to response**: `info` object and `shared_data` (snapshot mode) or period-specific data (historical mode)

**Snapshot mode**:
```json
{
  "info": {
    "security_type": "Stock",
    "start_date": "1980-12-12",
    "shares_outstanding": 15550061000
  },
  "shared_data": {
    "pricing": {
      "current_close_price": 195.23,
      "market_cap": 3037000000000
    },
    "yield": {
      "yield_income": 0.0055,
      "yield_all": 0.0055
    }
  }
}
```

**Historical mode**:
```json
{
  "yearly": {
    "2023": {
      "pricing": { /* end-of-2023 pricing */ },
      "yield": { /* end-of-2023 yield */ }
    }
  }
}
```

**Always recommended**: Provides essential context for understanding the security

### Default Behavior

**When no includes specified**: Only basic period statistics are calculated (returns, risk, no optional data)

**Automatic inclusions**: 
- Risk calculations are included by default for periods 1m+
- Return calculations are always included
- Error information is always included when applicable

### Performance Considerations

| Option | Performance Impact | Calculation Complexity |
|--------|-------------------|------------------------|
| `security_info` | Low | Simple data lookup |
| `moving_averages` | Low | Price averaging |
| `volume_data` | Low | Volume averaging |
| `drawdown_analysis` | Moderate | Peak/valley analysis |
| `position_details` | None | Generated during model calculation |

### Common Combinations
 
#### **Basic Analysis**
```json
{
  "include": ["security_info"]
}
```
**Use case**: Simple performance review with security context

#### **Comprehensive Analysis**
```json
{
  "include": ["security_info", "drawdown_analysis", "moving_averages"]
}
```
**Use case**: Full investment analysis with risk assessment

#### **Portfolio Analysis**
```json
{
  "include": ["security_info", "drawdown_analysis", "position_details"]
}
```
**Use case**: Portfolio model analysis with holdings details

#### **Stock-Specific Analysis**
```json
{
  "include": ["security_info", "volume_data", "moving_averages", "drawdown_analysis"]
}
```
**Use case**: Individual stock deep dive with volume analysis

### Error Handling

**Incompatible options**: If you request `volume_data` for a mutual fund, the field will be omitted from the response without generating an error

**Missing data**: If underlying data is unavailable (e.g., no volume data for a date), fields may contain `null` or be omitted

**Partial success**: Some include options may succeed while others fail - check each section of the response independently

## Advanced Settings Behavior 

Advanced settings control the underlying calculation parameters for statistical computations. These settings can significantly impact the accuracy and relevance of risk metrics.

### Correlation Period 

**Parameter**: `advanced.correlation_period`  
**Default**: 21 days  
**Range**: 5-252 days  
**Affects**: Beta, Alpha, Correlation, R-Squared calculations

#### What It Controls
The correlation period determines the rolling window of daily returns used to calculate the relationship between an asset and its benchmark.

#### Impact on Calculations
- **Beta**: Uses this period to calculate covariance and variance
- **Alpha**: Depends on beta calculation, so indirectly affected
- **Correlation**: Direct calculation window
- **R-Squared**: Derived from correlation, so directly affected

#### Choosing the Right Period

| Period Length | Use Case | Characteristics |
|---------------|----------|-----------------|
| **5-10 days** | Short-term trading analysis | Very responsive, high volatility in metrics |
| **21 days** (default) | Standard monthly analysis | Balance of responsiveness and stability |
| **63 days** | Quarterly analysis | More stable, slower to adapt to changes |
| **252 days** | Annual analysis | Very stable, represents long-term relationship |

#### Example Impact
```json
{
  "advanced": {"correlation_period": 10}
}
// vs
{
  "advanced": {"correlation_period": 63}
}
```

**10-day period**: Beta might be 1.2 (recent high volatility relationship)  
**63-day period**: Beta might be 0.9 (longer-term stable relationship)

#### When Correlation Period Matters Most
- **Volatile markets**: Shorter periods capture recent regime changes
- **Stable markets**: Longer periods provide more reliable estimates
- **Strategy analysis**: Match period to your rebalancing frequency

### Moving Average Periods

**Parameter**: `advanced.moving_average_periods`  
**Default**: [50, 200]  
**Range**: 1-1000 days per period  
**Affects**: `moving_averages` array in response when included

#### What It Controls
Specifies which moving average periods to calculate when `moving_averages` is included in the request.

#### Common Configurations
```json
{
  "advanced": {
    "moving_average_periods": [20, 50, 200]  // Short, medium, long-term
  }
}
```

#### Standard Period Meanings
- **20 days**: Short-term trend (1 month)
- **50 days**: Medium-term trend (2.5 months)  
- **200 days**: Long-term trend (10 months)
- **252 days**: Full year trend

#### Response Structure
```json
{
  "moving_averages": [
    {"period": 20, "value": 175.23},
    {"period": 50, "value": 172.45}, 
    {"period": 200, "value": 168.32}
  ]
}
```

#### Performance Considerations
- **More periods**: Increases calculation time
- **Longer periods**: Require more historical data
- **Recommended**: Limit to 5 or fewer periods for optimal performance

### Month-End Returns

**Parameter**: `advanced.use_month_end_returns`  
**Default**: false  
**Affects**: Period boundary calculations for all time periods

#### What It Controls
Changes how period start and end dates are calculated, using month-end boundaries instead of exact date arithmetic.

#### Behavior Differences

**Default (false) - Exact Date Math**:
- 1-month period from July 15th looks back to June 15th
- Uses exact trading day counts
- More precise for custom date ranges

**Month-End (true) - Month Boundary Math**:
- 1-month period from July 15th looks back to June 30th
- Aligns with standard financial reporting periods
- Better for comparing with published financial data

#### Example Impact

**Request**: 3-month period as of July 15, 2024

**use_month_end_returns: false**:
- Period: April 15, 2024 → July 15, 2024
- Exact 3-month lookback

**use_month_end_returns: true**:
- Period: April 30, 2024 → July 15, 2024  
- Boundary-aligned lookback

#### When to Use Month-End Returns
- **Financial reporting alignment**: Match standard quarterly/monthly reporting
- **Comparative analysis**: When comparing with published fund data
- **Institutional analysis**: Many institutions use month-end calculations
- **Benchmark comparisons**: When benchmark data uses month-end periods

#### When to Use Exact Date Math (default)
- **Precise period analysis**: When exact timeframes matter
- **Custom date ranges**: For specific event analysis
- **Technical analysis**: When trading day precision is important

### Interaction Effects

Advanced settings can interact with each other and with other request parameters:

#### Correlation Period + Time Periods
Short correlation periods with long time periods can produce unstable metrics:
```json
{
  "time_periods": ["3y"],
  "advanced": {"correlation_period": 5}  // May be too volatile
}
```

**Better approach**: Match correlation period to analysis timeframe
```json
{
  "time_periods": ["3y"], 
  "advanced": {"correlation_period": 63}  // More appropriate
}
```

#### Month-End + Custom Periods
Month-end setting affects custom period calculations:
```json
{
  "time_periods": [
    {"custom": {"label": "crash", "from": "2020-02-15", "to": "2020-04-15"}}
  ],
  "advanced": {"use_month_end_returns": true}
}
```
May adjust the custom period boundaries to align with month-ends.

### Best Practices

#### For Short-Term Analysis (< 6 months)
```json
{
  "advanced": {
    "correlation_period": 21,
    "use_month_end_returns": false
  }
}
```

#### For Long-Term Analysis (> 1 year)  
```json
{
  "advanced": {
    "correlation_period": 63,
    "use_month_end_returns": true
  }
}
```

#### For Technical Analysis
```json
{
  "advanced": {
    "correlation_period": 21,
    "moving_average_periods": [10, 20, 50, 200],
    "use_month_end_returns": false
  }
}
```

#### For Institutional Reporting
```json
{
  "advanced": {
    "correlation_period": 63,
    "moving_average_periods": [50, 200],
    "use_month_end_returns": true
  }
}
```

### Performance Impact 

| Setting | Performance Impact | Calculation Complexity |
|---------|-------------------|------------------------|
| **Short correlation period** | Faster | Less data to process |
| **Long correlation period** | Slower | More data to process |
| **Many MA periods** | Slower | Multiple calculations |
| **Month-end returns** | Minimal | Slight date calculation overhead |

### Validation and Limits 

#### Correlation Period Validation
- **Minimum**: 5 days (statistical minimum for meaningful correlation)  
- **Maximum**: 252 days (one full trading year)
- **Invalid values**: Defaults to 21 days with no error

#### Moving Average Validation  
- **Minimum period**: 1 day
- **Maximum period**: 1000 days
- **Maximum count**: 10 periods (performance limit)
- **Invalid periods**: Skipped with no error

#### Month-End Validation
- **Valid values**: true/false only
- **Invalid values**: Defaults to false