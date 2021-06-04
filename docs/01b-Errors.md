# Error Codes

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
-1510000 | Subscription required to access this data. Only indexes will return data. Search view [https://ftlightning.fasttrack.net/v1/family/All%20Index] for list of available indexes
**Dates**|  _
-1600000 | Invalid start and/or end date
-1610000 | Invalid start date. Start date cannot be after end date
-1611000 | Invalid start date. Start date cannot be equal to end date
-1612000 | Invalid start date. Try yyyy-MM-dd format.
-1613000 | Invalid end date. Try yyyy-MM-dd format.
-1614000 | Invalid end date. End date can not be on or before 1988-09-01
**Parameters**| _
-1630000 | Invalid count value. Parameter must be a number greater than 0 or A to return ALL values
-1620000|Invalid chart value. Parameter must be 0 or 1
-1671000|Requested model does not exist.
-1700000 | Invalid/Unable to read post body 
-2000000 | Family does not exist. Must be exact match. Use search to get exact name
-2100000 | Family search can not be blank.
-4000000|Unknown Error
**Models**|_
-4200000| Requested static model does not exist
-4300000|Unable to delete static model
-4400000|Invalid Momentum Model Parameters \[parameter\]
**Auth**| _
-5000000 | Invalid Account/Password
-5200000 | Invalid App ID
-5210000|Credentials not valid for this App ID
-5220000|App ID not activated
-5300000 | No token or App ID provided. Token and App ID must be sent in request header
-5310000|Subscription expired
-5500000 | Invalid or expired token
-5600000 | Restricted to free user 
-5700000 | FastTrack's Excel Addin requires a FT Cloud+ subscription or atleast 2 FT4web database subscriptions.
-5800000 | Subscription does not include dividend export. Contact ftlightning@fasttrack.net to upgrade.
-5810000|Subscription does not include unadjusted prices or ticker/cusip translation. Contact ftlightning@fasttrack.net to upgrade.