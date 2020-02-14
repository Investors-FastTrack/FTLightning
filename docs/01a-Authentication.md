# Authentication

FT Lightning uses a token based authentication system. All request to FT Lightning must include two headers:


#### appid 
This is a unique 36 charager string that identifies your application. This is sent to in your sign up email.

#### token
This is a temporary 36 character string that is retrieved from the **/auth/login** endpoint.

<!-- theme: warning -->
>The token is reset daily at approximately 7:20am, 6:35PM, and 8:20pm EST.

### Example HTTP GET 
##### with Authentication Headers
```javascript
GET https://ftlightning.fasttrack.net/v1/data/sp-da HTTP/1.1
appid: ff867cec-64c8-4d5d-a8d7-3a956d696c5e
token: 0ca5061c-4688-4e4a-ab15-eadd7c2944c7
Host: ftlightning.fasttrack.net

```

### Authentication
Authentication is done via the **/auth/login** endpoint. 

Call this endpoint with valid **account number**, **password**, and **app id** in the query string. A user info object is returned. This contains the "token" required to make all other requests 



### Credentials
**account number**, **password**, and **app id** are sent when you sign up for FT Lightning API

If you need credentials, Use this link to sign up for an account:
[https://subscribe.fasttrack.net/landing/api/apilanding](https://subscribe.fasttrack.net/landing/api/apilanding.html)


---
<!-- theme: success -->
>### **[Jump to Login Code Example](./Examples/01b-Login-Example.md)**

---

### Sign Up Free Account
Use this link to sign up for an account:
[https://subscribe.fasttrack.net/landing/api/apilanding.html](https://subscribe.fasttrack.net/landing/api/apilanding.html)


