# Handling Error Codes in FastTrack Market Data API


Understanding and efficiently handling error responses is crucial for maintaining the robustness of applications using the FastTrack Market Data API. Our system employs a straightforward approach to error reporting, ensuring you can quickly identify and rectify issues.

## Error Response Strategy
### HTTP Status Codes
- **200 (OK)**: Indicates a successful request. However, it may still contain an error within the response body if there's an issue with the query.
- **400 (Bad Request)**: Returned for requests that the API cannot process due to fatal errors, such as malformed request syntax.

### Error Object
Every response includes an err object. A null value signifies a successful operation, while any non-null value indicates an error. Always check the err object to ensure the integrity of the response.

```json
{
  "err": null, // or an error object if something went wrong
}


```

### Error Magic Numbers
When querying the API, a single error in data computation does not affect the entire data set's integrity. To maintain efficiency and ensure clear error signaling, FastTrack utilizes specifically defined, extremely low negative numbers to represent errors or uncomputable values.

#### Example Usage:
If a request is made for a 3-year standard deviation for a security that has only 1 year of data available, the API will return a value of -999.99.

#### Best Practices:
It is advisable for clients to check for values less than -999 as part of their error handling routine. This approach helps in identifying and segregating erroneous or uncomputable data points efficiently.

This practice allows for quick identification of errors during automated processes and reduces the chances of misinterpretations or incorrect data analysis due to unexpected error values.

## Best Practices
- **Null Check**: Before processing the results, confirm that the err object is not null. A null err object typically indicates that your request was processed successfully without errors.

- **Magic Number Check**: Routinely check for extremely negative values in your data. Values significantly below zero, such as less than -999, are used to flag uncomputable or invalid data points.

- **Error Handling**: Utilize the error codes and messages provided in the err object to develop robust error handling mechanisms within your application. These details will guide you in managing errors effectively and ensure that your application responds appropriately to various data anomalies.

## Common Error Codes and Solutions

Error Code | Definition | Suggested Action
---------|---------|---------
-1500000 | Invalid ticker  |Verify the ticker symbol and retry.
-1510000 | Subscription required to access this data.|Upgrade your subscription to access the requested data.
**Dates**|  
-1600000 to -1614000 |Date-related errors (invalid format, range issues)| Ensure date parameters meet the API requirements (yyyy-MM-dd is always best practice).
**Parameters**| 
-1630000 | Invalid count value. Parameter must be a number greater than 0 or A to return ALL values|Ensure the count is a positive number or 'A' for all values.
-1620000|Invalid chart value. Parameter must be 0 or 1|Parameter must be 0 or 1.
-1671000|Requested model does not exist.|Review model request
-1700000 | Invalid/Unable to read post body | Review post body
**Family**| 
-2000000 | Family does not exist.| Must be exact match. Use search to get exact name
-2100000 | Family search can not be blank.
-4000000|Unknown Error
**Models**|
-4200000 to -4400000| Model-related errors|Verify model parameters and retry.
**Auth**| 
-5000000 to -5810000 | Authentication and subscription errors|Confirm your credentials, app ID, and subscription status.

### Detailed Error Responses
Errors provide detailed messages to guide troubleshooting. Example:
```json
{
  "err": {
    "code": -1500000,
    "message": "Invalid ticker symbol. Please verify and try again."
  }
}

```

## Fatal Request Errors (HTTP 400)
Fatal errors result in an HTTP 400 response, indicating issues such as malformed requests or invalid query parameters. Review your request syntax and parameters to ensure compliance with the API specifications.

## Further Assistance
If you encounter an error that you cannot resolve or need clarification on error codes, please contact our support team at [api@fasttrack.net](mailto:api@fasttrack.net). Our goal is to ensure a seamless integration and efficient troubleshooting process for all users of the FT Market Data API.
