# Login and Data Access Example

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