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
FT Lightning responds with HTTP status codes of 200 or 400 only.

Errors are returned as a 200, and errors are indicated in the "err" object of the response. 

Valid responses will always have a "err" object set to null. 

Best practice is to test if the "err" object is null. If it is not null, then the response contains an error. Error codes and text are returned in err object.

400 responses are returned only on fatal request errors.

#### Error Codes

Error Code | Definition 
---------|----------
-1500000 | Invalid ticker  
-1510000 | Subscription required to access this data. Only indexes will return data. Search view https://ftlightning.fasttrack.net/v1/family/All%20Index for list of available indexes
-1600000 | Invalid start and/or end date
-1700000 | Invalid/Unable to read post body 
-2000000 | Family does not exist. Must be exact match. Use search to get exact name
-2100000 | Family search can not be blank.
-4000000|Unknown Error
-5000000 | Invalid Account/Password
-5200000 | Invalid App ID
-5300000 | No token or App ID provided. Token and App ID must be sent in request header
-5500000 | Invalid or expired token
-5600000 | Restricted to free user 
-5700000 | FastTrack's Excel Addin requires a FT Cloud+ subscription or atleast 2 FT4web database subscriptions.
-5800000 | Subscription does not include dividend export. Contact ftlightning@fasttrack.net to upgrade.


