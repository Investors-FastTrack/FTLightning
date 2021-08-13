# Login

The code below shows how exchange your account number, password, and appid for a temporary token.

The temporary token and appid must be passed as a header to all other FT Market Data API requests.

>Save the token as a local variable. The request to the /auth/login end point only needs to be called once.

### Request Token

Pass your account number, password, and app id as query strings to the "auth/login" end point.

```javascript
var account = '123456';
var pass = '12345678';
var appid = 'xxxxxx-xxxxx-xxxxxxx';
var token;
var url = 'https://ftl.fasttrack.net/v1/auth/login';

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

### Save Token Locally
Save the retrieved token locally and reuse until expires. 

### Token Expiration
The token is reset daily at approximately 
- 7:20am EST
- 6:35PM EST
- 8:20pm EST

Request new tokens periodically by calling the /auth/login endpoint again.

Any time an expired or invalid token is used, FT Market Data API returns a -5500000 Invalid Token error. 

See all errors here: [Error Codes](../01b-Errors.md)

