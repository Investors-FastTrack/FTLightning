---
stoplight-id: g4bu9w0lwxy00
---

# Javascript - Performance Download

This is a JavaScript function that is used to download performance data for a specified ticker.

The function first gets the ticker, token, and appid from the query string.

Then it gets the month end and quarter end date using the functions getMonthEndDate() and getLastDayPriorQuarter().

Next, it uses the fetch() method to make three requests to FastTrack for the current, month end, and quarter end performance data.
The response JSON and assigned to variables.

The HTML is updated fund's name "header_name"

Finally, three data tables are updated using the jQuery DataTable.net plugin, one for current performance data, one for month end date, and one for quarter end data.

Table columns are mapped to the JSON response. Add or remove items from each tables columns[] object to update table with new columns


