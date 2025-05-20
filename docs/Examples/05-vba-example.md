# VBA Code Example for FastTrack Market Data API
## Integrating with Microsoft Excel or Access

The **FastTrack Market Data API** enables seamless integration with Microsoft Office applications like Excel and Access, allowing users to retrieve historical market data, statistics, and portfolio modeling results. This guide demonstrates how to:

1. **Screen for Vanguard Funds**: Use the `/v1/family/screener` endpoint to retrieve the "All - Vanguard" fund family.
2. **Fetch Market Data**: Use the `/v1/export/xmulti/{TickerList}` endpoint to download CSV data for the first 10 tickers from the screener results.

---

## Prerequisites

- **API Credentials**:
  - Obtain an `appid` (36-character GUID) from the welcome email after signing up for a trial at [http://apitrial.fasttrack.net](http://apitrial.fasttrack.net).
  - Acquire a `token` (36-character GUID) from the `/auth/login` endpoint.
- **VBA Environment**: Ensure your Excel or Access environment has the `WinHttp.WinHttpRequest.5.1` library enabled for HTTP requests.
- **Security**: Store your `appid` and `token` securely (e.g., in a configuration file or environment variable) to avoid exposing them in the code.

---

## Overview of API Endpoints Used

### 1. `/v1/family/screener` (POST)
- **Purpose**: Searches for fund families based on a query, such as "All - Vanguard".
- **Request**:
  - **Method**: POST
  - **Headers**: `appid`, `token`
  - **Body**: JSON object with a `query` field (e.g., `{"query": "All - Vanguard"}`).
- **Response**: Returns a JSON object (`FTFamilySearchResults`) containing a list of matching families, including their names, ticker counts, and comma-separated ticker lists.
- **Example Response**:
  ```json
  {
    "searchresults": [
      {
        "family_name": "All - Vanguard",
        "ticker_count": "50",
        "tickers": "VFINX,VTSMX,..."
      }
    ],
    "err": null
  }
  ```

### 2. `/v1/export/xmulti/{TickerList}` (GET)
- **Purpose**: Retrieves historical market data (e.g., OHLCV) for multiple tickers over a specified date range in CSV format.
- **Request**:
  - **Method**: GET
  - **Headers**: `appid`, `token`
  - **Path Parameter**: `TickerList` (comma-separated ticker symbols, e.g., `VFINX,VTSMX`).
  - **Query Parameters**:
    - `start`: Start date (yyyy-MM-dd).
    - `end`: End date (yyyy-MM-dd, defaults to the last market day).
    - `unadj`: Include unadjusted prices (0 or 1, default 0).
    - `olhv`: Include open, low, high, volume data (0 or 1, default 0).
- **Response**: Returns a CSV string with the following columns: `ticker`, `date`, `adjusted price`, `unadjusted price`, `open`, `low`, `high`, `volume`.
- **Example Response**:
  ```csv
  ticker,date,adjusted price,unadjusted price,open,low,high,volume
  JPM,5/16/2025,267.56,267.56,267.5,264.71,268.459,8932.91
  JPM,5/19/2025,264.88,264.88,265.55,261.93,268.32,12140.24
  BAC,5/16/2025,44.69,44.69,44.34,43.655,44.79,49488.14
  BAC,5/19/2025,44.77,44.77,44.11,44.11,45.14,37898.77
  GS,5/16/2025,619.03,619.03,616.95,613.56,620.79,2021.77
  GS,5/19/2025,612.3,612.3,609.19,598.34,619.38,2841.08
  ```

---

## VBA Code

### Step 1: HTTP Request Function
This function creates a reusable `WinHttp.WinHttpRequest.5.1` object to send HTTP requests to the FastTrack API.

```vba
Function MakeFastTrackRequest(url As String, method As String, token As String, appid As String, Optional body As String) As String
    Dim objRequest As Object
    Dim strResponse As String
    Dim strStatus As String
    
    ' Create HTTP request object
    Set objRequest = CreateObject("WinHttp.WinHttpRequest.5.1")
    
    On Error GoTo ErrorHandler
    
    ' Build and send request
    With objRequest
        .Open method, url, False
        .SetRequestHeader "Content-Type", "application/json"
        .SetRequestHeader "appid", appid
        .SetRequestHeader "token", token
        If method = "POST" And Len(body) > 0 Then
            .Send body
        Else
            .Send
        End If
        strStatus = .StatusText
        strResponse = .ResponseText
    End With
    
    ' Check for successful response
    If InStr(strStatus, "OK") = 0 Then
        MsgBox "API Request Failed: " & strStatus & vbCrLf & "Response: " & strResponse, vbCritical
        MakeFastTrackRequest = ""
        Exit Function
    End If
    
    MakeFastTrackRequest = strResponse
    Exit Function

ErrorHandler:
    MsgBox "Error in HTTP Request: " & Err.Description, vbCritical
    MakeFastTrackRequest = ""
End Function
```

### Step 2: Retrieve Vanguard Fund Family
This subroutine queries the `/v1/family/screener` endpoint to retrieve the "All - Vanguard" fund family and extracts the first 10 tickers.

```vba
Sub GetVanguardFunds(token As String, appid As String)
    Dim strUrl As String
    Dim strBody As String
    Dim strResponse As String
    Dim tickers As String
    Dim jsonResponse As Object
    Dim tickerList As String
    
    ' Set API endpoint and request body
    strUrl = "https://ftl.fasttrack.net/v1/family/screener"
    strBody = "{""query"": ""All - Vanguard""}"
    
    ' Make POST request
    strResponse = MakeFastTrackRequest(strUrl, "POST", token, appid, strBody)
    
    If strResponse = "" Then
        Exit Sub
    End If
    
    ' Parse JSON response (requires VBA-JSON library or similar)
    ' Note: You may need to add a JSON parser library, such as VBA-JSON
    Set jsonResponse = ParseJson(strResponse) ' Replace with your JSON parsing method
    
    ' Extract tickers (comma-separated string)
    tickerList = jsonResponse("searchresults")(1)("tickers")
    
    ' Split tickers and take the first 10
    Dim tickerArray As Variant
    tickerArray = Split(tickerList, ",")
    
    If UBound(tickerArray) >= 9 Then
        tickers = Join(Array(tickerArray(0), tickerArray(1), tickerArray(2), tickerArray(3), tickerArray(4), _
                             tickerArray(5), tickerArray(6), tickerArray(7), tickerArray(8), tickerArray(9)), ",")
        Debug.Print "Selected Tickers: " & tickers
    Else
        MsgBox "Not enough tickers found in the Vanguard family.", vbExclamation
        Exit Sub
    End If
    
    ' Call function to fetch data for these tickers
    FetchTickerData tickers, "2022-01-01", "2022-01-23", token, appid
End Sub
```

**Note**: The `ParseJson` function requires a JSON parsing library (e.g., [VBA-JSON](https://github.com/VBA-tools/VBA-JSON)). Alternatively, implement a custom parser to extract the `tickers` field from the response.

### Step 3: Fetch and Parse Market Data
This subroutine uses the `/v1/export/xmulti/{TickerList}` endpoint to retrieve CSV market data for the specified tickers and includes a basic example of parsing the CSV response.

```vba
Sub FetchTickerData(tickers As String, startdate As String, enddate As String, token As String, appid As String)
    Dim strUrl As String
    Dim strResponse As String
    Dim csvLines As Variant
    Dim i As Long
    Dim lineData As Variant
    
    ' Build URL for GET request
    strUrl = "https://ftl.fasttrack.net/v1/export/xmulti/" & tickers & _
             "?token=" & token & "&appid=" & appid & "&start=" & startdate & "&end=" & enddate & "&olhv=1&unadj=1"
    
    ' Make GET request
    strResponse = MakeFastTrackRequest(strUrl, "GET", token, appid)
    
    If strResponse = "" Then
        Exit Sub
    End If
    
    ' Output raw CSV response to debug window
    Debug.Print "Raw CSV Response:"
    Debug.Print strResponse
    
    ' Parse CSV response
    csvLines = Split(strResponse, vbCrLf)
    
    ' Skip header row and process data
    For i = 1 To UBound(csvLines)
        If Len(Trim(csvLines(i))) > 0 Then
            lineData = Split(csvLines(i), ",")
            If UBound(lineData) >= 7 Then
                ' Example: Write to worksheet or process data
                Debug.Print "Ticker: " & lineData(0) & ", Date: " & lineData(1) & ", Adjusted Price: " & lineData(2)
                ' Optional: Write to worksheet (example below)
                ' Cells(i, 1).Value = lineData(0) ' Ticker
                ' Cells(i, 2).Value = lineData(1) ' Date
                ' Cells(i, 3).Value = lineData(2) ' Adjusted Price
                ' Cells(i, 4).Value = lineData(3) ' Unadjusted Price
                ' Cells(i, 5).Value = lineData(4) ' Open
                ' Cells(i, 6).Value = lineData(5) ' Low
                ' Cells(i, 7).Value = lineData(6) ' High
                ' Cells(i, 8).Value = lineData(7) ' Volume
            End If
        End If
    Next i
End Sub
```

### Step 4: Trigger the Workflow
Add a button or action in your Excel/Access application to initiate the process.

```vba
Private Sub CommandButton1_Click()
    Dim token As String
    Dim appid As String
    
    ' Replace with your actual credentials
    token = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" ' Add your token
    appid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" ' Add your appid
    
    ' Start the process
    GetVanguardFunds token, appid
End Sub
```

---

## Additional Notes

### Handling CSV Responses
- The `/v1/export/xmulti/{TickerList}` endpoint returns a CSV string. The `FetchTickerData` subroutine includes a basic example of parsing the CSV by splitting on newlines and commas.
- For robust CSV parsing, consider using a VBA CSV parsing library or writing data to an Excel worksheet for further processing.
- The CSV includes headers (`ticker`, `date`, `adjusted price`, `unadjusted price`, `open`, `low`, `high`, `volume`). Ensure your parsing logic skips the header row.

### Handling JSON Responses
- The `/v1/family/screener` endpoint returns a JSON object. Use a VBA JSON parser like [VBA-JSON](https://github.com/VBA-tools/VBA-JSON) to extract the `tickers` field. Alternatively, implement a custom parser.

### Error Handling
- The `MakeFastTrackRequest` function includes basic error handling for HTTP failures and invalid responses. Enhance it based on the error codes listed in the [FastTrack Error Codes](https://docs.fasttrack.net/docs/ftlightning/ZG9jOjMwNzY2Mw-error-codes).
- Check for specific error codes (e.g., `-5000000` for invalid ticker) in the `err` object of the `/v1/family/screener` response or validate the CSV response for errors.

### Performance Considerations
- The `/v1/export/xmulti/{TickerList}` endpoint can return large CSV datasets. Limit the date range (`start` and `end`) to reduce response size.
- Set `olhv=1` and `unadj=1` only if you need open, high, low, volume, and unadjusted price data.

### Security Best Practices
- Avoid hardcoding `appid` and `token` in your VBA code. Store them in a secure location, such as a password-protected configuration file or environment variable.
- Implement a token refresh mechanism using the `/auth/login` endpoint to handle token expiration.

### Extending the Code
- **Write to Worksheet**: Uncomment the worksheet writing code in `FetchTickerData` to populate an Excel worksheet with CSV data.
- **Dynamic Date Ranges**: Allow users to input start and end dates via a form or cell values.
- **Batch Processing**: If the Vanguard family contains many tickers, process them in batches to avoid exceeding API limits.

---

## Example Workflow
1. Click the button in your Excel/Access application.
2. The `GetVanguardFunds` subroutine sends a POST request to `/v1/family/screener` with the query "All - Vanguard".
3. The response is parsed to extract the first 10 tickers (e.g., `VFINX,VTSMX,...`).
4. The `FetchTickerData` subroutine sends a GET request to `/v1/export/xmulti/{TickerList}` to retrieve CSV market data for these tickers from January 1, 2022, to January 23, 2022.
5. The CSV response is parsed and printed to the debug window (or can be written to a worksheet).

---

## Troubleshooting

- **Invalid Credentials**: Ensure your `appid` and `token` are correct and not expired.
- **No Tickers Returned**: Verify that the "All - Vanguard" family exists in the screener results. Check the JSON response for errors.
- **CSV Parsing Errors**: Ensure the CSV response is well-formed and handle cases where rows may have missing data.
- **API Errors**: Refer to the [FastTrack Error Codes](https://docs.fasttrack.net/docs/ftlightning/ZG9jOjMwNzY2Mw-error-codes) for details on error messages.

---

## Contact Support
For assistance, contact FastTrack API Support:
- **Email**: [api@fasttrack.net](mailto:api@fasttrack.net)
- **Website**: [http://www.fasttrack.net](http://www.fasttrack.net)

---

This documentation and code provide a robust starting point for integrating the FastTrack Market Data API into your Excel or Access application. Customize the code to fit your specific requirements, such as adding data visualization or additional API endpoints.