# Javascript - Working with Bulk Data in FT Market Data API

<!--
title: "My code snippet"
lineNumbers: true
-->

The FT Market Data API provides access to comprehensive timeseries data for all trading dates since September 1, 1988. These "bulk data" endpoints are essential for applications requiring extensive historical market data.

## Bulk Data Endpoints
The API includes two endpoints for accessing bulk data:

- `GET /data/{ticker}`:  Retrieves timeseries data for a specific ticker.
- `POST /data/xmulti`:  Allows for multiple tickers' data retrieval in a single request.


## Understanding Data Format
The timeseries data is delivered as an array of numerical values, organized from the earliest to the most recent trading day in the database.

## Market Days Concept
Each index in the data array corresponds to a "market day," enabling data retrieval by referencing specific market days.

For instance, a request to `/data/JPM/divadjprices` returns an array `FTData.prices`, where:
- `FTData.prices[1]` accesses data for the first market day.
- `FTData.prices[2]` for the second, and so on.


## Array Characteristics
- **Size**: The length of the data arrays matches the total number of market days in the FastTrack database, starting from September 1, 1988. With each market day's close, the array grows by one index.
- **First Day of Data**: The database begins on September 1, 1988.


## Date Translation
Utilizing the `/data/dates/` endpoint is crucial for translating bulk data array indexes to actual calendar dates and vice versa.


## Important Considerations
The bulk data arrays' indexing starts with the database's first date, leading to specific behaviors:

- Index 0 is a placeholder, with the actual data starting from index 1, corresponding to September 1, 1988.
- Subsequent indexes follow the chronological order of market days.


## Preparing for Data Requests

Prior to requesting bulk data, it's recommended to download and locally store the array from `[`v1/data/dates`]`. This array, containing all database dates from the earliest to the latest, is vital for accurately matching date indexes with ticker data.

### Fetching Date Array
```javascript
var token = 'XXXXX-XXXX-XXXXx';
var appid = 'XXXXX-XXXX-XXXXX';

var ftdate = {};
var dte = null;

function downloadDates() {
    //call ftlightning and get data object
  fetch("https://ftl.fasttrack.net/v1/data/dates", 
  {
    headers: {
      "appid": appid,
      "token": token
    }
  })
	.then(response => response.json())
	.then(function (result) {
	    ftdate = result;
	})
}
```


### Requesting Adjusted Closing Prices

```javascript
var ftdata;

function downloadData() {

  var ticker = "TSLA";

    //call ftlightning and get data object
  fetch("https://ftl.fasttrack.net/v1/data/" + ticker, 
  {
    headers: {
      "appid": appid,
      "token": token
    }
  })
	.then(response => response.json())
	.then(function (result) {
	    ftdata = result;
	})
}
```


### Retrieving Specific Price Values
To find a specific date's closing price, iterate through the `v1/data/dates` object, match the desired date, and use the corresponding index to fetch the price from the bulk data array.
```javascript
function getDate(dte) {

// IMPORTANT//
// loop through ftdate object and find the "strdate"
// that matches your desired date
// get the "marketday" from ftdate.dtes object that matches your desired date
// marketdate  == index in data array
// use that index to get the value from data.prices

  for (var i = 0; i < ftdate.dtes.length; i++) {

    if (Date.parse(ftdate.dtes[i].strdate) == dte) {
      /// get marketday of desired date
      // market day == index in the "data.prices" array
      var mktday = ftdate.dtes[i].marketdate;

      // set value in form
      document.getElementById("output").value = data.prices[mktday];
      //exit loop
      break;
    }
  }
}

```

By following these guidelines, developers can efficiently work with bulk data from the FT Market Data API, enabling the construction of rich, data-driven financial applications.
