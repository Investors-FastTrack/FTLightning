# Data Information
This section describes FastTrack's data collection process, update schedule and data formatting.

## Data Update Schedule

FastTrack updates our database 3 times daily:

#### Publish 1 - 6:40 PM EST

Publish 1 includes updated prices and dividends for all stocks, ETFs, COF, approximately 90% of open end funds, and approx 50% of indexes. 

Liquid Alts, smaller funds, some international funds commonly report delayed NAVs and get updated in Publish 2.

#### Publish 2 - 8:25 PM EST
Publish 2 includes any changes made to Publish 1 and all remaining open-end funds and indexes not reported in Publish 1.

#### Publish 3 - 7:25 AM EST
Publish 3 is released the following morning and includes any updates and dividends processed overnight and early morning.

---

## Missing Prices

**Closing prices** - any missing closing price in the FastTrack dataset will report value of the priod market day


*Example:*

ETF "XYZ" priced at $10.00 on 8/3/2020, `no price on 8/4/2020`, and $11.00 on 8/5/2020.
 
The closing price data will report 10 for 8/3/2020, `10 for 8/4/2020`, and 11 for 8/5/2020. (The 8/3/2020 price of 10 is repeated for 8/4/2020)

**OLHV data** - any missing open, high, low, or volume data will return "-1"

---
## Dividend Adjusted Prices

"price" in this documentation and in FastTrack universally reffers to the "adjusted price" of a security. 

All statistics and total return is calculated on the dividend adjusted data. 

Unadjusted prices are noted as such."

---

## Default Parameter Values
API end points have various parameters and when not specified, default values are used:

* `end` - Any time "end" is not specified, the last day in the FastTrack database is used
* `start` - 1 year minus the last day in the database
* `basis` - SP-DA, the total return S&P 500 index 
* `len` - 21 market days
* `count` - 1000
* `riskfree` = VMMXX, Vanguard money market funds




