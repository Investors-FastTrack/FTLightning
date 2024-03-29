# Javascript - Get Price



After retrieving your token from the [/auth/login](./01b-Login-Example.md) request, you can now use the retrieved token to access the other API functions.

In this example, the code requests dividend adjusted data from the /data/{ticker} end point. 

> All requests to the API (except [/auth/login](./01b-Login-Example.md)) must contain the headers **appid** and **token**

#### Request Data
In this example, the code request all dividend adjusted closing prices for the ticker JPM. 

The appid and token recieved in the [/auth/login](./01b-Login-Example.md) request are included in the HTTP GET header.

Once the data is downloaded, the code pops and alert to indicate JPM's most recent closing price.




``` javascript

function getData(){
  var ticker = 'JPM';
  var url = 'https://ftl.fasttrack.net/v1/data/' + ticker + '/range';

  fetch(url,
      {headers: {
        "appid" : appid,
        "token" : token
        }}
    )
  .then(
      function (response) {
        return response.json();
      }
    )
  .then(
    function (ftdata) {
        if (ftdata.err == null) {

          var last_price = ftdata.datarange[ftdata.datarange.length - 1].price;
          var last_date = ftdata.datarange[ftdata.datarange.length - 1].date.strdate;
          var name = ftdata.name;

          alert("The last price of " + ftdata.name + " is $" + last_price + " for date " + last_date);
        
        }
        else {
        
          alert(ftdata.err.message);
        
        }
      }
    )
  .catch(
    function (err) {
      alert(err.message);
      }
    )
}
```

### Bulk Data Format
Read more about the Bulk Data format and how to translate array to date/price values: [/data/dates example](./02-date_javascript.md)