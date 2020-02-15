# Login

The code below shows how exchange your account number, password, and appid for a temporary token.

The temporary token and appid must be passed as a header to all other FT Cloud API requests.

>Save the token as a local variable. The request to the /auth/login end point only needs to be called once.

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

