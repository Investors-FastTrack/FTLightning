# VBA Integration Example

## Overview

FastTrack's Market Data API can be integrated with Microsoft Excel to retrieve and display historical financial data for a list of tickers. This guide includes step-by-step instructions and VBA code to:

- Make an API request using `WinHttp.WinHttpRequest.5.1`
- Parse CSV-like response data
- Write the results into an Excel worksheet
- Trigger the request using a Form Button

---

## Prerequisites

- Microsoft Excel with macro support enabled
- A valid **FastTrack API Token** and **App ID**
- Internet access
- Reference to `WinHttp.WinHttpRequest.5.1` (typically available by default)

---

## Endpoint: `GET /v1/export/xmulti/{tickerlist}`

**Base URL:**  
`https://ftl.fasttrack.net/v1/export/xmulti/`

**Query Parameters:**
- `token` – *Your API token*
- `appid` – *Your application ID*
- `start` – *Start date in YYYY-MM-DD format*
- `end` – *End date in YYYY-MM-DD format*

---

## VBA Code – Output CSV Response to Worksheet

The API response is a CSV-formatted string, which we can parse and write to an Excel sheet:

```vba
Sub Test_FastTrack_Request(tickers As String, startDate As String, endDate As String, token As String, appId As String)

    Dim strUrl As String
    strUrl = "https://ftl.fasttrack.net/v1/export/xmulti/" & tickers & _
             "?token=" & token & "&appid=" & appId & "&start=" & startDate & "&end=" & endDate

    Dim objRequest As Object
    Set objRequest = CreateObject("WinHttp.WinHttpRequest.5.1")

    With objRequest
        .Open "GET", strUrl, False
        .SetRequestHeader "Content-Type", "application/json"
        .Send
    End With

    Dim responseText As String
    responseText = objRequest.ResponseText

    ' Split response into lines
    Dim lines() As String
    lines = Split(responseText, vbLf)

    Dim ws As Worksheet
    Set ws = ThisWorkbook.Sheets("Sheet1")
    ws.Cells.ClearContents

    Dim i As Long, j As Long
    Dim values() As String

    ' Write line by line into worksheet
    For i = 0 To UBound(lines)
        If Trim(lines(i)) <> "" Then
            values = Split(lines(i), ",")
            For j = 0 To UBound(values)
                ws.Cells(i + 1, j + 1).Value = Trim(values(j))
            Next j
        End If
    Next i

End Sub
```

---

## Adding a Button in Excel

To let users trigger the macro with a button:

1. Open Excel.
2. Go to the **Developer** tab. (If it's hidden, enable it via File > Options > Customize Ribbon > check *Developer*)
3. Click **Insert** > choose a **Form Control Button**.
4. Draw the button on the worksheet.
5. When prompted, assign the macro: `CommandButton1_Click`

---

## Macro to Trigger API Request

```vba
Sub CommandButton1_Click()

    Dim token As String: token = "your_token_here"
    Dim appId As String: appId = "your_appid_here"

    Dim tickers As String: tickers = "JPM,BAC"
    Dim startDate As String: startDate = "2025-04-09"
    Dim endDate As String: endDate = "2025-05-09"

    Call Test_FastTrack_Request(tickers, startDate, endDate, token, appId)

End Sub
```

---

## Notes

- Ensure that `"Sheet1"` exists or change the sheet name in the code.
- The CSV header includes: `ticker, date, adjusted price, unadjusted price, open, low, high, volume`.
- Adjust column formatting (e.g., date or number formats) as needed.
- You do **not** need a JSON parser for this example since the response is plain CSV.

---

## Optional: Add Basic Error Handling

```vba
On Error GoTo ErrHandler

' ... request code ...

Exit Sub

ErrHandler:
    MsgBox "An error occurred: " & Err.Description, vbCritical
```
