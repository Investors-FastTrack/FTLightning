---
stoplight-id: kenui32q3tehl
---
# Interactive Investment Growth Calculator Example

FastTrack Market Data API makes it easy to compute and visualize investment growth over any date range. This example shows how to fetch cumulative percent-from-start data (`p_percents2`) from the **`/v1/stats/{ticker}`** endpoint with `chart=1&count=A`, convert those percentages into signed numbers (handling both `"12.34%"` and `"(5.67%)"` formats), and calculate how a \$10,000 investment evolves day-by-day.

> **Note for Developers:**  
> • We use `appid` and `token` HTTP headers for authentication.  
> • We request **all** data points with `count=A`.  
> • The code below is fully self-contained—drop it into your Stoplight.io markdown page and it will render an interactive calculator.

---

## Usage Instructions

1. **Copy** the markdown below into your Stoplight.io documentation page.  
2. **Replace** the placeholder `YOUR_APPID` and `YOUR_TOKEN` with your FastTrack credentials.  
3. **Save** and **preview**—you’ll see inputs for Ticker, Start/End dates, AppID, Token, and a **Recalculate** button.  
4. **Interact**: change values and watch the tables update in real time.

---

## Example Code

```html
<!--
  Interactive Investment Growth Calculator
  ----------------------------------------
  Fetches /v1/stats/{ticker}?chart=1&count=A,
  parses p_percents2, and displays growth of $10,000.
-->

<style>
  .calculator-form {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 12px;
    margin-bottom: 20px;
  }
  .calculator-form label {
    display: flex;
    flex-direction: column;
    font-weight: 600;
  }
  .calculator-form input,
  .calculator-form button {
    padding: 6px 8px;
    font-size: 1em;
  }
  .calculator-form button {
    grid-column: span 2;
    background: #007ACC;
    color: #fff;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  .calculator-form button:hover {
    background: #005A9E;
  }

  table {
    border-collapse: collapse;
    width: 100%;
    max-width: 700px;
    margin-bottom: 30px;
  }
  th, td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: center;
  }
  th {
    background: #f4f4f4;
    font-weight: 700;
  }
</style>

<div class="calculator-form">
  <label>
    Ticker
    <input type="text" id="tickerInput" value="AAPL">
  </label>
  <label>
    Start Date
    <input type="date" id="startInput" value="2020-01-01">
  </label>
  <label>
    End Date
    <input type="date" id="endInput" value="2021-01-01">
  </label>
  <label>
    AppID
    <input type="text" id="appidInput" placeholder="YOUR_APPID">
  </label>
  <label>
    Token
    <input type="text" id="tokenInput" placeholder="YOUR_TOKEN">
  </label>
  <button id="calcButton">Recalculate</button>
</div>

<table id="summary">
  <thead>
    <tr>
      <th>Ticker</th>
      <th>Start</th>
      <th>End</th>
      <th>Initial ($)</th>
      <th>Final ($)</th>
      <th>% Change</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<h2>Day-by-Day Growth of $10,000</h2>
<table id="allValues">
  <thead>
    <tr>
      <th>Day #</th>
      <th>% Change</th>
      <th>Value ($)</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script>
  /**
   * Fetches the cumulative percent-from-start array (p_percents2)
   * from /v1/stats/{ticker}?chart=1&count=A.
   */
  async function fetchPercentArray(ticker, start, end, appid, token) {
    const url = new URL(\`https://ftl.fasttrack.net/v1/stats/\${ticker}\`);
    url.searchParams.set('start', start);
    url.searchParams.set('end', end);
    url.searchParams.set('chart', '1');
    url.searchParams.set('count', 'A');

    const resp = await fetch(url, {
      headers: { 'appid': appid, 'token': token }
    });
    if (!resp.ok) throw new Error(\`API error: \${resp.statusText}\`);
    const data = await resp.json();
    return data.chartprices.p_percents2;  // ["12.34%", "(5.67%)", …]
  }

  /**
   * Normalizes values like "82.31%", "(10.1%)", or 12.34
   * into signed floats (negative if in parentheses).
   */
  function parsePct(val) {
    let s = String(val).trim();
    const neg = s.startsWith('(') && s.endsWith(')');
    s = s.replace(/[()%]/g, '').replace(',', '.');
    let num = parseFloat(s);
    return neg ? -num : num;
  }

  async function calculateAndDisplay() {
    const ticker = document.getElementById('tickerInput').value.trim();
    const start  = document.getElementById('startInput').value;
    const end    = document.getElementById('endInput').value;
    const appid  = document.getElementById('appidInput').value.trim();
    const token  = document.getElementById('tokenInput').value.trim();
    const INITIAL = 10000;

    if (!ticker || !start || !end || !appid || !token) {
      alert('Please fill in all fields.');
      return;
    }

    // Clear previous results
    document.querySelector('#summary tbody').innerHTML = '';
    document.querySelector('#allValues tbody').innerHTML = '';

    try {
      const arr = await fetchPercentArray(ticker, start, end, appid, token);

      // Summary row (last data point)
      const lastPct  = parsePct(arr[arr.length - 1]);
      const finalVal = INITIAL * (1 + lastPct / 100);
      document.querySelector('#summary tbody').innerHTML = \`
        <tr>
          <td>\${ticker}</td>
          <td>\${start}</td>
          <td>\${end}</td>
          <td>\${INITIAL.toLocaleString()}</td>
          <td>\${finalVal.toLocaleString(undefined,{minimumFractionDigits:2})}</td>
          <td>\${lastPct.toFixed(2)}%</td>
        </tr>\`;

      // Day-by-day details
      const allTbody = document.querySelector('#allValues tbody');
      arr.forEach((raw, i) => {
        const pct = parsePct(raw);
        const val = INITIAL * (1 + pct / 100);
        const row = document.createElement('tr');
        row.innerHTML = \`
          <td>\${i + 1}</td>
          <td>\${pct.toFixed(2)}%</td>
          <td>\${val.toLocaleString(undefined,{minimumFractionDigits:2})}</td>
        \`;
        allTbody.appendChild(row);
      });
    } catch (err) {
      alert('Error: ' + err.message);
    }
  }

  document.getElementById('calcButton')
          .addEventListener('click', calculateAndDisplay);
  window.addEventListener('DOMContentLoaded', calculateAndDisplay);
</script>
