
# Dates

Dates are important when calculating market stats! Read on to ensure you're getting the calculations you expect.

## Date Parameters
Many endpoints in the FastTrack data API take **start=** and/or **end=** query strings as parameters. These parameters determine the date range used to calculate various stats and time series.

FastTrack's API accepts a variety of date formats:

Format |
-|-
yyyy-MM-dd|
yyyyMMdd| 
MM/dd/yyyy|
yyyy-MM (converts to yyyy-MM-01)|

You may also send the desired [marketdate](./Examples/02-date_javascript.md#market-days) as an integer. 

---
## Non Market Days
Any non market day (Saturday, Sunday or Market Holliday) sent as a **start=** or **end=** parameter will automatically convert to the prior valid market day. 

#### Example:
Date | Conversion|Notes 
-----|-----|-----
August 8, 2020| August 7, 2020 |a Saturday converts to the prior Friday
January 1, 2020|December 31, 2019|a Market holiday converts to prior valid market day
February 1, 2030|last day in the database|a date in the future converts to last day of database
March 31, 1970|August 1, 1988|a date before the first date in the FastTrack database converts to the first day of the FastTrack database

---
## Date Defaults
#### end=
If no end date is provided, the default is the last market day in the database
#### start=
If not start date is provided, the default is 12 months prior to the end date. 

#### Examples
End|Start|Result Date Range
-|-|-
2021-03-31| 2021-03-31| 2020-03-31 to 2021-03-31 
2021-03-31| null| 2020-03-31 to 2021-03-31 
null| 2020-03-31| 2020-03-31 to \[Last Market Day\]
null|null| \[Last market Day\]  to \[last market day - 12 months\]

---
## Date Rules
* Start cannot be after end
* Dates greater than \[last market day\] convert to \[last market day\]
* Start cannot be before 9/1/88, the first day of the FastTrack database

---
## General FastTrack Date Formats

FastTrack has three date types: 

#### strdate
This is a traditional caledar date formatted date. yyyy-MM-dd

#### marketdate
This is an integer representing the number of market days since 9/1/1988, the first day in the FastTrack database.

#### juliandate
This is a Microsoft date standard used in Excel. This value coresponds to the excel function =DATEVALUE(\[date value\])


---
## How to use MarketDates

[Jump to examples and description of market days with "bulk data"](./Examples/02-date_javascript.md#market-days)


---
## [Jump to Date Defaults](./02a-DataInfo.md)