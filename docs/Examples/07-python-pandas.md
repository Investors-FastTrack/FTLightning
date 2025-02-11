---
stoplight-id: bz12tzm4twvhj
---

# Historical Market Data Retrieval with Python and Pandas

## Overview

This guide walks you through retrieving and processing historical market data using Python and pandas. You'll learn how to fetch adjusted closing prices for multiple securities, process the data into a clean pandas DataFrame, and handle the data effectively for analysis.

The solution combines the power of FastTrack's Market Data API with pandas' data manipulation capabilities to create a robust data pipeline. This integration allows you to:

- Fetch historical market dates and trading data efficiently
- Process multiple securities in a single request
- Transform raw API responses into analysis-ready DataFrames
- Handle missing data and errors gracefully
- Scale your data retrieval for large datasets

Whether you're performing financial analysis, building trading models, or conducting market research, this implementation provides a solid foundation for your data needs.

## Getting Started

Before diving into the code, you'll need:

1. FastTrack API credentials (APP_ID and TOKEN)
2. Python with pandas and requests libraries installed
3. Basic understanding of DataFrame operations

Let's break down the implementation into manageable components and explore each part in detail.

## Authentication

The FastTrack API uses header-based authentication. You'll need to include your credentials in every request:

```python
APP_ID = "your_app_id"
TOKEN = "your_token"

headers = {
    'appid': APP_ID,
    'token': TOKEN,
    'Content-Type': "application/json",
    'Accept': "application/json"
}
```

## Core Functionality

The implementation consists of three main components:

1. **Market Dates Retrieval**: Fetches valid trading dates
2. **Price Data Download**: Retrieves adjusted closing prices for specified securities
3. **Data Processing**: Transforms API responses into a clean DataFrame

### Market Dates Retrieval

First, we fetch a list of valid market dates. This ensures our price data aligns with actual trading days:

```python
def download_dates():
    response = requests.get(f"{BASE_URL}/dates", headers=headers)
    response.raise_for_status()
    dates_data = response.json()
    return dates_data.get("dtes", [])
```

### Price Data Download

Next, we retrieve adjusted closing prices for multiple securities in a single request:

```python
def download_data(tickers):
    payload = json.dumps(tickers)    
    response = requests.post(
        f"{BASE_URL}/xmulti/divadjprices", 
        headers=headers, 
        data=payload
    )
    response.raise_for_status()
    return response.json() or {}
```

### Data Processing with Pandas

The heart of our implementation is the data processing function that transforms raw API responses into a clean DataFrame:

```python
def merge_data_with_dates(dates, data):
    records = []
    
    if not data or "datalist" not in data:
        print("Warning: No valid data received.")
        return pd.DataFrame()
    
    for item in data.get("datalist", []):
        ticker = item.get("ticker", "Unknown")
        prices = item.get("prices", [])
        
        if not prices:
            print(f"Warning: No price data for {ticker}.")
            continue
        
        market_days = dates[1:len(prices) + 1] if len(dates) > 1 else []
        if not market_days:
            print("Warning: No valid market days available.")
            continue
        
        for md, price in zip(market_days, prices):
            records.append({
                "Ticker": ticker,
                "Date": md["strdate"],
                "Price": price
            })
    
    return pd.DataFrame(records)
```

## Putting It All Together

Here's how to use these components to create a complete data retrieval pipeline:

```python
import requests
import json
import pandas as pd

# Set up your credentials and headers
APP_ID = "your_app_id"
TOKEN = "your_token"
headers = {
    'appid': APP_ID,
    'token': TOKEN,
    'Content-Type': "application/json",
    'Accept': "application/json"
}

# Download market dates
dates = download_dates()

# Specify the securities you want to analyze
tickers = ["AAPL", "MSFT"]
data = download_data(tickers)

# Create a clean DataFrame
df = merge_data_with_dates(dates, data)

# Your DataFrame is now ready for analysis
print(df.head(252))
```

## Working with the DataFrame

The resulting DataFrame provides a clean, analysis-ready structure:

- Each row represents a single observation (price on a specific date)
- Multiple securities are easily comparable
- Data is properly aligned with market dates
- Missing values are handled gracefully

You can now perform common pandas operations like:

```python
# Get the latest prices
latest_prices = df.groupby('Ticker').last()

# Calculate daily returns
df['Daily_Return'] = df.groupby('Ticker')['Price'].pct_change()

# Resample to monthly data
monthly_prices = df.set_index('Date').groupby('Ticker')['Price'].resample('M').last()
```

## Error Handling and Best Practices

The implementation includes robust error handling:

1. HTTP error detection with `raise_for_status()`
2. Graceful handling of missing or invalid data
3. Warning messages for data quality issues
4. Default empty returns to prevent crashes

For production use, consider:

1. Implementing retry logic for failed requests
2. Adding rate limiting for large data pulls
3. Caching frequently accessed data
4. Logging errors and warnings systematically
5. Validating data integrity after retrieval

## Dependencies

- requests: For API communication
- pandas: For data processing and analysis
- json: For request payload formatting (built-in)
- datetime: For date handling (if needed)

## Next Steps

With this foundation, you can:

1. Expand the data retrieval to include more securities
2. Add custom calculations and indicators
3. Implement data validation checks
4. Create automated data update pipelines
5. Build visualization and reporting tools

Remember to handle your API credentials securely and monitor your API usage to stay within rate limits.