# Python Example

### Authenticate to web service
Import the necesary libraries, then make a request to the `/v1/auth/login` end point with your account, password and appid

Be sure to save the token in the auth result. The token must be included in the header of all other calls to access data. 

```python

import requests
import json

# use your own account number, password and appid to authenticate 
account = "******"
password = "********"
appid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"

# Build auth url
authurl = f"https://ftl.fasttrack.net/v1/auth/login?account={account}&pass={password}&appid={appid}"

# make authentication request
authresponse = requests.get(authurl)

# parse result and extract token
token = authresponse.json()['token']

```

### Get Expense ratio data
The code below requests data for multiple tickers. We are using the `/v1/ref/{ticker}/details` GET request in a loop. This data is parsed, collected, and then printed to a CSV format. 

**Trial accounts can only access GET requests. For paid accounts, use the POST request for more efficient data retrieval**



```python

tickers = ["SPAB", "SPBO", "SPDW", "SPEM", "SPEU", "SPGM", "SPHY", "SPIB", "SPIP", "SPLB", "SPLG", "SPMB", "SPMD", "SPSB", "SPSM", "SPTI", "SPTL", "SPTM", "SPTS", "SPYD", "SPYG", "SPYV"]

# construct a header object containing your credentials. 
headers = {
    'appid': appid,
    'token': token
}

# Initialize an empty list to hold all details
all_details = []

for ticker in tickers:
    # Build URL for each ticker
    url = f"https://ftl.fasttrack.net/v1/ref/{ticker}/details"

    # Make GET request for each ticker
    response = requests.get(url, headers=headers)

    # Check if the response is successful
    if response.status_code == 200:
        detail = response.json()
        # Append the detail to the all_details list
        all_details.append(detail)
    else:
        print(f"Failed to get data for ticker: {ticker}")

# Print all items in all_details
print("Ticker, Expense, Objective, Security Type")
for detail in all_details:
    print(
        detail['ticker'],
        ",",
        '{:.2f}%'.format(float(detail["expenseratio"])*100),
        ",",
        detail["objective"],
        ",",
        detail["security_type"]
    )

  ```