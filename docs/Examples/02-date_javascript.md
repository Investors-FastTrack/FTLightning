# Using Bulk Data

<!--
title: "My code snippet"
lineNumbers: true
-->

FT Lightning include 2 end points that return timeseries data for ALL dates since 9/1/1988. We refer to these as "bulk data" endpoints

- `GET /data/{ticker}`
- `POST /data/xmulti`


### Data Format
The timeseries data is returned as an array of numbers. The array is sorted first day in the database to last day in the database. 

### Market Days

The index of the data array is called the "market day", and data is retrieve from the array by referencing the market day.

For example, a request to `/data/JPM/divadjprices` will return an `FTData.prices` array.

Data for market day #1 is retrieved by referencing `FTData.prices[1]`

Data for market day #2 is retrieved by referencing `FTData.prices[2]`

Data for market day #2000 is retrieved by referencing `FTData.prices[2000]`



### Array Size
These data arrays are always sized to the total number of market days in the FastTrack database from 9/1/88 to present. 

Each market day close, the data array gets 1 index larger.


### First Day
The FastTrack database starts on 9/1/88. 


### Data Date Translation
The [/data/dates/] endpoint returns a complete list of market days in the database. This array of dates is critical to translating the bulk data arrays indexes to date and vice versa.


<!-- theme: warning -->
>### Important ! ! ! !
>Bulk data endpoints return an array of data values. The array's indexes are in order from the first date in the database to the most recent. 
>- Index 0 is the same value of index 1. 
>- Index 1 is the 9/1/88
>- Index 2 is 9/2/88
>- Index 3 is 9/6/1988
>- etc


### Request Dates

Before requesting any Bulk data, you need to download the [/data/dates/] array and store locally. This data is critical to translating the bulk data arrays.


```javascript
var ftdate = {};
var dte = null;

function downloadDates() {
    //call ftlightning and get data object
  fetch("https://ftlightning.fasttrack.net/v1/data/dates", 
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


### Request Adjusted Closing Prices
```javascript
var token = 'XXXXX-XXXX-XXXXx';
var appid = 'XXXXX-XXXX-XXXXX';
var ftdata;
function downloadData() {
    //call ftlightning and get data object
  fetch("https://ftlightning.fasttrack.net/v1/data/jpm/divadjprices", 
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


### Get Price Value

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
		    document.getElementById("output").value = data.prices[mktday]
		    //exit loop
		    break;
		}
	    }
	})
}
```
