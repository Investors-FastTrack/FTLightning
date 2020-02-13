# API Credentials

### Sign Up Free Account
Use this link to sign up for an account:
[https://subscribe.fasttrack.net/landing/api/apilanding.html](https://subscribe.fasttrack.net/landing/api/apilanding.html)

### Credentials
FastTrack will send an **account number**, **password**, and **app id** to your email.

### Authentication
Authentication is done via the **/auth/login** endpoint. 

Call this endpoint with valid credentials in the query string. A user info object is returned. This contains the "token" required to make all other requests 

### Errors
FT Lightning resonds with HTTP status codes of 200 or 400 only.

Errors are returned as a 200, and errors are indicated in the "err" object of the response. 

Valid responses will always have a "err" object set to null. 

Best practice is to test if the "err" object is null. If it is not null, then the response contains an error.

400 responses are returned only on fatal request errors.

### Javascript example
#### Get User Info and Token

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
#### Use token to Access Data
``` javascript

function getData(){
  var url = 'https://ftlightning.fasttrack.net/v1/data/jpm'
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
