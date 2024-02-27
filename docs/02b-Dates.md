
# Mastering Dates with FastTrack Data API

Understanding dates is crucial for accurate market analysis. FastTrack's Data API offers flexible date handling to ensure your market statistics and time series analyses are precisely aligned with your expectations.

## Tailored Date Parameters
The FastTrack Data API provides **start=** and **end=** query parameters to define the date range for your calculations, accommodating a range of formats for your convenience:
FastTrack's API accepts a variety of date formats:

**Accepted Date Formats:**
- **yyyy-MM-dd**: Standard ISO format for clarity and consistency.
- **yyyyMMdd**: Compact format for easy parsing.
- **MM/dd/yyyy**: U.S. standard format for familiarity.
- **yyyy-MM**: Specifies a month (automatically assumes the first day of the month as the start date).

**Advanced Market Date Handling:**
Convert market dates into integers for direct reference, simplifying date calculations in financial analyses.


## Intelligent Non-Market Day Adjustments
FastTrack seamlessly navigates non-market days, ensuring your date parameters always align with valid trading days:
**Conversion Logic:**
- **Weekends and Holidays**: Inputs falling on non-market days adjust to the nearest preceding market day.
- **Future Dates**: Automatically align with the most recent data available.
- **Pre-Database Dates**: Adjust to the earliest date in FastTrack's repository for historical consistency.


#### Example Conversions:
- **August 8, 2020 (Saturday)** converts to August 7, 2020 (Friday).
- **January 1, 2020 (Market Holiday)** shifts to December 31, 2019.
- **Dates beyond our latest records** default to the last available market day.
- **Pre-database dates** are set to the inception date of FastTrack's dataset.

---
## Simplified Date Defaults
When parameters are unspecified, FastTrack's intelligent defaults ensure you're still equipped with a meaningful date range:

- **End Date**: Defaults to the last available market day.
- **Start Date**: Set to 12 months prior to the specified or defaulted end date, offering a comprehensive yearly overview.

#### Default Examples:
- **Specified End Date Only**: Analyzes the year leading up to the specified end date.
- **Specified Start Date Only**: Ranges from the specified start to the last available market day.
- **No Date Specified**: Provides the last 12 months of data.

---
## Date Parameter Rules for Precision
To maintain the integrity of your data queries:

- **Chronological Order**: Start dates must precede end dates.
- **Database Boundaries**: Dates are automatically adjusted to fit within FastTrack's historical records.

---
## FastTrack Date Formats Explained
FastTrack utilizes three date formats for versatility:

- **strdate**: The standard calendar date format (yyyy-MM-dd).
- **marketdate**: An integer count of market days since FastTrack's inception, ideal for sequential market analysis.
- **juliandate**: Compatible with Excel's DATEVALUE, facilitating easy integration with spreadsheets.

---
## Navigating Market Dates
Discover how to leverage market dates for bulk data analysis and enhance your financial models with our intuitive examples:

[Market Days Examples and Usage](https://docs.fasttrack.net/docs/ftlightning/5108ce6297e7a-javascript-bulk-data#market-days)

