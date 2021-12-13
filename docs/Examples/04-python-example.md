# Python - Export Expense Ratios

### Authenticate to web service
Import the necesary libraries, then make a request to the v1/auth/login end point with your account, password and appid

Be sure to save the token in the auth result. The token must be included in the header of all other calls to access data. 

```python

import requests
import json

# use your own account number, password and appid to authenticate 
account = "******"
password = "********"
appid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"

# Build auth url
authurl = "https://ftl.fasttrack.net/v1/auth/login?account=" + account + "&pass=" + password + "&appid=" + appid 

# make authentication request
authresponse = requests.request("GET", authurl)

# parse result and extract token
a = json.loads(authresponse.text)

token = a['token']

```

### Get Expense ratio data
The code below requests data for multiple tickers from the /v1/ref/xmulti/details endpoint.
This data is parsed and printed to a CSV format. 

```python

# build url to access expense ratio details
url = "https://ftl.fasttrack.net/v1/ref/xmulti/details"

# request date for all "free" tickers in the FT Data API Trial
payload = "[\"SPAB\",\"SPBO\",\"SPDW\",\"SPEM\",\"SPEU\",\"SPGM\",\"SPHY\",\"SPIB\",\"SPIP\",\"SPLB\",\"SPLG\",\"SPMB\",\"SPMD\",\"SPSB\",\"SPSM\",\"SPTI\",\"SPTL\",\"SPTM\",\"SPTS\",\"SPYD\",\"SPYG\",\"SPYV\"]"

## construct a header object containing your credentials. 
# You will send this with the POST  request to authenticate
headers = {
    'appid': appid,
    'token': token,
    'Content-Type': "application/json"
    }

# request date
response = requests.request("POST", url, data=payload, headers=headers)

# parse data
y = json.loads(response.text)

# print data in CSV format
print("Ticker, Expense, Objective, Security Type")
for detail in y['datalist']:
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