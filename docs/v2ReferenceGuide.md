---
stoplight-id: q1qm3qbjxdfhx
---
# Data API Reference Guide

## Asset Type Compatibility {#asset-type-compatibility}

Different data types are available depending on the asset type you're requesting data for:

| Data Type | Individual Stocks | ETFs | Mutual Funds | Portfolio Models | Custom Data |
|-----------|-------------------|------|--------------|------------------|-------------|
| `prices`    | ✅ Available | ✅ Available | ✅ Available | ✅ Available | ✅ Available |
| `volumes`   | ✅ Available | ❌ Not Available | ❌ Not Available | ❌ Not Available | ❌ Not Available |
| `ohlv`      | ✅ Available | ❌ Not Available | ❌ Not Available | ❌ Not Available | ❌ Not Available |
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

## Data Frequency {#data-frequency}

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

## Null Value Handling {#null-values}

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

## Dividend Types {#dividend-types}

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

## Price Adjustments {#price-adjustments}

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