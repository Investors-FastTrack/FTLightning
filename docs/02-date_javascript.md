# Using Bulk Data

<!--
title: "My code snippet"
lineNumbers: true
-->

### Bulk Data Results Objects
The FastTrack database starts on 9/1/88. 

The bulk data end points return data points for *ALL DATES* in the FastTrack Database.


### Important

When you call /data/{ticker}/divadjprices, the result is an array of closing prices. The array's indexes are in order from the first date in the database to the most recent. 

Index 0 is the same value of index 1. 

Index 1 is the 9/1/88, index 2 is 9/2/88, index 3 is 9/6/1988.

The /data/dates endpoint returns an array of date objects that is necessary to convert dates to the right index.

### Request Dates


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
