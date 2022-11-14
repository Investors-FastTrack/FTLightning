---
stoplight-id: rbosu9gmxd4tv
---

# Overview

Use this API to backtest investment portfolios.

You supply the investment portfolio weightings and rebalance, FastTrack calculate stats, holdings, values, and more.

## Model Types

The API offers two portfolio modeling types:


### Static Models
A static model is a list of investment securities and their weights, plus a rebalance schedule. 

At the start date you input, the API will allocate $100,000 according to the model weights, then rebalance to the model weights at the instructed interval.

```
SPY : 50%
TLT : 50%
Rebalance Monthly
```

Above is an example of a 50%-50% portfolio of stocks and bonds rebalanced every month.


### Dynamic Model
A Dynamic model is multiple static models, stitched together at certain dates.


```
Portfolio 1
Jan 1, 2010 - Dec 31, 2010
SPY : 50%
TLT : 50%
Rebalance Monthly

Portfolio 2
Jan 1, 2011 - Dec 31, 2011
IVV : 50%
BND : 50%
Rebalance Quarterly

Portfolio 3
Jan 1, 2012 - Dec 31, 2021
GS : 50%
BAC : 50%
Rebalance Monthly
```

The above Dynamic model will trade Portfolio 1 in 2010, switch to portfolio 2 in 2011, then trade portfolio 3 from 2012 through 2021.


## Result Types

### Calculate
Calcualte end points return calcualted statistics and daily closing values for the models. 


* Total return
* Yearly Return
* Risk
* Drawdown
* Historical Holdings
* Historical change of price


### Reference
Reference end points return non statistical data about the model. This includes
* Sector weightings
* Region weightings
* Asset Class weightings
* Blended expense ratio
* And more
