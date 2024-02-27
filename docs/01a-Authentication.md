# Authentication
## Secure Your Data Access with Authentication

### Before You Begin
<!-- theme: warning -->
> Ensure you have your free trial or subscription ready. Not there yet? Sign up for a free trial to get started with the FastTrack Market Data API.
>[Start Free Trial](https://subscribe.fasttrack.net/landing/api/apilanding.html)

## Authentication Overview
FastTrack Market Data API secures your data transactions with a robust token-based authentication system. Every API request must include specific headers to ensure secure access to our financial datasets.

### Required Headers
- `appid`: Your unique application identifier, a 36-character string provided in your sign-up confirmation email.

- `token` A temporary 36-character token obtained from the /auth/login endpoint for authenticated API access.

<!-- theme: warning -->
> ##### Stay Ahead of Token Expiration
> **Note on Token Validity**
>Tokens are refreshed daily around 7:20 AM, 6:35 PM, and 8:20 PM EST. Keep your data flow uninterrupted by renewing your tokens with the `/auth/login` endpoint as needed.

### How to Authenticate Your Requests
Authenticating is Simple:

1. **Retrieve Your Token:** Use the `/auth/login` endpoint with your **account number**, **password**, and **appid** provided during sign-up.
2. **Receive a User Info Object:** This includes the essential "token" for making subsequent API requests.

### Example: A GET Request with Authentication

```javascript
GET https://ftl.fasttrack.net/v1/data/sp-da HTTP/1.1
appid: ff867cec-64c8-4d5d-a8d7-3a956d696c5e
token: 0ca5061c-4688-4e4a-ab15-eadd7c2944c7
Host: ftl.fasttrack.net

```


### Getting Your Credentials
Upon signing up for the FastTrack Market Data API, you'll receive your **account number**, **password**, and **appid**. If you haven't signed up yet or need to retrieve your credentials:
[Sign up or manage your account here](https://subscribe.fasttrack.net/landing/api/apilanding.html)


---
## Ready for More?
<!-- theme: success -->
>Dive deeper into our API with a [Login Code Example](https://docs.fasttrack.net/docs/ftlightning/ZG9jOjMwNzY4OQ) that showcases a practical implementation of our authentication process.



### Your Next Steps
Unlock the power of financial data with your FastTrack account. [Sign up now](https://subscribe.fasttrack.net/landing/api/apilanding.html)


