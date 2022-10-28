# Introduction

### FastTrack Market Data API
FastTrack Market Data API is Investors FastTrack's data REST API. All requests should be made over SSL. All request and response bodies, including errors, are encoded in JSON.

<!-- theme: danger -->
>### *Technical Documentation*
>This documentation is **intended for computer programmers and developers**. 

<!-- theme: none -->
>FastTrack offers other software products that do not require technical or coding knowledge. 
>Visit http://www.fasttrack.net for information on our non technical software and data products.


### Base URL
>https://ftl.fasttrack.net/v1/

### Free Trial Tickers
Free Trials have access to all indexes in our database, including equity, bond, commodity, etc. 

21 Sector SPRD ETFs and 13 Large Cap Stocks are available to demo. Sign up for a trial account and app id, then request on any GET request. (POST request not included on free symbols)

See the full list of tickers here: https://ftl.fasttrack.net/v3/api/freetickers

### FastTrack Database
FastTrack's database includes all US listed Mutual Fund, ETF, and Stocks (approx 30k securities). We additionally provide over 1200+ global indexes.

Search and chart our database here: https://ftcloud.fasttrack.net/web/search

[Click to Read More about our Data here](./02a-DataInfo.md)

### Available Data
The following items are available for all tickers in the FastTrack database

#### [Pricing](https://fasttrack.stoplight.io/docs/ftlightning/aff23f320c5df-data-range-multiple-tickers)
- Daily Adjusted and Unadjusted Closing Prices
- Daily Open, High, Low Prices
- Daily volume

#### Descriptive
- Name
- Shares Outstanding
- Market Capitalization
- 52 Week High
- 52 Week Low
- Inception Date
- Yield (Income only)
- Yield Total (Income, Short, Long Term)

#### [Security Portfolio Data](https://fasttrack.stoplight.io/docs/ftlightning/264aa1a7657f9-portfolio-details)
- Top 10 positions
- Asset weighting
- Sector weighting
- Region weighting
- Category
- Expense Ratio
- Prospectus Benchmark
- FT Benchmark
- Category Benchmark
- 1 year turnover

#### [Risk](https://fasttrack.stoplight.io/docs/ftlightning/c5068d5e59f7e-multiple-tickers)
- Alpha
- Annualized Return
- Beta
- Correlation
- FastTrack Alpha
- RSI
- Sharpe Ratio
- Standard Deviation
- Total Return
- Treynor
- Ulcer Index
- Ulcer Performance Index

#### Drawdowns
- Max Drawdown
- Max Draw Length
- Max Draw Recovery Length
- Max Draw Peak Date
- Max Draw Valley Date

#### Returns
- ANY Time Period
- 1 and 5 days
- 1, 3, 6, 9 months
- 1, 3, 5, 10 year
- Year to date
- Yearly total return 1988 to the current year

#### [Dividends](https://fasttrack.stoplight.io/docs/ftlightning/9609a82a71987-dividends)
- Short Term
- Long Term
- Income


#### Moving Average
- Any length moving average for any day back to 1988

#### [Portfolio Modeling](https://fasttrack.stoplight.io/docs/ftlightning/c6683def5f895-calculate-static-model)
- Static models
- Momentum models

---

### Free Trial
FastTrack offers a free API trial. No credit card is necessary. 

The trial allows full access to our index database, covering over 1,200 stock, bond, commodities, and other global indexes. 

#### [Click here to Sign for a Trial](https://subscribe.fasttrack.net/landing/api/apilanding.html)
