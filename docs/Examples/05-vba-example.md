# VBA Code Example
## For Microsoft Excel or Access


FastTrack's Market Data API can integrate into many differnt systems, including traditional Microsoft Office applications like Excel and Access.



## HTTP GET Request Sub
The code below builds a `WinHttp.WinHttpRequest.5.1` request object to make a HTTP GET request to the **/v1/export/xmulti/{tickerlist}** end point
```csharp

Sub Test_FastTrack_Request(tickers As String, startdate As String, enddate As String, token As String, appid as String)  
    
  '--- URL to request
  Dim strUrl As String
  strUrl = "https://ftl.fasttrack.net/v1/export/xmulti/" + tickers + "?token=" + token + "&appid=" + appid + "&start=" + startdate + "&end=" + enddate

  '--- Response from http request
  Dim strResponse As String

  '--- create windows http request object to make HTTP request
  Dim objRequest As Object
  Set objRequest = CreateObject("WinHttp.WinHttpRequest.5.1")

  '--- build request object
  With objRequest
    .Open "GET", strUrl, False, "XXXX", "XXXX"
    .SetRequestHeader "Content-Type", "application/json"
    .Send body
    strResponseHeaders = .StatusText
    strResponse = .ResponseText
    allResponseHeader = .GetAllResponseHeaders
  End With

  '--- write http response to debug window
  Debug.Print strResponse

End Sub

```

## Add Action to call GET Sub
Add a button or action item to your Excel/Access Application to trigger the download. 

Of course, update the ticker list, appid, token, and dates to meet your project's needs.

```csharp
Private Sub CommandButton1_Click()

Dim token As String
token = "" ''' add your token here

Dim appid As String
appid = "" ''' add your appid here


Test_FastTrack_Request "JPM,BAC,GS,MS", "2022-01-01", "2022-01-23", token, appid


End Sub
```