# C\# Downloading Data 

## Step 1 - Authenicate to web service
FastTrack's API token is reset every 8 hours. Make a request to /v1/auth/login with your account number, password, and app id. 

Recieve the response and extract the token to use in all other API requests. 

```csharp
// authenicate to web service
// docs: https://fasttrack.stoplight.io/docs/ftlightning/openapi.json/paths/~1v1~1auth~1login/get

string account = "xxxxxx";
string pass = "xxxxxx";
string appid = "xxxxxx";
auth a = new auth();
a = (auth)makeGET("https://ftl.fasttrack.net/v1/auth/login?account=" + account + "&pass=" + pass + "&appid=" + appid, a, null, null);

```

## Step 2 - Request all ETFs Tickers
```csharp

// request all funds and ETFs in FT database with screener
// docs: https://fasttrack.stoplight.io/docs/ftlightning/openapi.json/paths/~1v1~1family~1screener/post

// create screener request
FTScreener ftr = new FTScreener();
ftr.screener = new List<FTScreenItem>();

// Add ALL - ETF screen to object
FTScreenItem fti1 = new FTScreenItem();
fti1.family = "ALL - ETF";/// family for ALL funds in database
fti1.operation = "add";
ftr.screener.Add(fti1);

// request screen
FTScreenerResult ftscreen = new FTScreenerResult();
ftscreen = (FTScreenerResult)makePOST("https://ftl.fasttrack.net/v1/family/screener", ftscreen, ftr, a.token, a.appid);

// Extract ticker list from screener. 
// Transform to List<> to send in POST body
string[] tickers = ftscreen.resulttickers.ToArray();

// Split ticker arrya into 200 ticker mini arrays
// to multithread request 
String[][] chunks = tickers
        .Select((s, i) => new { Value = s, Index = i })
        .GroupBy(x => x.Index / 200)
        .Select(grp => grp.Select(x => x.Value).ToArray())
        .ToArray();


```

## Step 3 - Get Stats

```csharp

// set end date to last day of prior month, start date last day of month 5 years prior
int pervious_month = DateTime.Now.AddMonths(-1).Month;
int pervious_year = DateTime.Now.AddMonths(-1).Year;

string dteend = pervious_year.ToString() + "-" + pervious_month.ToString("0#") + "-" + DateTime.DaysInMonth(pervious_year, pervious_month);
string dtestart = dtestart = (pervious_year - 5).ToString() + "-" + pervious_month.ToString("0#") + "-" + DateTime.DaysInMonth(pervious_year, pervious_month);

// multi thread the 200 ticker chunk request
// store all results in "master_statslist"

FTStatsList master_statslist = new FTStatsList();
master_statslist.statslist = new List<FTStats>();
ParallelOptions popt = new ParallelOptions() { MaxDegreeOfParallelism = 4 };
Parallel.For(0, chunks.Length, popt, i =>
{
    // request stats data for array of tickers
    // make sure to pass start and end dates as query strings
    // docs: https://fasttrack.stoplight.io/docs/ftlightning/openapi.json/paths/~1v1~1stats~1xmulti/post
    FTStatsList stats = new FTStatsList();
    stats = (FTStatsList)makePOST("https://ftl.fasttrack.net/v1/stats/xmulti?start=" + dtestart + "&end=" + dteend, stats, chunks[i], a.token, a.appid);
    lock (master_statslist)
    {
        master_statslist.statslist.AddRange(stats.statslist);
        Console.WriteLine(master_statslist.statslist.Count().ToString() + " / " + tickers.Count().ToString());
    }
});

```


Stats Classes

```csharp
public class FTStats
{
  public string ticker;

  public FTDate dtestart;
  public FTDate dteend;

  public FTChartData chartprices = new FTChartData();
  public FTRisk risk = new FTRisk();
  public FTDescribe describe = new FTDescribe();
  public FTReturns returns = new FTReturns();
  public FTMaxDraw max = new FTMaxDraw();

  public int aux;

  public err err;
}

public class FTStatsList
{
  public List<FTStats> statslist;
  public err err;
}

public class FTDescribe
{
  public string name;
  public string sharesout;
  public string price;
  public string marketcap;
  public string security_type;
  public string high_52;
  public string low_52;
  public string yield;
  public string yield_all;
  public string yieldL;
  public string yieldS;
  public FTDate startdate = new FTDate();
}

public class FTRisk
{
  public string sd;
  public string ui;
  public string upi;
  public string rsi;
  public string sharpe;
  public string alpha;
  public string ftalpha;
  public string corr;
  public string beta;
}

public class FTReturns
{
  public FTReturn annualized = new FTReturn();
  public FTReturn total = new FTReturn();
  public List<FTYearReturns> years = new List<FTYearReturns>();
}

public class FTReturn
{
  public string period;

  public string oneday;
  public string fiveday;

  public string onemonth;
  public string threemonths;
  public string sixmonths;
  public string ninemonths;

  public string one;
  public string three;
  public string five;
  public string ten;
  public string inception;
  public string ytd;
}

public class FTYearReturns
{
  public string year;
  public string returns;
  public string jan;
  public string feb;
  public string mar;
  public string apr;
  public string may;
  public string jun;
  public string jul;
  public string aug;
  public string sep;
  public string oct;
  public string nov;
  public string dec;
}

public class FTChartData
{
  public double[] p_log;
  public string[] p_percents;
  public string[] p_percents2;
  public string[] p_divadj;
  public FTDates dates;
}

public class FTMaxDraw
{
  public string maxdrawdown;
  public string maxdrawlength;
  public string recoverylength;
  public string peakdate;
  public string valleydate;
}

```
Details Classes

```csharp 
public class FTDetailsList
{
  public List<FTDetails> datalist;
  public err err;
}

public class FTDetails
{
  public string ticker { get; set; }
  public string expenseratio { get; set; }
  public expense_detail expense_detail { get; set; }
  public string category { get; set; }
  public string objective { get; set; }
  public string security_type { get; set; }
}

public class expense_detail
{
  public string deferred { get; set; }
  public string b12 { get; set; }
  public string frontload { get; set; }
  public string initpurchase { get; set; }
  public string redemption { get; set; }
}

#endregion

#region "SCREEN"

public class FTScreenItem
{
  public string operation;
  public string level;
  public string rank;
  public string family;
  public string count;
}

public class FTScreener
{
  public List<FTScreenItem> screener;
  public string name;
  public string dtecreated;
  public err err;
}

public class FTScreenerResult
{
  public List<FTScreenItem> screener;
  public string resulttotal;
  public string name;
  public string id;
  public string dtecreated;
  public List<string> resulttickers;
  public err err;
}
```

Authentication, Date, Error Classes
```csharp
public class auth
{
  public string accountnumber { get; set; }
  public string appid { get; set; }
  public err err { get; set; }
  public string expiration { get; set; }
  public string pass { get; set; }
  public string token { get; set; }
}

public class FTDates
{
  public FTDate[] dtes;
  public int maxnumdays;
  public string strmaxnumdays;
  public err err;
}

public class FTDate
{
  public string marketdate = null;
  public string juliandate = null;
  public string strdate { get; set; }
}

public class err
{
  public string code;
  public string link;
  public string message;
}
```