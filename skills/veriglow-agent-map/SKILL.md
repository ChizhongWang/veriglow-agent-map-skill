---
name: veriglow-agent-map
description: >
  Look up VeriGlow Agent Map for any website URL to discover its data functions,
  internal APIs, browser automation recipes, and agent reliability reports.
  Use when you need to extract structured data from a website, call a website's
  hidden API, or automate browser interactions with a web page.
license: MIT
compatibility: Works with Claude Code, OpenClaw, Cursor, and compatible AI agents
metadata:
  author: VeriGlow
  version: "0.1.0"
  openclaw:
    emoji: "🗺️"
    homepage: "https://veri-glow.com"
    requires:
      bins:
        - curl
---

# VeriGlow Agent Map

VeriGlow Agent Map is a registry of Agent-readable documentation for websites. Each "map" tells you exactly how to extract structured data from a specific web page — including hidden APIs, request parameters, response schemas, browser automation recipes, and real-world agent reports.

## When to Use This Skill

Use Agent Map when you need to:
- Get structured data from a website (financial data, government data, e-commerce, etc.)
- Discover hidden/internal APIs behind a web page
- Automate browser interactions with a specific site
- Check if other agents have successfully used a data source

## How to Query Agent Map

To look up a map for any URL, visit:

```
https://veri-glow.com/{target-url-without-protocol}
```

For example, to find the map for `https://www.sse.com.cn/market/bonddata/overview/day/`:

```
https://veri-glow.com/www.sse.com.cn/market/bonddata/overview/day/
```

You can also fetch the page content programmatically:

```bash
curl https://veri-glow.com/www.sse.com.cn/market/bonddata/overview/day/
```

## What a Map Contains

Each Agent Map page has three sections:

### 1. Available Data
Documents every data function the page exposes:
- **API endpoint** (method, URL, parameters)
- **Request example** (curl command you can run directly)
- **Response schema** (column names, types, examples)
- **Caveats** (rate limits, IP restrictions, data freshness)

### 2. Page Internals
For browser automation fallback:
- **JS controller objects** and trigger methods
- **DOM selectors** for data tables and input fields
- **Rendering method** (server-side vs client-side)
- **Auth and anti-scraping** status
- **Complete action steps** (Playwright/Puppeteer recipe)

### 3. Agent Reports
Real-world usage reports from agents who have called this data source:
- Success/failure status and response times
- Edge cases discovered (half-day trading, IP blocks, etc.)
- Recommended workarounds

## Example: SSE Bond Trading Data

Here is a complete example of using an Agent Map to get Shanghai Stock Exchange bond trading data.

### Direct API Call (Preferred)

```bash
curl "https://www.sse.com.cn/js/common/sseBond498Fixed.js?searchDate=2025-02-11"
```

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| searchDate | string | Yes | Query date in `YYYY-MM-DD` format |
| jsonCallBack | string | No | JSONP callback name. Omit for pure JSON. |

**Returns:** JSON with 17 rows × 4 columns:

| Column | Type | Example | Description |
|--------|------|---------|-------------|
| 债券品种 | string | 国债 | Bond type category |
| 成交笔数 | number | 15,234 | Number of trades |
| 成交面额(亿元) | number | 2,068.77 | Face value traded (100M RMB) |
| 成交金额(亿元) | number | 2,087.45 | Trading amount (100M RMB) |

**Bond types (17 categories):** 国债, 地方政府债, 企业债, 公司债, 可转换债, 可分离债, 资产支持证券, 国际机构债, 政策性金融债, 同业存单, and more.

**Caveats:**
- Non-trading days (weekends, holidays) return all-zero rows
- Data availability: T+1, updated ~1 hour after market close (15:30 CST)
- Overseas IP may get 403 — use a CN-based proxy if needed

### Browser Automation Fallback

If the direct API is blocked, use these action steps:

```javascript
// Step 1: Set date
document.querySelector('.js_date input').value = '2025-02-11'

// Step 2: Trigger query (wait 3s for AJAX response)
overviewDay.setOverviewDayParams()

// Step 3: Extract table
const table = document.querySelector('.table-responsive table')
const headers = [...table.querySelectorAll('thead th')].map(th => th.textContent.trim())
const rows = [...table.querySelectorAll('tbody tr')].map(tr =>
  [...tr.querySelectorAll('td')].map(td => td.textContent.trim())
)
```

## Best Practices

1. **Always prefer direct API calls** over browser automation — they are faster and more reliable
2. **Check the Agent Reports section** before calling a new data source — other agents may have documented edge cases
3. **Respect rate limits** — if a map documents rate limiting, throttle your requests accordingly
4. **Handle IP restrictions** — some Chinese government/financial data sources block overseas IPs; use a CN proxy when noted
5. **Verify data freshness** — check the "Data availability" note for each source (e.g., T+1 means data is from the previous trading day)

## Available Maps

Currently indexed data sources:

| Source | URL | Data |
|--------|-----|------|
| SSE Bond Trading (Daily) | `veri-glow.com/www.sse.com.cn/market/bonddata/overview/day/` | 17 bond categories × 4 trading metrics |

More maps are being added continuously. Visit [veri-glow.com](https://veri-glow.com) to search for available maps or request a new one.
