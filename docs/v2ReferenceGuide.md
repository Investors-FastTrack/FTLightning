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