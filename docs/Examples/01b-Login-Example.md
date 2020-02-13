# Login & Get Data

The example below is broken into two parts.

First, you must request a token from the login end point. That token should be saved locally and used in the header of every subsequent request.

Second, request dividend adjusted data from the data end point. The token recieved in the login request will be used in the second request header

> ### Authentication
> All requests to the API (except /auth/login) must contain the header **appid** and **token**


### Request Token

Pass your account number, password, and app id as query strings to the "auth/login" end point.

```javascript
var account = '123456';
var pass = '12345678';
var appid = 'xxxxxx-xxxxx-xxxxxxx';
var token;
var url = 'https://ftlightning.fasttrack.net/v1/auth/login';

function login(){
  url = url + "?account=" + account + "&pass=" + pass + "&appid=" + appid;
  fetch(url)
  .then(
      function (response) {
        return response.json();
      }
    )
  .then(
    function (userinfo) {
        if (userinfo.err == null) {
          apikey = userinfo.apikey;
          token = userinfo.token;
        }
        else {
          alert(userinfo.err.message);
        }
      }
    )
  .catch(
    function (err) {
      alert(err.message);
      }
    )
```
#### Request Data
Request all dividend adjusted closing prices for the ticker JPM. Include your app id and token recieved in the request above.

``` javascript

function getData(){
  var ticker = 'JPM' 
  var url = 'https://ftlightning.fasttrack.net/v1/data/' + ticker + '/divadjprices'
  fetch(url,
      {headers: {
        "appid" : appid ,
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
          var lastprice = ftdata.prices[ftdata.prices.length-1]
          alert("The last price of JPM is: $" + lastprice)
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