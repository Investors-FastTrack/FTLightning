---
stoplight-id: nvqt80kb9zl7a
---
## Pricing

Simple, predictable tokens for stats, historical data, and models.

---

### Plans

| Plan       | Price / mo | Included Tokens | Seats |
| ---------- | ---------: | ---------------: | -----: |
| Free       | $0         | 1,000            | 1 |
| Individual | $29        | 25,000           | 1 |
| Pro        | $99        | 250,000          | 1 |
| Team       | $299       | 1,000,000        | 3 |
| Enterprise | Custom     | Custom           | Custom |

---

### Token Costs

| Endpoint                  | Base Cost       | Extra Periods         | Extra Data               | Portfolio |
|---------------------------|-----------------|-----------------------|--------------------------|-----------|
| `/stats/stats2`           | 1 per ticker    | +1 each               | —                        | +1 |
| `/stats/stats2/historical`| 1 per ticker    | +1 each frequency     | —                        | +1 |
| `/data/data2`             | 1 per ticker    | —                     | +1 per 1K points         | +1 |
| `/reference`              | 1 per ticker    | —                     | —                        | +1 |

**Portfolio** = Any request involving a Static, Dynamic, or Momentum model.

---

### Usage Examples

**Example 1:** S&P 500 snapshot stats (periods: 1m, 2m, 3m, 1y, 3y, 5y)  
- Endpoint: `/stats/stats2`  
- Base: 500 tokens (1 per ticker)  
- Extra periods: 5 × 500 = 2,500 tokens  
- **Total per day:** 3,000 tokens  
- **Total per month (30 days):** 90,000 tokens → Fits in **Pro** plan

---

**Example 2:** 80 tickers, daily raw data, ~1K points each  
- Endpoint: `/data/data2`  
- Base: 80 tokens  
- Extra data: 80 tokens (1 per 1K points)  
- **Total per day:** 160 tokens  
- **Total per month (30 days):** 4,800 tokens → Fits in **Individual** plan

---

**Example 3:** Static 60/40 portfolio snapshot  
- Endpoint: `/stats/stats2`  
- Base: 2 tokens (2 tickers)  
- Portfolio flag: +1 token  
- **Total per call:** 3 tokens → Minimal usage, fits in any plan

---

**Example 4:** Momentum model with universe of 50 tickers (stats snapshot)  
- Endpoint: `/stats/stats2`  
- Base: 50 tokens  
- Portfolio flag: +1 token  
- **Total per call:** 51 tokens

---

**Example 5:** Reference data for 45 tickers  
- Endpoint: `/reference`  
- Base: 45 tokens  
- **Total per call:** 45 tokens → Fits easily in any plan

---

**Example 6:** S&P 500, 10-year daily prices (data)  
- Endpoint: `/data/data2`  
- Base: 500 tokens  
- Extra data: e.g., 3,650 points (~10 years daily) ≈ 4 × 500 = 2,000 tokens  
- **Total per call:** ~2,500 tokens  
- Daily pulls would require **Team** or higher

---

