
# Dates

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
## Non Market Days
Any non market day sent as a "start" or "end" parameter will automatically convert to the prior valid market day. 

#### Example:
Date | Conversion|Notes 
-----|-----|-----
August 8, 2020| August 7, 2020 |a Saturday converts to the prior Friday
January 1, 2020|December 31, 2019|a Market holiday converts to prior valid market day
February 1, 2030|last day in the database|a date in the future converts to last day of database
March 31, 1970|August 1, 1988|a date before the first date in the FastTrack database converts to the first day of the FastTrack database