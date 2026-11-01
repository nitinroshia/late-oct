# Upstox Fundamentals API vs TickerTape Master Field List

**Purpose:** Establish whether Upstox provides sufficient raw data to build a TickerTape-style stock screener and financial analytics platform.  
**Documentation reviewed:** 1 November 2026, directly from the live Upstox Developer API pages.  
**TickerTape source list:** `stock_screener_fields_master_list.txt` (18 sections, 249 field entries, 248 unique names; "PE Ratio" appears twice).

This document contains analysis only. It contains no code and no recommendations.

## Contents

0. Scope, method and legend
1. Upstox API endpoints
2. Complete API field inventory
3. TickerTape vs Upstox mapping
4. Derivable metrics
5. Missing data fields
6. Final coverage summary

---

## 0. Scope, method and legend

### 0.1 Pages opened and verified

| Page | URL | Verification |
|---|---|---|
| Fundamentals (index) | https://upstox.com/developer/api-documentation/fundamentals | Opened. Lists exactly 8 endpoints. |
| Get Company Profile | https://upstox.com/developer/api-documentation/get-company-profile | Full schema and sample response read |
| Get Balance Sheet | https://upstox.com/developer/api-documentation/get-balance-sheet | Full schema and sample response read |
| Get Cash Flow | https://upstox.com/developer/api-documentation/get-cash-flow | Full schema and sample response read |
| Get Income Statement | https://upstox.com/developer/api-documentation/get-income-statement | Full schema and sample response read |
| Get Share Holdings | https://upstox.com/developer/api-documentation/get-share-holdings | Full schema and sample response read |
| Get Key Ratios | https://upstox.com/developer/api-documentation/get-key-ratios | Full schema and sample response read |
| Get Corporate Actions | https://upstox.com/developer/api-documentation/get-corporate-actions | Full schema and sample response read |
| Get Competitors | https://upstox.com/developer/api-documentation/get-competitors | Full schema and sample response read |
| Historical Candle Data V3 | https://upstox.com/developer/api-documentation/v3/get-historical-candle-data | Full schema read (supplementary, non-Fundamentals) |
| Full Market Quotes V3 | https://upstox.com/developer/api-documentation/get-full-market-quote-v3 | Full schema read (supplementary, non-Fundamentals) |
| Put/Call Option Chain | https://upstox.com/developer/api-documentation/get-pc-option-chain | Full schema read (supplementary, non-Fundamentals) |
| Historical Data, Market Quote and Option Chain index pages | see Upstox navigation | Opened to enumerate sub-endpoints |

### 0.2 Listed in Upstox navigation but NOT opened in this analysis

These could contain data relevant to TickerTape fields. Anything that depends on them is flagged **[U]** (unverified).

- Instruments (instrument master file), Expired Instruments, Backtesting
- Intraday Candle Data (V2 and V3), Historical Candle Data (V2), OHLC Quotes (V2 and V3), LTP Quotes (V2 and V3), Full Market Quotes (V2), Option Greeks
- Option Contracts (only its one-line listing description was seen)
- Market Information, News
- Websocket, Webhook, Rate Limits, Authentication

### 0.3 Legend

| Term | Meaning |
|---|---|
| **Direct** | A field in a verified Upstox response corresponds to the TickerTape field without calculation. |
| **Derivable** | Not returned as such, but computable from verified Upstox fields (the inputs are listed in Section 4). |
| **Not available** | No verified Upstox field, and not computable from verified Upstox fields. |
| Source family F | Fundamentals API only |
| Source family M | Needs market-data endpoints (Historical Candle, Full Quote, Option Chain), outside the Fundamentals group |
| Source family F+M | Needs both |
| Fidelity: Exact | Arithmetic on verified fields; no assumption beyond the stated definition |
| Fidelity: Exact (given ...) | Arithmetic is exact once the stated condition is met: a look-back window, TickerTape's unstated definition, an external rate, or sufficient history/contract data |
| Fidelity: Approx | Depends on a definition, convention or window the master list does not state |
| Fidelity: Approx-Low | Depends on the estimated share count (B1); can be materially off |
| **[L]** | Label or semantics ambiguous in the documentation |
| **[H]** | History depth not documented (samples show only 4 periods) |
| **[X]** | Needs an input Upstox does not supply (for example a risk-free rate) |
| **[U]** | Depends on a page not opened in this analysis |
| **[S]** | Depends on the estimated share count B1 |

Endpoint short codes used in tables:

| Code | Endpoint |
|---|---|
| PROF | Get Company Profile |
| BS | Get Balance Sheet |
| CF | Get Cash Flow |
| IS | Get Income Statement |
| SH | Get Share Holdings |
| KR | Get Key Ratios |
| CA | Get Corporate Actions |
| COMP | Get Competitors |
| HC | Historical Candle Data V3 |
| FQ | Full Market Quotes V3 |
| OC | Put/Call Option Chain |

---

## 1. Upstox API endpoints

### 1.1 Fundamentals endpoints (complete list: 8)

All eight are `GET` requests keyed by **ISIN**, under base URL `https://api.upstox.com/v2/fundamentals/{isin}/...`, authenticated with a Bearer access token. Example ISIN in the docs: `INE002A01018`.

| # | Code | Endpoint | Path | Query parameters | Frequency | Consolidated / Standalone | Units |
|---|---|---|---|---|---|---|---|
| 1 | PROF | Get Company Profile | `/profile` | none | Current snapshot (no period field) | Not applicable (no parameter) | Sector market cap in INR crore and USD (billion or million) |
| 2 | BS | Get Balance Sheet | `/balance-sheet` | `type` (consolidated default, standalone); `fs` (boolean) | Annual. Summary `history` is labelled by period (for example "Mar 2025"). `full_statement` is always annual. The response field `time_period` lists yearly or quarterly, but no query parameter to choose it is documented. | Selectable via `type`; default consolidated | INR crore |
| 3 | CF | Get Cash Flow | `/cash-flow` | `type`; `fs` | Annual. Same `time_period` caveat as the balance sheet. `full_statement` is always annual. | Selectable via `type`; default consolidated | INR crore |
| 4 | IS | Get Income Statement | `/income-statement` | `type`; `time_period` (yearly default, quarterly); `fs` | Yearly or quarterly for the 3-category summary. `full_statement` is always annual regardless of `time_period`. | Selectable via `type`; default consolidated | INR crore (EPS is per share) |
| 5 | SH | Get Share Holdings | `/share-holdings` | none | Quarterly (labels such as Mar 2026, Dec 2025) | Not specified (no parameter) | Percent of total shares |
| 6 | KR | Get Key Ratios | `/key-ratios` | none | Current values with no period field; refresh cadence not documented | Not specified (no parameter) | Strings; some carry a % sign |
| 7 | CA | Get Corporate Actions | `/corporate-actions` | none | Event-based (dated events) | Not applicable | Rupees per share (dividends); ratio strings |
| 8 | COMP | Get Competitors | `/competitors` | none | Current snapshot | Not applicable | Sector market cap in INR crore and USD |

**Common facts**

- Every response has `status` (`success` or `error`). The docs have an "Error codes" section, but its table was not present in the retrieved page content.
- Statements are requested by ISIN, so coverage is by listed company. ETF or index-fund coverage is not documented.
- Rate limits for the Fundamentals group were not reviewed [U]. Eight calls per ISIN are needed to pull everything.

### 1.2 Supplementary market-data endpoints verified (outside the Fundamentals group)

These were opened because many TickerTape fields (price, volume, technicals, F&O) can only come from market data.

| Code | Endpoint | Method and path | Key parameters | Frequency / granularity |
|---|---|---|---|---|
| HC | Historical Candle Data V3 | `GET /v3/historical-candle/{instrument_key}/{unit}/{interval}/{to_date}/{from_date}` | `unit`: minutes, hours, days, weeks, months; `interval`; dates as YYYY-MM-DD | Intraday to monthly candles (table below) |
| FQ | Full Market Quotes V3 | `GET /v3/market-quote/quotes?instrument_key=...` | Up to 500 comma-separated instrument keys | Exchange snapshot at request time |
| OC | Put/Call Option Chain | `GET /v2/option/chain?instrument_key=...&expiry_date=...` | Underlying key; expiry as a date or a keyword such as current_week or current_month. Not available for MCX. | Snapshot per expiry |

**Historical Candle V3 availability (as documented)**

| Unit | Intervals | History available from | Max records per request |
|---|---|---|---|
| minutes | 1 to 300 | January 2022 | 1 month (intervals 1 to 15); 1 quarter (above 15) |
| hours | 1 to 5 | January 2022 | 1 quarter |
| days | 1 | January 2000 | 1 decade |
| weeks | 1 | January 2000 | No limit |
| months | 1 | January 2000 | No limit |

The docs do not say whether candles are adjusted for splits, bonuses or dividends. India VIX (`NSE_INDEX` key) and global indices are called out as supported instruments.

---

## 2. Complete API field inventory

Field paths are shown as documented. `history` arrays are ordered most recent first. Row counts include envelope and nested fields.

### 2.1 Get Company Profile (PROF)

| Field path | Type | Description / allowed values | Notes |
|---|---|---|---|
| status | string | `success` or `error` |  |
| data.company_profile | string | Business description | Free text |
| data.sector | string | Sector the company belongs to | Sample: "Refineries" |
| data.sector_market_cap_inr | object | Total market cap of the sector in INR | Sector level, not company level |
| data.sector_market_cap_inr.value | number | Numeric value in crore |  |
| data.sector_market_cap_inr.unit | string | `crore` |  |
| data.sector_market_cap_inr.formatted | string | Human-readable, for example "1,942,866.05 Cr" |  |
| data.sector_market_cap_usd | object | Total market cap of the sector in USD |  |
| data.sector_market_cap_usd.value | number | Numeric value in the stated unit |  |
| data.sector_market_cap_usd.unit | string | `billion` or `million` |  |
| data.sector_market_cap_usd.formatted | string | For example "$215.87B" |  |

No company name, ticker, industry code, listing date, face value, share count or company market cap is returned.

### 2.2 Get Balance Sheet (BS)

| Field path | Type | Description / allowed values | Notes |
|---|---|---|---|
| status | string | `success` or `error` |  |
| data | object | Balance sheet for the requested type and period |  |
| data.type | string | `consolidated` or `standalone` | Echoes the `type` parameter |
| data.time_period | string | `yearly` or `quarterly` | No query parameter documented to select this |
| data.units_in | string | `crore` |  |
| data.history | array | Summary records, most recent first | Sample: 4 periods |
| data.history[].total_asset | number | Total assets (INR crore) |  |
| data.history[].total_liability | number | Total liabilities (INR crore) | Sample reconciles to Current + Non-Current Liabilities |
| data.history[].period | string | Period label, for example "Mar 2025" |  |
| data.full_statement | array | Line items; only when `fs=true`; always annual |  |
| data.full_statement[].particular | string | Line-item label | Documented as "e.g." examples; see line items below |
| data.full_statement[].history | array | Values across periods |  |
| data.full_statement[].history[].period | string | Period label |  |
| data.full_statement[].history[].value | number | Value (INR crore) |  |

**Balance sheet `particular` values seen in the sample response (8 line items)**

| # | Line item | Sample reconciliation (Mar 2025) |
|---|---|---|
| 1 | Non-Current Assets | 1,450,851 |
| 2 | Current Assets | 499,270 |
| 3 | Total Assets | = Non-Current + Current |
| 4 | Current Liabilities | 453,737 |
| 5 | Net Current Asset | = Current Assets minus Current Liabilities (45,533) |
| 6 | Non-Current Liabilities | 486,758 |
| 7 | Equity Capital | 1,009,626 (behaves as total equity; see [L]) |
| 8 | Total Equity & Liabilities | = Equity Capital + Current + Non-Current Liabilities = Total Assets |

### 2.3 Get Cash Flow (CF)

| Field path | Type | Description / allowed values | Notes |
|---|---|---|---|
| status | string | `success` or `error` |  |
| data | object | Cash-flow data |  |
| data.type | string | `consolidated` or `standalone` |  |
| data.time_period | string | `yearly` or `quarterly` | No query parameter documented to select this |
| data.units_in | string | `crore` |  |
| data.cash_flow | array | Category series, most recent first |  |
| data.cash_flow[].category | string | `operating`, `investing`, `financing` |  |
| data.cash_flow[].history | array | History for the category | Sample: 4 periods |
| data.cash_flow[].history[].value | number | Amount (INR crore); negative means outflow |  |
| data.cash_flow[].history[].period | string | Period label |  |
| data.cash_flow[].history[].change | string | Period-over-period change, for example "+12.54%" | Absent for the oldest period |
| data.full_statement | array | Line items; only when `fs=true`; always annual |  |
| data.full_statement[].particular | string | Line-item label | See line items below |
| data.full_statement[].history | array | Values across periods |  |
| data.full_statement[].history[].period | string | Period label |  |
| data.full_statement[].history[].value | number | Value (INR crore) |  |

**Cash flow `particular` values seen in the sample response (11 line items)**

| # | Line item | Note |
|---|---|---|
| 1 | Profit before tax | Starting point of the indirect method |
| 2 | Income before WC changes |  |
| 3 | Change in Assets | Working-capital component |
| 4 | Change in Liabilities | Working-capital component |
| 5 | Change in WC | Net working-capital change |
| 6 | Cash flow from Operations | Equals the `operating` category |
| 7 | Cash flow from Investing | Equals the `investing` category |
| 8 | Cash flow from Financing | Equals the `financing` category |
| 9 | Total Cash Flow | Sample: equals CFO + CFI + CFF, and End minus Start cash |
| 10 | Cash (Start of the year) |  |
| 11 | Cash (End of the year) |  |

No capital-expenditure, dividend-paid, depreciation or interest-paid lines appear.

### 2.4 Get Income Statement (IS)

| Field path | Type | Description / allowed values | Notes |
|---|---|---|---|
| status | string | `success` or `error` |  |
| data | object | Income-statement data |  |
| data.type | string | `consolidated` or `standalone` |  |
| data.time_period | string | `yearly` or `quarterly` | Selectable via the `time_period` query parameter |
| data.units_in | string | `crore` | EPS values are per share |
| data.income_statement | array | Category series, most recent first |  |
| data.income_statement[].category | string | `revenue`, `operating_profit`, `net_profit` | `operating_profit` is not defined further |
| data.income_statement[].history | array | History for the category | Sample: 4 periods |
| data.income_statement[].history[].value | number | Value (INR crore); negative means loss |  |
| data.income_statement[].history[].period | string | Period label |  |
| data.income_statement[].history[].change | string | Period-over-period change, for example "+10.53%" | Absent for the oldest period |
| data.full_statement | array | Line items; only when `fs=true`; annual regardless of `time_period` |  |
| data.full_statement[].particular | string | Line-item label | See line items below |
| data.full_statement[].history | array | Values across periods |  |
| data.full_statement[].history[].period | string | Period label |  |
| data.full_statement[].history[].value | number | Value (INR crore, or per share for EPS) |  |

**Income statement `particular` values seen in the sample response (9 line items)**

| # | Line item | Note |
|---|---|---|
| 1 | Revenue | Operating revenue |
| 2 | Other Income |  |
| 3 | Total Revenue | Revenue + Other Income |
| 4 | Total Expenses | Single aggregate; sample: Total Revenue minus Total Expenses = PBT, so it includes finance costs and D&A |
| 5 | Profit Before Tax |  |
| 6 | Tax | Sample: PBT minus Tax = PAT |
| 7 | Profit After Tax |  |
| 8 | EPS - Basic | Per share |
| 9 | EPS - Diluted | Per share |

The three summary categories (`revenue`, `operating_profit`, `net_profit`) are the only series available quarterly.

### 2.5 Get Share Holdings (SH)

| Field path | Type | Description / allowed values | Notes |
|---|---|---|---|
| status | string | `success` or `error` |  |
| data | array | One entry per shareholder category |  |
| data[].category | string | `promoters`, `fii`, `other_dii`, `mutual_funds`, `retail_and_other` | Sample: five categories sum to about 100% |
| data[].history | array | Quarterly history | Sample: 4 quarters |
| data[].history[].period | string | Quarter label, for example "Mar 2026" |  |
| data[].history[].value | number | Percent of total shares held |  |

### 2.6 Get Key Ratios (KR)

| Field path | Type | Description / allowed values | Notes |
|---|---|---|---|
| status | string | `success` or `error` |  |
| data | array | List of ratio entries |  |
| data[].name | string | `P/E`, `P/B`, `ROA`, `ROE`, `ROCE`, `EV/EBITDA` | Exactly six ratios documented |
| data[].company_value | string | Current ratio value for the company | String type; ROA, ROE, ROCE carry a % sign |
| data[].sector_value | string | Sector benchmark for the same ratio | Benchmark method not documented |

Documented definitions: P/E = price / EPS; P/B = price / book value per share; ROA = net income / total assets; ROE = net income / shareholders' equity; ROCE = EBIT / capital employed; EV/EBITDA = enterprise value / EBITDA. No period, date or consolidated/standalone flag accompanies the values.

### 2.7 Get Corporate Actions (CA)

| Field path | Type | Description / allowed values | Notes |
|---|---|---|---|
| status | string | `success` or `error` |  |
| data | array | List of corporate-action events |  |
| data[].name | string | Type of action: Dividend, Bonus, Split, Rights | Only a Dividend example is shown |
| data[].expiry_date | string | Ex-date or effective date, for example "14 Aug 2025" | Date is a text string |
| data[].amount | number | Monetary amount (dividends) |  |
| data[].ratio | string | Ratio for bonus or split, for example "1:1"; null otherwise | No bonus or split sample shown |
| data[].event_details | array | Key-value detail pairs |  |
| data[].event_details[].name | string | Label of the detail | See labels below |
| data[].event_details[].value | string | Value of the detail | Always a string |

**`event_details[].name` labels seen in the Dividend sample (7):** Announcement date, Ex dividend date, Record date, Dividend type, Amount, Dividend %, Details. Labels for Bonus, Split and Rights events are not documented [U].

### 2.8 Get Competitors (COMP)

| Field path | Type | Description / allowed values | Notes |
|---|---|---|---|
| status | string | `success` or `error` |  |
| data | array | List of competitor profiles |  |
| data[].instrument_key | string | Exchange and ISIN joined by a pipe, for example NSE_EQ then INE242A01010 | Usable with market-data endpoints |
| data[].company_profile | string | Business description |  |
| data[].sector | string | Sector of the competitor |  |
| data[].sector_market_cap_inr | object | Sector market cap in INR |  |
| data[].sector_market_cap_inr.value | number | Value in crore |  |
| data[].sector_market_cap_inr.unit | string | `crore` |  |
| data[].sector_market_cap_inr.formatted | string | Formatted string |  |
| data[].sector_market_cap_usd | object | Sector market cap in USD |  |
| data[].sector_market_cap_usd.value | number | Value in stated unit |  |
| data[].sector_market_cap_usd.unit | string | `billion` or `million` |  |
| data[].sector_market_cap_usd.formatted | string | Formatted string |  |

Returns peer identity only. No peer financials, prices or ratios.

### 2.9 Fundamentals field-count summary

| Endpoint | Documented field rows (incl. nested) | Distinct data series / line items |
|---|---|---|
| PROF | 11 | 3 data elements (description, sector, sector market cap in two currencies) |
| BS | 14 | 2 summary series + 8 line items |
| CF | 16 | 3 category series + 11 line items |
| IS | 16 | 3 summary categories + 9 line items |
| SH | 6 | 5 shareholder categories |
| KR | 5 | 6 ratios x (company value, sector value) |
| CA | 9 | 4 action types; 7 detail labels seen for dividends |
| COMP | 13 | Peer list with 3 elements each |
| **Total** | **90** |  |

### 2.10 Supplementary market-data field inventory

**HC response (`data.candles`, each candle is an array)**

| Position | Field | Type | Description |
|---|---|---|---|
| candle[0] | Timestamp | string (ISO 8601 with offset) | Start time of the candle |
| candle[1] | Open | number | Opening price |
| candle[2] | High | number | Highest price |
| candle[3] | Low | number | Lowest price |
| candle[4] | Close | number | Closing price |
| candle[5] | Volume | number | Traded quantity in the period |
| candle[6] | Open Interest | number | Outstanding derivative contracts (0 for cash equities) |

**FQ response (`data`, keyed by exchange and trading symbol, for example NSE_EQ:NHPC)**

| Field | Type | Description |
|---|---|---|
| ohlc.open / high / low / close | number | Session OHLC |
| ohlc.volume | integer | Candle volume |
| ohlc.ts | integer | Candle start, ms |
| depth.buy[] / depth.sell[] | object[] | Top 5 bids and asks, each with quantity, price, orders |
| timestamp | string | Response generation time |
| instrument_token | string | Instrument key |
| symbol | string | Trading symbol |
| last_price | number | Last traded price |
| volume | integer | Volume traded today |
| average_price | number | "Average price" (not further defined) |
| oi | number | Open interest (F&O only) |
| net_change | number | Absolute change from previous close |
| total_buy_quantity / total_sell_quantity | number | Aggregate bid and ask quantity |
| lower_circuit_limit / upper_circuit_limit | number | Circuit limits |
| last_trade_time | string | Last trade time, ms |
| oi_day_high / oi_day_low | number | Day range of open interest |
| prev_close_price | number | Previous session close |
| year_high / year_low | number | Trailing-year high and low |
| previous_oi | number | Previous session open interest (F&O only) |
| indicative_equilibrium_price / _quantity | number / integer | Closing-auction indicators |
| indicative_imbalance_quantity_total / _market | integer | Closing-auction imbalance |
| reference_price | number | Base price for price bands |
| cas_eligible | boolean | Closing-auction eligibility |

No market-cap, shares-outstanding, face-value, free-float or fundamental fields appear in FQ.

**OC response (`data[]`, one row per strike)**

| Field | Type | Description |
|---|---|---|
| expiry | string | Expiry date YYYY-MM-DD |
| pcr | number | Put-call ratio for the strike row (sample equals put oi / call oi of that strike) |
| strike_price | number | Strike |
| underlying_key | string | Underlying instrument key |
| underlying_spot_price | number | Underlying spot |
| call_options.instrument_key / put_options.instrument_key | string | Contract key |
| *.market_data.ltp, close_price, volume, oi, prev_oi | number | Price, volume, current and previous-session open interest |
| *.market_data.bid_price, bid_qty, ask_price, ask_qty | number | Top of book |
| *.option_greeks.vega, theta, gamma, delta, iv, pop | number | Greeks, implied volatility, probability of profit |

---

## 3. TickerTape vs Upstox mapping

### 3.1 Forward map: every TickerTape field classified (249 entries)

Class: **D** = Direct, **R** = Derivable (see the ID in Section 4), **N** = Not available. Fam = source family (F, M, F+M).

#### 3.1.1 ETF Metrics  (D 2, R 1, N 3)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 1 | Name | N | - | None | No company or instrument name field in any reviewed endpoint (profile and competitors return a description, not a name). Instruments file not reviewed [U]. |
| 2 | Ticker | D | M | FQ `symbol` (trading symbol); `instrument_token` carries exchange and ISIN | Market-data endpoint, not Fundamentals. |
| 3 | Sub-Sector | D | F | PROF `data.sector` | Single-level label (sample: "Refineries"). Taxonomy may differ from TickerTape sub-sector [L]. |
| 4 | Market Capital | R | F+M | Derived, see R-001 | Approx-Low. No company market-cap field exists; only sector market cap (PROF, COMP). Depends on B1 [S]. ETF coverage of Fundamentals is unspecified. |
| 5 | Expense Ratio | N | - | None | No ETF-specific data in any reviewed endpoint. |
| 6 | Tracking Error | N | - | None | Requires ETF NAV versus index; NAV not provided. |

#### 3.1.2 Cash Flow & Equity Metrics  (D 8, R 3, N 7)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 7 | Total Equity | D | F | BS `full_statement[particular="Equity Capital"]` | Annual only (needs fs=true). Label reads "Equity Capital" but sample reconciles to total equity (Equity + Current + Non-Current Liabilities = Total Equity & Liabilities) [L]. Minority-interest treatment undocumented. |
| 8 | Common Shares Outstanding | R | F | Derived, see R-002 | Approx-Low. Yields a weighted-average basic share count at best. Sample: 80,787 / 51.47 = approx 1,570 Cr; PAT may include minority interest, which would overstate shares [L]. EPS series shows a discontinuity in the sample (98.59 to 51.45). |
| 9 | Change in Working Capital | D | F | CF `full_statement[particular="Change in WC"]` | Annual only. |
| 10 | Operating Cash Flow | D | F | CF `cash_flow[category=operating].history[].value`; also `full_statement["Cash flow from Operations"]` | Annual. |
| 11 | Capital Expenditure | N | - | None | No capex line; investing cash flow is a single aggregate. |
| 12 | Investing Cash Flow | D | F | CF `cash_flow[category=investing]`; `full_statement["Cash flow from Investing"]` | Annual. |
| 13 | Total Cash Dividend Paid | R | F | Derived, see R-003 | Approx-Low. No cash-dividend line in the cash-flow statement. Declared vs paid timing; depends on B1 [S]. |
| 14 | Financing Cash Flow | D | F | CF `cash_flow[category=financing]`; `full_statement["Cash flow from Financing"]` | Annual. |
| 15 | Net Change in Cash | D | F | CF `full_statement[particular="Total Cash Flow"]` | Sample reconciles: CFO + CFI + CFF = Total Cash Flow = End minus Start cash. |
| 16 | Non Current Assets | D | F | BS `full_statement[particular="Non-Current Assets"]` | Annual only. |
| 17 | Non Current Liabilities | D | F | BS `full_statement[particular="Non-Current Liabilities"]` | Annual only. |
| 18 | Free Cash Flow | N | - | None | Requires capex, which is not provided. |
| 19 | Deferred Tax Assets (Net) | N | - | None | Not a line in the 8-item balance-sheet summary. |
| 20 | Book Value | R | F | Derived, see R-004 | Exact (aggregate); Approx (per share). Master list does not state aggregate vs per-share. KR values are strings [L]. |
| 21 | Other Current Assets | N | - | None | Only the `Current Assets` total is provided. |
| 22 | Other Assets | N | - | None | Only Non-Current and Current totals are provided. |
| 23 | Other Current Liabilities | N | - | None | Only the `Current Liabilities` total is provided. |
| 24 | Other Liabilities | N | - | None | Only Non-Current and Current totals are provided. |

#### 3.1.3 Balance Sheet Metrics  (D 5, R 0, N 15)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 25 | Cash and Equivalent | D | F | CF `full_statement[particular="Cash (End of the year)"]` | Cash-flow-statement closing balance, not a balance-sheet line (BS has no cash item). Definition may differ from balance-sheet cash [L]. |
| 26 | Total Receivables | N | - | None | Not provided. |
| 27 | Total Inventory | N | - | None | Not provided. |
| 28 | Total Current Assets | D | F | BS `full_statement[particular="Current Assets"]` | Annual only. |
| 29 | Loans & Advances | N | - | None | Not provided; no bank/NBFC-specific statement template is documented. |
| 30 | Net Property, Plant & Equipment | N | - | None | Not provided. |
| 31 | Goodwill & Intangibles | N | - | None | Not provided. |
| 32 | Long Term Investments | N | - | None | Not provided. |
| 33 | Total Assets | D | F | BS `history[].total_asset`; `full_statement["Total Assets"]` | Summary `history` is the headline series; `full_statement` is annual. |
| 34 | Accounts Payable | N | - | None | Not provided. |
| 35 | Total Deposits – Banks | N | - | None | Not provided; no bank-specific template documented. |
| 36 | Total Current Liabilities | D | F | BS `full_statement[particular="Current Liabilities"]` | Annual only. |
| 37 | Long Term Debt | N | - | None | No borrowings line. |
| 38 | Total Debt | N | - | None | No borrowings line; `total_liability` is all liabilities, not debt. |
| 39 | Deferred Tax Liabilities (Net) | N | - | None | Not provided. |
| 40 | Minority Interest | N | - | None | Not provided as a separate line. |
| 41 | Total Liabilities | D | F | BS `history[].total_liability` | Sample reconciles: equals Current + Non-Current Liabilities and Total Assets minus `Equity Capital`. |
| 42 | Share Capital | N | - | None | Not provided. "Equity Capital" appears to be total equity, not paid-up capital [L]. |
| 43 | Additional Paid–in Capital | N | - | None | Not provided. |
| 44 | Reserves & Surplus | N | - | None | Not provided; cannot be backed out without Share Capital. |

#### 3.1.4 Income Statement & Earnings Metrics  (D 8, R 2, N 13)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 45 | Total Revenue | D | F | IS `full_statement[particular="Total Revenue"]` | Sample: Revenue + Other Income. The summary `revenue` category equals Total Revenue (not `Revenue`) in the sample [L]. |
| 46 | EBITDA | N | - | None | No D&A or finance-cost lines. The `operating_profit` category has an unconfirmed definition (sample equals PBT) [L]. |
| 47 | Depreciation & Amortization | N | - | None | Not provided. |
| 48 | PBIT | N | - | None | No interest line, so PBT cannot be bridged to PBIT. |
| 49 | Interest & Other Items | N | - | None | Not provided; `Total Expenses` is a single aggregate (sample: Total Revenue minus Total Expenses = PBT). |
| 50 | PBT | D | F | IS `full_statement[particular="Profit Before Tax"]` | Annual only. |
| 51 | Taxes & Other Items | D | F | IS `full_statement[particular="Tax"]` | Sample reconciles: PBT minus Tax = PAT. "Other items" not separately reported. |
| 52 | Net Income | D | F | IS `full_statement["Profit After Tax"]`; IS `income_statement[category=net_profit]` | Whether PAT is before or after minority interest is not documented [L]. |
| 53 | Earnings Per Share | D | F | IS `full_statement["EPS - Basic"]` and `["EPS - Diluted"]` | Annual only. Per-share value (the `units_in: crore` flag applies to monetary totals). |
| 54 | Dividend Per Share | R | F | Derived, see R-005 | Exact (if event history complete). History depth of corporate actions is undocumented [H]. |
| 55 | Payout Ratio | R | F | Derived, see R-006 | Exact. Inherits B4 completeness. |
| 56 | Raw Materials | N | - | None | Expense breakdown not provided. |
| 57 | Power & Fuel Cost | N | - | None | Expense breakdown not provided. |
| 58 | Employee Cost | N | - | None | Expense breakdown not provided. |
| 59 | Selling & Administrative Expenses | N | - | None | Expense breakdown not provided. |
| 60 | Operating & Other expenses | N | - | None | Only the aggregate `Total Expenses` is provided. |
| 61 | Total revenue (Q) | D | F | IS with `time_period=quarterly`: `income_statement[category=revenue]` | Quarterly access is documented for the income statement only. Same label ambiguity as annual [L]. |
| 62 | Net Income (Q) | D | F | IS with `time_period=quarterly`: `income_statement[category=net_profit]` | Quarterly. |
| 63 | EBITDA (Q) | N | - | None | No D&A or finance-cost data; quarterly summary has only revenue, operating_profit, net_profit. |
| 64 | EPS (Q) | N | - | None | Quarterly summary has no EPS; `full_statement` is annual-only. |
| 65 | Operating and Other Expenses (Q) | N | - | None | Not provided. |
| 66 | PBIT (Q) | N | - | None | No interest data. |
| 67 | PBT (Q) | D | F | IS quarterly: `income_statement[category=operating_profit]` (unconfirmed) | UNCONFIRMED mapping. In the annual sample `operating_profit` equals Profit Before Tax for three periods, but the docs only call it "Operating profit" [L]. |

#### 3.1.5 Technical Indicators  (D 0, R 14, N 0)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 68 | Stochastic %K | R | M | Derived, see R-007 | Exact. Look-back N not given in master list. |
| 69 | Stochastic %D | R | M | Derived, see R-008 | Exact. Smoothing length not given. |
| 70 | 1W Change in On Balance Volume | R | M | Derived, see R-009 | Exact. OBV depends on the series start point. |
| 71 | 1W Change in AD Line | R | M | Derived, see R-010 | Exact |
| 72 | Super Trend | R | M | Derived, see R-011 | Exact. ATR period and multiplier not given. |
| 73 | 10D EMA | R | M | Derived, see R-012 | Exact. Candle adjustment for splits/bonus is undocumented. |
| 74 | 10D SMA | R | M | Derived, see R-013 | Exact |
| 75 | 50D EMA | R | M | Derived, see R-014 | Exact |
| 76 | 50D SMA | R | M | Derived, see R-015 | Exact |
| 77 | 100D EMA | R | M | Derived, see R-016 | Exact |
| 78 | 200D EMA | R | M | Derived, see R-017 | Exact |
| 79 | 100D SMA | R | M | Derived, see R-018 | Exact |
| 80 | 200D SMA | R | M | Derived, see R-019 | Exact |
| 81 | 1Y Max Loss | R | M | Derived, see R-020 | Approx. Definition not stated in master list [L]. |

#### 3.1.6 Price, Momentum & Risk Metrics  (D 0, R 19, N 0)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 82 | VWAP | R | M | Derived, see R-021 | Exact (intraday) / Approx (proxy). FQ describes `average_price` only as "Average price" [L]. Averaging window in master list not given. |
| 83 | % Price above 1M SMA | R | M | Derived, see R-022 | Exact |
| 84 | % Price above 1Y SMA | R | M | Derived, see R-023 | Exact |
| 85 | % Price above 1M EMA | R | M | Derived, see R-024 | Exact |
| 86 | Beta | R | M | Derived, see R-025 | Exact (given window). Window and return frequency not given. Index candles via HC are implied (India VIX and global indices are called out) but a Nifty 50 historical example is not shown; FQ and OC show `NSE_INDEX` Nifty 50 [U]. |
| 87 | Sharpe Ratio | R | M | Derived, see R-026 | Exact (given rf). Risk-free rate is not supplied by Upstox [X]. |
| 88 | Alpha | R | M | Derived, see R-027 | Exact (given rf). Risk-free rate not supplied [X]. |
| 89 | Relative Volume | R | M | Derived, see R-028 | Exact. Look-back not given. |
| 90 | Volatility | R | M | Derived, see R-029 | Exact (given window). Window and annualisation not given. |
| 91 | Volatility vs Nifty | R | M | Derived, see R-030 | Exact (given definition). Ratio vs difference not given [L]. |
| 92 | RSI – 14D | R | M | Derived, see R-031 | Exact |
| 93 | RSI Exponential – 14D | R | M | Derived, see R-032 | Exact |
| 94 | ADX Rating – Trend Strength | R | M | Derived, see R-033 | Exact. "Rating" bucketing rule not given [L]. |
| 95 | MACD Line 1 – Trend Indicator | R | M | Derived, see R-034 | Exact. Standard parameters assumed. |
| 96 | MACD Line 2 – Signal Line Comp | R | M | Derived, see R-035 | Exact |
| 97 | % From Upper Bollinger Band | R | M | Derived, see R-036 | Exact. Denominator (price vs band) not given. |
| 98 | % From Lower Bollinger Band | R | M | Derived, see R-037 | Exact |
| 99 | % From Parabolic SAR | R | M | Derived, see R-038 | Exact. Acceleration parameters not given. |
| 100 | William %R | R | M | Derived, see R-039 | Exact |

#### 3.1.7 Options & Open Interest Metrics  (D 0, R 11, N 0)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 101 | 1D Change in Call OI | R | M | Derived, see R-040 | Exact. Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| 102 | 1D Change in Put OI | R | M | Derived, see R-041 | Exact. Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| 103 | 1W Change in Call OI | R | M | Derived, see R-042 | Exact (if history reachable). `prev_oi` covers only the previous session. Contract-level history for expired contracts would depend on the Expired Instruments API (not reviewed) [U]. |
| 104 | 1W Change in Put OI | R | M | Derived, see R-043 | Exact (if history reachable). Same constraint as call side [U]. |
| 105 | Highest Call OI Strike | R | M | Derived, see R-044 | Exact. Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| 106 | Highest Put OI Strike | R | M | Derived, see R-045 | Exact. Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| 107 | Highest 1D OI Change CE Strike | R | M | Derived, see R-046 | Exact. Absolute vs percentage change not given [L]. |
| 108 | Highest 1D OI Change PE Strike | R | M | Derived, see R-047 | Exact. Absolute vs percentage change not given [L]. |
| 109 | Highest 1W OI Change CE Strike | R | M | Derived, see R-048 | Exact (if history reachable). Needs contract-level history for every strike [U]. |
| 110 | Highest 1W OI Change PE Strike | R | M | Derived, see R-049 | Exact (if history reachable). Needs contract-level history for every strike [U]. |
| 111 | Put Call Ratio | R | M | Derived, see R-050 | Exact. OC `pcr` is returned per strike row (sample: put oi / call oi for that strike), not as an aggregate. |

#### 3.1.8 Futures & Derivatives Metrics  (D 3, R 13, N 1)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 112 | Fair Value | R | M | Derived, see R-051 | Exact (given rate). Financing rate not supplied by Upstox [X]. Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| 113 | Future Close Price | D | M | HC `close` or FQ `ohlc.close` for the futures instrument | Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| 114 | Lot Size | N | - | None | Not present in any reviewed page; instrument master file not reviewed [U]. |
| 115 | Future Open Interest | D | M | FQ `oi` (F&O only); HC candle `[6]` | Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| 116 | 1D Change in Future OI | R | M | Derived, see R-052 | Exact. Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| 117 | 1W Change in Future OI | R | M | Derived, see R-053 | Exact. Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| 118 | Future Volume | D | M | FQ `volume`; HC candle `[5]` | Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| 119 | 1D Change in Future Volume | R | M | Derived, see R-054 | Exact. Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| 120 | 1W Change in Future Volume | R | M | Derived, see R-055 | Exact. Point vs aggregate comparison not given [L]. Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| 121 | Basis | R | M | Derived, see R-056 | Exact. Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| 122 | Fair Value Spread | R | M | Derived, see R-057 | Exact (given rate). Financing rate external [X]. |
| 123 | Cash & Carry Profit | R | M | Derived, see R-058 | Approx. Cost assumptions are not in Upstox data [X]. |
| 124 | Rollover Cost | R | M | Derived, see R-059 | Exact. Needs both contract keys [U]. |
| 125 | Percentage Rollover | R | M | Derived, see R-060 | Exact (given definition). Definition not stated in master list [L]. |
| 126 | Calendar Spread | R | M | Derived, see R-061 | Exact. Needs both contract keys [U]. |
| 127 | Call Open Interest | R | M | Derived, see R-062 | Exact. Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| 128 | Put Open Interest | R | M | Derived, see R-063 | Exact. Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |

#### 3.1.9 Valuation & Fundamental Scores  (D 1, R 0, N 5)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 129 | PE Ratio | D | F | KR `data[name="P/E"].company_value` | String value; current snapshot only (no history). Basis (TTM vs annual, price date) is not documented [L]. Duplicate of Valuation Ratios entry. |
| 130 | Value Momentum Rank | N | - | None | Proprietary ranking; no Upstox equivalent. |
| 131 | Price Momentum Rank | N | - | None | Proprietary ranking; no Upstox equivalent. |
| 132 | Earnings Quality Rank | N | - | None | Proprietary ranking; no Upstox equivalent. |
| 133 | Price to Intrinsic Value Rank | N | - | None | Proprietary ranking; intrinsic value needs forecasts that are not provided. |
| 134 | Fundamental Score | N | - | None | Proprietary score; no Upstox equivalent. |

#### 3.1.10 Insider Trading & Shareholder Activity  (D 0, R 0, N 7)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 135 | Insider Trades - 1M Cumulative | N | - | None | No insider-trade data in any reviewed endpoint. Market Information and News pages not reviewed [U]. |
| 136 | Insider Trades - 3M Cumulative | N | - | None | No insider-trade data in any reviewed endpoint. Market Information and News pages not reviewed [U]. |
| 137 | Insider Trades - 6M Cumulative | N | - | None | No insider-trade data in any reviewed endpoint. Market Information and News pages not reviewed [U]. |
| 138 | Bulk Deals - 1M Cumulative | N | - | None | No bulk-deal data in any reviewed endpoint. Market Information and News pages not reviewed [U]. |
| 139 | Bulk Deals - 3M Cumulative | N | - | None | No bulk-deal data in any reviewed endpoint. Market Information and News pages not reviewed [U]. |
| 140 | Bulk Deals - 6M Cumulative | N | - | None | No bulk-deal data in any reviewed endpoint. Market Information and News pages not reviewed [U]. |
| 141 | No. of Shareholders | N | - | None | SH returns percentages only, not holder counts. |

#### 3.1.11 Institutional & Ownership Metrics  (D 4, R 9, N 2)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 142 | Mutual Fund Holding | D | F | SH `category=mutual_funds` | Percent of total shares, quarterly. |
| 143 | MF Holding Change – 3M | R | F | Derived, see R-064 | Exact. Percentage-point change. Needs 2 quarters. |
| 144 | MF Holding Change – 6M | R | F | Derived, see R-065 | Exact. Needs 3 quarters. |
| 145 | Promoter Holding | D | F | SH `category=promoters` | Percent of total shares, quarterly. |
| 146 | Promoter Holding Change – 3M | R | F | Derived, see R-066 | Exact |
| 147 | Promoter Holding Change – 6M | R | F | Derived, see R-067 | Exact |
| 148 | Pledged Promoter Holdings | N | - | None | No pledge data in SH. |
| 149 | Domestic Institutional Holding | R | F | Derived, see R-068 | Exact (given exclusivity). Sample's five categories sum to about 100 (100.01), implying mutually exclusive buckets. Composition of `other_dii` is undocumented [L]. |
| 150 | DII Holding Change – 3M | R | F | Derived, see R-069 | Exact (given exclusivity) |
| 151 | DII Holding Change – 6M | R | F | Derived, see R-070 | Exact (given exclusivity) |
| 152 | Foreign Institutional Holding | D | F | SH `category=fii` | Percent of total shares, quarterly. |
| 153 | FII Holding Change – 3M | R | F | Derived, see R-071 | Exact |
| 154 | FII Holding Change – 6M | R | F | Derived, see R-072 | Exact |
| 155 | Insurance Firms Holding | N | - | None | Not reported separately; any insurer holding is inside `other_dii` (composition undocumented). |
| 156 | Retail Investor Holding | D | F | SH `category=retail_and_other` | Combined "retail and other" bucket; narrower retail definition may differ [L]. |

#### 3.1.12 Analyst Recommendations  (D 0, R 0, N 6)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 157 | Percentage Buy Reco’s | N | - | None | No analyst or consensus data in any of the 8 Fundamentals endpoints. |
| 158 | Percentage Upside | N | - | None | No analyst or consensus data in any of the 8 Fundamentals endpoints. |
| 159 | No. of analysts with buy reco | N | - | None | No analyst or consensus data in any of the 8 Fundamentals endpoints. |
| 160 | Percentage Hold Reco's | N | - | None | No analyst or consensus data in any of the 8 Fundamentals endpoints. |
| 161 | Percentage Sell Reco's | N | - | None | No analyst or consensus data in any of the 8 Fundamentals endpoints. |
| 162 | Total no. of analysts | N | - | None | No analyst or consensus data in any of the 8 Fundamentals endpoints. |

#### 3.1.13 Sub-Sector Valuation Comparison  (D 2, R 3, N 3)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 163 | PE Premium vs Sub-sector | R | F | Derived, see R-073 | Exact (on given values). Sector benchmark method (mean/median, constituent set) undocumented. Upstox `sector` may not match TickerTape sub-sector [L]. |
| 164 | PB Premium vs Sub-sector | R | F | Derived, see R-074 | Exact (on given values). Sector benchmark method (mean/median, constituent set) undocumented. Upstox `sector` may not match TickerTape sub-sector [L]. |
| 165 | PS Premium vs Sub-sector | N | - | None | KR has no P/S ratio and no sector P/S benchmark. |
| 166 | Dividend Yield vs Sub-sector | N | - | None | No sector dividend-yield benchmark. |
| 167 | Sector PE | D | F | KR P/E `sector_value` | String value. Sector benchmark method (mean/median, constituent set) undocumented. Upstox `sector` may not match TickerTape sub-sector [L]. |
| 168 | Sector PB | D | F | KR P/B `sector_value` | String value. Sector benchmark method (mean/median, constituent set) undocumented. Upstox `sector` may not match TickerTape sub-sector [L]. |
| 169 | Sector Dividend Yield | N | - | None | No sector dividend-yield benchmark; COMP gives a peer list but not a full sector constituent list. |
| 170 | TTM PE Ratio | R | F+M | Derived, see R-075 | Approx-Low. Alternative is KR P/E if its basis is TTM (undocumented). Depends on B1 [S]. |

#### 3.1.14 Valuation Ratios  (D 3, R 6, N 10)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 171 | PE Ratio | D | F | KR `data[name="P/E"].company_value` | Duplicate of entry in Valuation & Fundamental Scores. String value; basis undocumented [L]. |
| 172 | Forward PE Ratio | N | - | None | Requires forward EPS estimates; none provided. |
| 173 | PE Premium vs Sector | R | F | Derived, see R-076 | Exact (on given values). Sector benchmark method (mean/median, constituent set) undocumented. Upstox `sector` may not match TickerTape sub-sector [L]. |
| 174 | PB Ratio | D | F | KR `data[name="P/B"].company_value` | String value; basis undocumented [L]. |
| 175 | PB Premium vs Sector | R | F | Derived, see R-077 | Exact (on given values). Sector benchmark method (mean/median, constituent set) undocumented. Upstox `sector` may not match TickerTape sub-sector [L]. |
| 176 | Dividend Yield | R | F+M | Derived, see R-078 | Exact (if dividend history complete). History depth undocumented [H]. |
| 177 | Dividend Yield vs Sector | N | - | None | No sector dividend-yield benchmark. |
| 178 | PS Ratio | R | F+M | Derived, see R-079 | Approx-Low. Depends on B1 [S]. Trailing basis not specified. |
| 179 | Forward PS Ratio | N | - | None | Requires forward revenue estimates; none provided. |
| 180 | PS Premium vs Sector | N | - | None | No sector P/S benchmark. |
| 181 | EV/EBITDA Ratio | D | F | KR `data[name="EV/EBITDA"].company_value` | Current snapshot only. Underlying EV and EBITDA are not exposed, so the ratio cannot be reproduced [L]. |
| 182 | Enterprise Value | N | - | None | Needs market cap, total debt and cash; no debt line is provided (market cap itself is derived). |
| 183 | EV / EBIT Ratio | N | - | None | Needs EV and EBIT; neither available. |
| 184 | EV / Revenue Ratio | N | - | None | Needs EV; not available. |
| 185 | EV / Invested Capital | N | - | None | Needs EV and invested capital (debt); not available. |
| 186 | EV / Free Cash Flow | N | - | None | Needs EV and FCF (capex); not available. |
| 187 | Price / Free Cash Flow | N | - | None | Needs FCF (capex); not available. |
| 188 | Price / CFO | R | F+M | Derived, see R-080 | Approx-Low. Depends on B1 [S]. |
| 189 | Price / Sales | R | F+M | Derived, see R-081 | Approx-Low. Depends on B1 [S]. |

#### 3.1.15 Growth Metrics  (D 2, R 5, N 6)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 190 | 1Y Forward Revenue Growth | N | - | None | No forward estimates. |
| 191 | 1Y Historical Revenue Growth | D | F | IS `income_statement[category=revenue].history[0].change` (yearly) | String such as "+10.53%". Also computable from annual values. |
| 192 | 5Y Historical Revenue Growth | R | F | Derived, see R-082 | Exact (if depth allows). Needs 6 annual points; samples show 4 [H]. |
| 193 | 1Y Forward EBITDA Growth | N | - | None | No forward estimates. |
| 194 | 1Y Historical EBITDA Growth | N | - | None | EBITDA not available. |
| 195 | 5Y Historical EBITDA Growth | N | - | None | EBITDA not available. |
| 196 | 1Y Forward EPS Growth | N | - | None | No forward estimates. |
| 197 | 1Y Historical EPS Growth | R | F | Derived, see R-083 | Approx. Annual only (fs=true). Sample EPS series jumps from 98.59 (Mar 2023) to 51.45 (Mar 2024); adjustment for share-count changes is undocumented [L]. |
| 198 | 5Y Historical EPS Growth | R | F | Derived, see R-084 | Approx. Depth [H]; adjustment treatment [L]; negative bases break CAGR. |
| 199 | 1Y Fwd Op. Cash Flow Growth | N | - | None | No forward estimates. |
| 200 | 1Y Hist Op. Cash Flow Growth | D | F | CF `cash_flow[category=operating].history[0].change` | String such as "+12.54%". |
| 201 | 5Y Hist Op. Cash Flow Growth | R | F | Derived, see R-085 | Exact (if depth allows). Depth [H]; sign changes break CAGR. |
| 202 | 3Y Historical Dividend Growth | R | F | Derived, see R-086 | Exact (if history complete). Corporate-action history depth undocumented [H]. |

#### 3.1.16 Financial Health & Efficiency Ratios  (D 0, R 4, N 10)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 203 | Long Term Debt to Equity | N | - | None | Long-term debt not available. |
| 204 | Debt to Equity | N | - | None | Debt not available. |
| 205 | Quick Ratio | N | - | None | Inventory not available. |
| 206 | Current Ratio | R | F | Derived, see R-087 | Exact. Annual only (fs=true). |
| 207 | Cash Conversion Cycle | N | - | None | Needs DIO, DSO, DPO; none computable. |
| 208 | Interest Coverage Ratio | N | - | None | No interest or EBIT data. |
| 209 | Inventory Turnover Ratio | N | - | None | Inventory and COGS not available. |
| 210 | Asset Turnover Ratio | R | F | Derived, see R-088 | Exact. End vs average convention not stated [L]. |
| 211 | Days of Inventory Outstanding | N | - | None | Inventory and COGS not available. |
| 212 | Days of Sales Outstanding | N | - | None | Receivables not available. |
| 213 | Days Payable Outstanding | N | - | None | Payables and COGS not available. |
| 214 | Working Capital Turnover Ratio | R | F | Derived, see R-089 | Exact. Sample: Net Current Asset = Current Assets minus Current Liabilities. |
| 215 | Net Income / Liabilities | R | F | Derived, see R-090 | Exact |
| 216 | Earning Power | N | - | None | Definition not given; the standard form (EBIT / total assets) needs EBIT, which is not available. |

#### 3.1.17 Profitability, Returns & Performance Metrics  (D 3, R 6, N 5)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 217 | Return on Equity | D | F | KR `data[name="ROE"].company_value` | String with % sign; current snapshot. Reproducible approximately from PAT / equity. |
| 218 | 5Y Avg Return on Equity | R | F | Derived, see R-091 | Approx. Depth [H]; uses year-end equity unless averaged. |
| 219 | Return on Assets | D | F | KR `data[name="ROA"].company_value` | String with % sign; current snapshot. |
| 220 | 5Y Avg Return on Assets | R | F | Derived, see R-092 | Approx. Depth [H]. |
| 221 | Net Profit Margin | R | F | Derived, see R-093 | Exact |
| 222 | 5Y Avg Net Profit Margin | R | F | Derived, see R-094 | Exact (if depth allows). Depth [H]. |
| 223 | Return on Investment | N | - | None | Definition not given; a ROIC-type measure needs EBIT and debt, which are not available. ROCE exists in KR but is not identical. |
| 224 | 5Y Avg Return on Investment | N | - | None | Same as above. |
| 225 | EBITDA Margin | N | - | None | EBITDA not available. |
| 226 | 5Y Avg EBITDA Margin | N | - | None | EBITDA not available. |
| 227 | Cost of Goods Sold | N | - | None | Not provided. |
| 228 | Cash Flow Margin | R | F | Derived, see R-095 | Exact. TickerTape's numerator (CFO vs FCF) not stated [L]. |
| 229 | 5Y Avg Cash Flow Margin | R | F | Derived, see R-096 | Exact (if depth allows). Depth [H]. |
| 230 | ROCE | D | F | KR `data[name="ROCE"].company_value` | Docs define it as EBIT / capital employed; EBIT and capital employed are not exposed, so it cannot be reproduced. |

#### 3.1.18 Price & Return Performance  (D 2, R 17, N 0)

| # | TickerTape field | Class | Fam | Upstox source | Notes |
|---|---|---|---|---|---|
| 231 | Close Price | D | M | FQ `ohlc.close` / `last_price`; HC candle `[4]` | FQ is an exchange snapshot at request time. |
| 232 | 1D Return | R | M | Derived, see R-097 | Exact. Corporate-action adjustment of candles is undocumented [L]. |
| 233 | 1W Return | R | M | Derived, see R-098 | Exact |
| 234 | 1M Return | R | M | Derived, see R-099 | Exact |
| 235 | 6M Return | R | M | Derived, see R-100 | Exact |
| 236 | 1Y Return | R | M | Derived, see R-101 | Exact |
| 237 | 1W Return vs Nifty | R | M | Derived, see R-102 | Exact (given definition). Difference vs ratio not given [L]. Nifty 50 historical example not shown in HC docs [U]. |
| 238 | 1M Return vs Nifty | R | M | Derived, see R-103 | Exact (given definition). Difference vs ratio not given [L]. Nifty 50 historical example not shown in HC docs [U]. |
| 239 | 6M Return vs Nifty | R | M | Derived, see R-104 | Exact (given definition). Difference vs ratio not given [L]. Nifty 50 historical example not shown in HC docs [U]. |
| 240 | 1Y Return vs Nifty | R | M | Derived, see R-105 | Exact (given definition). Difference vs ratio not given [L]. Nifty 50 historical example not shown in HC docs [U]. |
| 241 | % Away From 52W High | R | M | Derived, see R-106 | Exact. FQ `year_high` basis (intraday vs close) undocumented [L]. |
| 242 | % Away From 52W Low | R | M | Derived, see R-107 | Exact. FQ `year_low` basis undocumented [L]. |
| 243 | Daily Volume | D | M | FQ `volume`; HC candle `[5]` |  |
| 244 | 1D Change in Volume | R | M | Derived, see R-108 | Exact |
| 245 | 1W Change in Volume | R | M | Derived, see R-109 | Exact. Point vs aggregate not given [L]. |
| 246 | 1M Average Volume | R | M | Derived, see R-110 | Exact |
| 247 | 3M Average Volume | R | M | Derived, see R-111 | Exact |
| 248 | Face value | R | F | Derived, see R-112 | Exact (conditional). No face-value field exists. Sample: 5.5 / 0.55 = 10. Works only where a dividend event exists; verified on one sample event. |
| 249 | 5Y CAGR | R | M | Derived, see R-113 | Approx. Adjusted-price status undocumented; CA split/bonus `ratio` could inform adjustment [L]. Interpretation as price CAGR is assumed. |

### 3.2 Reverse map: every Upstox field against TickerTape fields

Role: **Direct** = the field is the TickerTape field; **Input** = used to derive TickerTape fields; **None** = no TickerTape counterpart in the master list.

#### Fundamentals fields

| Endpoint | Upstox field | TickerTape fields supported | Role |
|---|---|---|---|
| PROF | company_profile | none | None |
| PROF | sector | Sub-Sector | Direct |
| PROF | sector_market_cap_inr (value, unit, formatted) | none (sector context only) | None |
| PROF | sector_market_cap_usd (value, unit, formatted) | none (sector context only) | None |
| BS | history[].total_asset | Total Assets; inputs to Asset Turnover, ROA 5Y Avg, Net Income / Liabilities | Direct + Input |
| BS | history[].total_liability | Total Liabilities; Net Income / Liabilities | Direct + Input |
| BS | full_statement: Non-Current Assets | Non Current Assets | Direct |
| BS | full_statement: Current Assets | Total Current Assets; Current Ratio | Direct + Input |
| BS | full_statement: Total Assets | Total Assets; Asset Turnover; 5Y Avg ROA | Direct + Input |
| BS | full_statement: Current Liabilities | Total Current Liabilities; Current Ratio | Direct + Input |
| BS | full_statement: Net Current Asset | Working Capital Turnover Ratio | Input |
| BS | full_statement: Non-Current Liabilities | Non Current Liabilities | Direct |
| BS | full_statement: Equity Capital | Total Equity; Book Value; 5Y Avg Return on Equity | Direct + Input |
| BS | full_statement: Total Equity & Liabilities | none (control total) | None |
| BS | type, time_period, units_in, period | none (metadata) | None |
| CF | cash_flow: operating (value) | Operating Cash Flow; Price / CFO; Cash Flow Margin; 5Y Hist Op. Cash Flow Growth; 5Y Avg Cash Flow Margin | Direct + Input |
| CF | cash_flow: operating (change) | 1Y Hist Op. Cash Flow Growth | Direct |
| CF | cash_flow: investing (value) | Investing Cash Flow | Direct |
| CF | cash_flow: financing (value) | Financing Cash Flow | Direct |
| CF | cash_flow: investing and financing (change) | none | None |
| CF | full_statement: Profit before tax | none (duplicates IS PBT) | None |
| CF | full_statement: Income before WC changes | none | None |
| CF | full_statement: Change in Assets | none | None |
| CF | full_statement: Change in Liabilities | none | None |
| CF | full_statement: Change in WC | Change in Working Capital | Direct |
| CF | full_statement: Cash flow from Operations / Investing / Financing | same as the three category series above | Direct |
| CF | full_statement: Total Cash Flow | Net Change in Cash | Direct |
| CF | full_statement: Cash (Start of the year) | none (reconciliation only) | None |
| CF | full_statement: Cash (End of the year) | Cash and Equivalent | Direct |
| IS | income_statement: revenue | Total revenue (Q); 1Y Historical Revenue Growth (change); 5Y Historical Revenue Growth | Direct + Input |
| IS | income_statement: operating_profit | PBT (Q), unconfirmed mapping | Direct (unconfirmed) |
| IS | income_statement: net_profit | Net Income (Q); TTM PE Ratio | Direct + Input |
| IS | full_statement: Revenue | none (TickerTape has no revenue excluding other income) | None |
| IS | full_statement: Other Income | none | None |
| IS | full_statement: Total Revenue | Total Revenue; PS Ratio; Price / Sales; Asset Turnover; margins | Direct + Input |
| IS | full_statement: Total Expenses | none (aggregate; no counterpart) | None |
| IS | full_statement: Profit Before Tax | PBT | Direct |
| IS | full_statement: Tax | Taxes & Other Items | Direct |
| IS | full_statement: Profit After Tax | Net Income; Net Profit Margin; Net Income / Liabilities; 5Y Avg ROE and ROA; Common Shares Outstanding | Direct + Input |
| IS | full_statement: EPS - Basic | Earnings Per Share; Payout Ratio; EPS growth; Common Shares Outstanding | Direct + Input |
| IS | full_statement: EPS - Diluted | none (TickerTape lists a single EPS) | None |
| SH | promoters | Promoter Holding; Promoter Holding Change 3M and 6M | Direct + Input |
| SH | fii | Foreign Institutional Holding; FII Holding Change 3M and 6M | Direct + Input |
| SH | other_dii | Domestic Institutional Holding; DII Holding Change 3M and 6M | Input |
| SH | mutual_funds | Mutual Fund Holding; MF Holding Change 3M and 6M; Domestic Institutional Holding | Direct + Input |
| SH | retail_and_other | Retail Investor Holding | Direct |
| KR | P/E (company_value) | PE Ratio (both entries); PE Premium vs Sector and vs Sub-sector; Market Capital bridge | Direct + Input |
| KR | P/E (sector_value) | Sector PE; PE Premium vs Sector and vs Sub-sector | Direct + Input |
| KR | P/B (company_value) | PB Ratio; PB Premium vs Sector and vs Sub-sector; Book Value per share | Direct + Input |
| KR | P/B (sector_value) | Sector PB; PB Premium vs Sector and vs Sub-sector | Direct + Input |
| KR | ROA / ROE / ROCE (company_value) | Return on Assets; Return on Equity; ROCE | Direct |
| KR | ROA / ROE / ROCE (sector_value) | none (no sector return benchmarks in master list) | None |
| KR | EV/EBITDA (company_value) | EV/EBITDA Ratio | Direct |
| KR | EV/EBITDA (sector_value) | none (no sector EV/EBITDA field in master list) | None |
| CA | Dividend: amount | Dividend Per Share; Dividend Yield; Payout Ratio; 3Y Historical Dividend Growth; Total Cash Dividend Paid; Face value (with Dividend %) | Input |
| CA | Dividend: expiry_date and event_details dates | Used to assign dividends to fiscal years and trailing 12 months | Input |
| CA | event_details: Dividend % | Face value | Input |
| CA | Bonus / Split / Rights: ratio and details | none directly (relevant to share-count and price adjustment) | None |
| CA | event_details: Announcement date, Record date, Dividend type, Details | none | None |
| COMP | instrument_key, company_profile, sector, sector_market_cap_* | none in master list (peer identification only) | None |

#### Market-data fields used

| Endpoint | Upstox field | TickerTape fields supported | Role |
|---|---|---|---|
| HC | candle[1..4] open, high, low, close | All Technical Indicators (14); most Price, Momentum & Risk metrics (19); returns and returns vs Nifty; 5Y CAGR; Close Price | Direct + Input |
| HC | candle[5] volume | Daily Volume; volume changes and averages; Relative Volume; OBV and AD Line; Future Volume | Direct + Input |
| HC | candle[6] open interest | Future Open Interest; 1W OI changes (futures and options) | Direct + Input |
| FQ | symbol / instrument_token | Ticker | Direct |
| FQ | last_price, ohlc.close | Close Price; valuation bridges (Market Capital, Dividend Yield, TTM PE, PS, Price/CFO) | Direct + Input |
| FQ | volume | Daily Volume; Future Volume; Relative Volume | Direct + Input |
| FQ | oi, previous_oi | Future Open Interest; 1D Change in Future OI; Percentage Rollover | Direct + Input |
| FQ | net_change, prev_close_price | 1D Return | Input |
| FQ | year_high, year_low | % Away From 52W High and Low | Input |
| FQ | average_price | VWAP (possible day-level proxy) | Input |
| FQ | depth, circuit limits, total_buy/sell_quantity, oi_day_high/low, CAS fields | none in master list | None |
| OC | pcr (per strike) | none directly (the aggregate Put Call Ratio is derived from strike OI) | None |
| OC | strike_price, call/put oi, prev_oi | Put Call Ratio; Highest OI strikes; 1D OI changes; Call and Put Open Interest | Input |
| OC | option_greeks.*, bid/ask, ltp, underlying_spot_price | none in master list | None |

---

## 4. Derivable metrics

### 4.1 Shared building blocks

| ID | Building block | Definition | Inputs | Fidelity and limits |
|---|---|---|---|---|
| B1 | Shares outstanding estimate | Profit After Tax divided by EPS - Basic (annual) | IS `Profit After Tax`, `EPS - Basic` | Approx-Low. Gives a weighted-average basic count at best. If PAT includes minority interest the estimate is overstated (sample: 80,787 / 51.47 = approx 1,570 Cr). EPS adjustment for share changes undocumented. |
| B2 | Market capitalisation estimate | Last price x B1. Alternative bridges: P/E x Net Income; P/B x Equity (both depend on undocumented ratio bases). | FQ `last_price`; B1; KR P/E, P/B | Approx-Low. No company market-cap or share-count field exists in any verified endpoint. |
| B3 | TTM net income | Sum of the last four quarterly `net_profit` values | IS with `time_period=quarterly` | Exact if four consecutive quarters are returned (depth undocumented [H]). |
| B4 | Dividend per share by period | Sum of Dividend `amount` per fiscal year or trailing 12 months, dated by ex-date | CA `amount`, `expiry_date` | Exact if event history is complete (depth undocumented [H]). |
| B5 | Index return and volatility series | Nifty 50 closes | HC with the Nifty 50 index key | Index support in HC implied but not shown for Nifty 50 [U]. |
| B6 | Risk-free rate and financing rate | External series or constant | Not from Upstox | Required for Sharpe, Alpha, Fair Value and carry [X]. |
| B7 | Option-chain aggregation | Sum or argmax across strikes of call and put `oi` and `prev_oi` | OC | Exact for the chosen expiry. Stock-option underlyings not demonstrated [U]. |
| B8 | DII total | `other_dii` + `mutual_funds` | SH | Exact if buckets are mutually exclusive (sample sums to about 100%). |

### 4.2 All derivable fields (113)

| ID | TickerTape field | Derivation | Required Upstox fields | Granularity | Fidelity | Caveats |
|---|---|---|---|---|---|---|
| R-001 | Market Capital | Last price x estimated shares outstanding (B2). Alternative bridge: P/E x Net Income. | FQ `last_price` (or HC close); IS `Profit After Tax`, `EPS - Basic`; KR P/E | Point in time | Approx-Low | No company market-cap field exists; only sector market cap (PROF, COMP). Depends on B1 [S]. ETF coverage of Fundamentals is unspecified. |
| R-002 | Common Shares Outstanding | Net Income divided by basic EPS (B1) | IS `Profit After Tax`; IS `EPS - Basic` | Annual | Approx-Low | Yields a weighted-average basic share count at best. Sample: 80,787 / 51.47 = approx 1,570 Cr; PAT may include minority interest, which would overstate shares [L]. EPS series shows a discontinuity in the sample (98.59 to 51.45). |
| R-003 | Total Cash Dividend Paid | Dividend per share for the FY (B4) x estimated shares (B1) | CA `data[name=Dividend].amount`, `expiry_date`; IS `Profit After Tax`, `EPS - Basic` | Annual | Approx-Low | No cash-dividend line in the cash-flow statement. Declared vs paid timing; depends on B1 [S]. |
| R-004 | Book Value | Aggregate: Total Equity. Per share: price divided by P/B, or Total Equity divided by shares (B1). | BS `Equity Capital`; KR P/B `company_value`; FQ `last_price` | Annual / point in time | Exact (aggregate); Approx (per share) | Master list does not state aggregate vs per-share. KR values are strings [L]. |
| R-005 | Dividend Per Share | Sum of per-share dividend amounts whose ex-date falls in the fiscal year (B4) | CA `data[name=Dividend].amount`, `expiry_date` (ex-date), `event_details` | Event-level to annual | Exact (if event history complete) | History depth of corporate actions is undocumented [H]. |
| R-006 | Payout Ratio | DPS divided by basic EPS | CA dividends (B4); IS `EPS - Basic` | Annual | Exact | Inherits B4 completeness. |
| R-007 | Stochastic %K | (Close minus lowest low over N) divided by (highest high minus lowest low over N) | HC daily candles: `candle[1..5]` (open, high, low, close, volume) | Daily | Exact | Look-back N not given in master list. |
| R-008 | Stochastic %D | Moving average of %K | HC daily candles: `candle[1..5]` (open, high, low, close, volume) | Daily | Exact | Smoothing length not given. |
| R-009 | 1W Change in On Balance Volume | OBV (cumulative signed volume) now minus OBV 5 sessions ago | HC `close`, `volume` | Daily | Exact | OBV depends on the series start point. |
| R-010 | 1W Change in AD Line | Accumulation/Distribution line now minus 5 sessions ago | HC high, low, close, volume | Daily | Exact |  |
| R-011 | Super Trend | ATR-based trailing bands from high, low, close | HC high, low, close | Daily | Exact | ATR period and multiplier not given. |
| R-012 | 10D EMA | 10-day exponential moving average of close | HC `close` | Daily | Exact | Candle adjustment for splits/bonus is undocumented. |
| R-013 | 10D SMA | 10-day simple moving average of close | HC `close` | Daily | Exact |  |
| R-014 | 50D EMA | 50-day EMA of close | HC `close` | Daily | Exact |  |
| R-015 | 50D SMA | 50-day SMA of close | HC `close` | Daily | Exact |  |
| R-016 | 100D EMA | 100-day EMA of close | HC `close` | Daily | Exact |  |
| R-017 | 200D EMA | 200-day EMA of close | HC `close` | Daily | Exact |  |
| R-018 | 100D SMA | 100-day SMA of close | HC `close` | Daily | Exact |  |
| R-019 | 200D SMA | 200-day SMA of close | HC `close` | Daily | Exact |  |
| R-020 | 1Y Max Loss | Largest peak-to-trough decline over trailing 1 year (assumed definition) | HC `close` (or high/low) | Daily | Approx | Definition not stated in master list [L]. |
| R-021 | VWAP | Sum of (typical price x volume) divided by sum of volume over intraday candles; FQ `average_price` is a possible day-level proxy | HC minute candles (from Jan 2022); FQ `average_price` | Intraday | Exact (intraday) / Approx (proxy) | FQ describes `average_price` only as "Average price" [L]. Averaging window in master list not given. |
| R-022 | % Price above 1M SMA | Close divided by ~21-session SMA, minus 1 | HC daily candles: `candle[1..5]` (open, high, low, close, volume) | Daily | Exact |  |
| R-023 | % Price above 1Y SMA | Close divided by ~250-session SMA, minus 1 | HC daily candles: `candle[1..5]` (open, high, low, close, volume) | Daily | Exact |  |
| R-024 | % Price above 1M EMA | Close divided by ~21-session EMA, minus 1 | HC daily candles: `candle[1..5]` (open, high, low, close, volume) | Daily | Exact |  |
| R-025 | Beta | Covariance of stock and Nifty returns divided by variance of Nifty returns | HC stock candles; HC Nifty 50 index candles | Daily/weekly | Exact (given window) | Window and return frequency not given. Index candles via HC are implied (India VIX and global indices are called out) but a Nifty 50 historical example is not shown; FQ and OC show `NSE_INDEX` Nifty 50 [U]. |
| R-026 | Sharpe Ratio | (Mean return minus risk-free rate) divided by standard deviation of returns | HC `close`; risk-free rate (external) | Daily | Exact (given rf) | Risk-free rate is not supplied by Upstox [X]. |
| R-027 | Alpha | Jensen's alpha: stock return minus [rf + beta x (market return minus rf)] | HC stock and Nifty candles; risk-free rate (external) | Daily/weekly | Exact (given rf) | Risk-free rate not supplied [X]. |
| R-028 | Relative Volume | Current volume divided by average volume over a look-back | FQ `volume` or HC `volume` | Daily/intraday | Exact | Look-back not given. |
| R-029 | Volatility | Standard deviation of returns over a window (annualised or not) | HC `close` | Daily | Exact (given window) | Window and annualisation not given. |
| R-030 | Volatility vs Nifty | Stock volatility relative to Nifty volatility (ratio or difference) | HC stock and Nifty candles | Daily | Exact (given definition) | Ratio vs difference not given [L]. |
| R-031 | RSI – 14D | Wilder RSI, 14 periods, on close | HC `close` | Daily | Exact |  |
| R-032 | RSI Exponential – 14D | RSI using exponential smoothing, 14 periods | HC `close` | Daily | Exact |  |
| R-033 | ADX Rating – Trend Strength | ADX from +DI/-DI (14 periods) | HC high, low, close | Daily | Exact | "Rating" bucketing rule not given [L]. |
| R-034 | MACD Line 1 – Trend Indicator | EMA(12) minus EMA(26) of close | HC `close` | Daily | Exact | Standard parameters assumed. |
| R-035 | MACD Line 2 – Signal Line Comp | EMA(9) of MACD line | HC `close` | Daily | Exact |  |
| R-036 | % From Upper Bollinger Band | Distance of close from SMA(20)+2 sigma band | HC `close` | Daily | Exact | Denominator (price vs band) not given. |
| R-037 | % From Lower Bollinger Band | Distance of close from SMA(20)-2 sigma band | HC `close` | Daily | Exact |  |
| R-038 | % From Parabolic SAR | Distance of close from Parabolic SAR value | HC high, low | Daily | Exact | Acceleration parameters not given. |
| R-039 | William %R | (Highest high minus close) divided by (highest high minus lowest low), x -100 | HC high, low, close | Daily | Exact |  |
| R-040 | 1D Change in Call OI | Sum over strikes of (call oi minus call prev_oi) (B7) | OC `call_options.market_data.oi`, `prev_oi` per strike | Snapshot | Exact | Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| R-041 | 1D Change in Put OI | Sum over strikes of (put oi minus put prev_oi) (B7) | OC `put_options.market_data.oi`, `prev_oi` per strike | Snapshot | Exact | Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| R-042 | 1W Change in Call OI | Total call OI now minus total call OI about 5 sessions ago | OC call `oi` (now); HC candle `[6]` open interest per option contract (contract keys from OC `call_options.instrument_key`), or stored snapshots | Daily | Exact (if history reachable) | `prev_oi` covers only the previous session. Contract-level history for expired contracts would depend on the Expired Instruments API (not reviewed) [U]. |
| R-043 | 1W Change in Put OI | Total put OI now minus total put OI about 5 sessions ago | OC put `oi`; HC candle `[6]` per option contract, or stored snapshots | Daily | Exact (if history reachable) | Same constraint as call side [U]. |
| R-044 | Highest Call OI Strike | `strike_price` where call oi is maximum | OC `strike_price`, `call_options.market_data.oi` | Snapshot | Exact | Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| R-045 | Highest Put OI Strike | `strike_price` where put oi is maximum | OC `strike_price`, `put_options.market_data.oi` | Snapshot | Exact | Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| R-046 | Highest 1D OI Change CE Strike | `strike_price` with maximum (call oi minus call prev_oi) | OC `call_options.market_data.oi`, `prev_oi` per strike | Snapshot | Exact | Absolute vs percentage change not given [L]. |
| R-047 | Highest 1D OI Change PE Strike | `strike_price` with maximum (put oi minus put prev_oi) | OC `put_options.market_data.oi`, `prev_oi` per strike | Snapshot | Exact | Absolute vs percentage change not given [L]. |
| R-048 | Highest 1W OI Change CE Strike | `strike_price` with maximum 5-session call OI change | OC call `oi`; HC candle `[6]` per contract | Daily | Exact (if history reachable) | Needs contract-level history for every strike [U]. |
| R-049 | Highest 1W OI Change PE Strike | `strike_price` with maximum 5-session put OI change | OC put `oi`; HC candle `[6]` per contract | Daily | Exact (if history reachable) | Needs contract-level history for every strike [U]. |
| R-050 | Put Call Ratio | Total put OI divided by total call OI across strikes (B7) | OC put and call `oi` per strike | Snapshot | Exact | OC `pcr` is returned per strike row (sample: put oi / call oi for that strike), not as an aggregate. |
| R-051 | Fair Value | Spot x (1 + r x days to expiry / 365) minus expected dividends | FQ/HC spot; contract expiry [U]; financing rate (external) | Snapshot | Exact (given rate) | Financing rate not supplied by Upstox [X]. Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| R-052 | 1D Change in Future OI | FQ `oi` minus FQ `previous_oi` | FQ `oi`, `previous_oi` | Snapshot | Exact | Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| R-053 | 1W Change in Future OI | OI now minus OI 5 sessions ago | HC candle `[6]` for the futures contract | Daily | Exact | Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| R-054 | 1D Change in Future Volume | Volume today versus previous session | HC candle `[5]` | Daily | Exact | Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| R-055 | 1W Change in Future Volume | Volume versus 5 sessions ago (or weekly aggregate) | HC candle `[5]` | Daily | Exact | Point vs aggregate comparison not given [L]. Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| R-056 | Basis | Futures price minus spot price | FQ/HC futures close; FQ/HC spot close | Snapshot/daily | Exact | Requires the futures contract's instrument key (instrument master / option-contracts pages not reviewed) [U]. |
| R-057 | Fair Value Spread | Futures price minus fair value | Fair Value (above); futures close | Snapshot | Exact (given rate) | Financing rate external [X]. |
| R-058 | Cash & Carry Profit | Futures minus spot minus carrying cost | Futures and spot prices; financing rate and costs (external) | Snapshot | Approx | Cost assumptions are not in Upstox data [X]. |
| R-059 | Rollover Cost | Next-month futures price minus current-month futures price | FQ/HC for both contracts | Snapshot | Exact | Needs both contract keys [U]. |
| R-060 | Percentage Rollover | Next-month OI divided by (current + next month OI) | FQ `oi` for both contracts | Snapshot | Exact (given definition) | Definition not stated in master list [L]. |
| R-061 | Calendar Spread | Far-month futures price minus near-month futures price | FQ/HC for both contracts | Snapshot | Exact | Needs both contract keys [U]. |
| R-062 | Call Open Interest | Sum of call OI across strikes (B7) | OC call `oi` per strike | Snapshot | Exact | Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| R-063 | Put Open Interest | Sum of put OI across strikes (B7) | OC put `oi` per strike | Snapshot | Exact | Expiry selection (current/next) is not given in master list. Sample is an index underlying; stock-option underlyings are not demonstrated [U]. |
| R-064 | MF Holding Change – 3M | Latest quarter value minus previous quarter value | SH `mutual_funds.history[]` | Quarterly | Exact | Percentage-point change. Needs 2 quarters. |
| R-065 | MF Holding Change – 6M | Latest quarter value minus value two quarters earlier | SH `mutual_funds.history[]` | Quarterly | Exact | Needs 3 quarters. |
| R-066 | Promoter Holding Change – 3M | Latest minus previous quarter | SH `promoters.history[]` | Quarterly | Exact |  |
| R-067 | Promoter Holding Change – 6M | Latest minus two quarters earlier | SH `promoters.history[]` | Quarterly | Exact |  |
| R-068 | Domestic Institutional Holding | `other_dii` plus `mutual_funds` (B8) | SH `other_dii`, `mutual_funds` | Quarterly | Exact (given exclusivity) | Sample's five categories sum to about 100 (100.01), implying mutually exclusive buckets. Composition of `other_dii` is undocumented [L]. |
| R-069 | DII Holding Change – 3M | Change in (other_dii + mutual_funds), one quarter | SH `other_dii`, `mutual_funds` | Quarterly | Exact (given exclusivity) |  |
| R-070 | DII Holding Change – 6M | Change in (other_dii + mutual_funds), two quarters | SH `other_dii`, `mutual_funds` | Quarterly | Exact (given exclusivity) |  |
| R-071 | FII Holding Change – 3M | Latest minus previous quarter | SH `fii.history[]` | Quarterly | Exact |  |
| R-072 | FII Holding Change – 6M | Latest minus two quarters earlier | SH `fii.history[]` | Quarterly | Exact |  |
| R-073 | PE Premium vs Sub-sector | KR P/E `company_value` divided by `sector_value`, minus 1 | KR P/E `company_value`, `sector_value` | Snapshot | Exact (on given values) | Sector benchmark method (mean/median, constituent set) undocumented. Upstox `sector` may not match TickerTape sub-sector [L]. |
| R-074 | PB Premium vs Sub-sector | KR P/B `company_value` divided by `sector_value`, minus 1 | KR P/B `company_value`, `sector_value` | Snapshot | Exact (on given values) | Sector benchmark method (mean/median, constituent set) undocumented. Upstox `sector` may not match TickerTape sub-sector [L]. |
| R-075 | TTM PE Ratio | Last price divided by (TTM net income divided by estimated shares); TTM net income = last four quarters (B3) | IS quarterly `net_profit` (B3); FQ `last_price`; B1 | Quarterly | Approx-Low | Alternative is KR P/E if its basis is TTM (undocumented). Depends on B1 [S]. |
| R-076 | PE Premium vs Sector | KR P/E `company_value` divided by `sector_value`, minus 1 | KR P/E | Snapshot | Exact (on given values) | Sector benchmark method (mean/median, constituent set) undocumented. Upstox `sector` may not match TickerTape sub-sector [L]. |
| R-077 | PB Premium vs Sector | KR P/B `company_value` divided by `sector_value`, minus 1 | KR P/B | Snapshot | Exact (on given values) | Sector benchmark method (mean/median, constituent set) undocumented. Upstox `sector` may not match TickerTape sub-sector [L]. |
| R-078 | Dividend Yield | Trailing-12-month dividends per share divided by last price | CA dividend `amount` and ex-date; FQ `last_price` | Event + snapshot | Exact (if dividend history complete) | History depth undocumented [H]. |
| R-079 | PS Ratio | Estimated market cap (B2) divided by Total Revenue | B2; IS `Total Revenue` | Annual + snapshot | Approx-Low | Depends on B1 [S]. Trailing basis not specified. |
| R-080 | Price / CFO | Estimated market cap (B2) divided by Cash flow from Operations | B2; CF `cash_flow[operating]` | Annual + snapshot | Approx-Low | Depends on B1 [S]. |
| R-081 | Price / Sales | Estimated market cap (B2) divided by Total Revenue (same as PS Ratio) | B2; IS `Total Revenue` | Annual + snapshot | Approx-Low | Depends on B1 [S]. |
| R-082 | 5Y Historical Revenue Growth | 5-year CAGR of Total Revenue | IS yearly `revenue` history or `full_statement["Total Revenue"]` | Annual | Exact (if depth allows) | Needs 6 annual points; samples show 4 [H]. |
| R-083 | 1Y Historical EPS Growth | Basic EPS this year divided by prior year, minus 1 | IS `full_statement["EPS - Basic"]` | Annual | Approx | Annual only (fs=true). Sample EPS series jumps from 98.59 (Mar 2023) to 51.45 (Mar 2024); adjustment for share-count changes is undocumented [L]. |
| R-084 | 5Y Historical EPS Growth | 5-year CAGR of basic EPS | IS `full_statement["EPS - Basic"]` | Annual | Approx | Depth [H]; adjustment treatment [L]; negative bases break CAGR. |
| R-085 | 5Y Hist Op. Cash Flow Growth | 5-year CAGR of Cash flow from Operations | CF `cash_flow[operating]` | Annual | Exact (if depth allows) | Depth [H]; sign changes break CAGR. |
| R-086 | 3Y Historical Dividend Growth | 3-year CAGR of annual DPS (B4) | CA dividend events | Annual | Exact (if history complete) | Corporate-action history depth undocumented [H]. |
| R-087 | Current Ratio | Current Assets divided by Current Liabilities | BS `Current Assets`, `Current Liabilities` | Annual | Exact | Annual only (fs=true). |
| R-088 | Asset Turnover Ratio | Total Revenue divided by Total Assets (end-of-period or two-period average) | IS `Total Revenue`; BS `Total Assets` | Annual | Exact | End vs average convention not stated [L]. |
| R-089 | Working Capital Turnover Ratio | Total Revenue divided by Net Current Asset | IS `Total Revenue`; BS `Net Current Asset` | Annual | Exact | Sample: Net Current Asset = Current Assets minus Current Liabilities. |
| R-090 | Net Income / Liabilities | Profit After Tax divided by total_liability | IS `Profit After Tax`; BS `total_liability` | Annual | Exact |  |
| R-091 | 5Y Avg Return on Equity | Mean of annual (PAT / Equity Capital) over 5 years | IS `Profit After Tax`; BS `Equity Capital` | Annual | Approx | Depth [H]; uses year-end equity unless averaged. |
| R-092 | 5Y Avg Return on Assets | Mean of annual (PAT / Total Assets) over 5 years | IS `Profit After Tax`; BS `Total Assets` | Annual | Approx | Depth [H]. |
| R-093 | Net Profit Margin | Profit After Tax divided by Total Revenue | IS `Profit After Tax`, `Total Revenue` | Annual (or quarterly via summary) | Exact |  |
| R-094 | 5Y Avg Net Profit Margin | Mean of annual net margin over 5 years | IS `Profit After Tax`, `Total Revenue` | Annual | Exact (if depth allows) | Depth [H]. |
| R-095 | Cash Flow Margin | Cash flow from Operations divided by Total Revenue | CF `cash_flow[operating]`; IS `Total Revenue` | Annual | Exact | TickerTape's numerator (CFO vs FCF) not stated [L]. |
| R-096 | 5Y Avg Cash Flow Margin | Mean of annual cash-flow margin over 5 years | CF operating; IS `Total Revenue` | Annual | Exact (if depth allows) | Depth [H]. |
| R-097 | 1D Return | Close today / previous close minus 1; or FQ `net_change` / `prev_close_price` | HC `close`; or FQ `net_change`, `prev_close_price` | Daily | Exact | Corporate-action adjustment of candles is undocumented [L]. |
| R-098 | 1W Return | Close / close ~5 sessions ago minus 1 | HC `close` | Daily | Exact |  |
| R-099 | 1M Return | Close / close ~1 month ago minus 1 | HC `close` | Daily | Exact |  |
| R-100 | 6M Return | Close / close ~6 months ago minus 1 | HC `close` | Daily | Exact |  |
| R-101 | 1Y Return | Close / close ~1 year ago minus 1 | HC `close` | Daily | Exact |  |
| R-102 | 1W Return vs Nifty | Stock 1-week return minus (or relative to) Nifty 1-week return | HC stock `close`; HC Nifty 50 index `close` | Daily | Exact (given definition) | Difference vs ratio not given [L]. Nifty 50 historical example not shown in HC docs [U]. |
| R-103 | 1M Return vs Nifty | Stock 1-month return minus (or relative to) Nifty 1-month return | HC stock `close`; HC Nifty 50 index `close` | Daily | Exact (given definition) | Difference vs ratio not given [L]. Nifty 50 historical example not shown in HC docs [U]. |
| R-104 | 6M Return vs Nifty | Stock 6-month return minus (or relative to) Nifty 6-month return | HC stock `close`; HC Nifty 50 index `close` | Daily | Exact (given definition) | Difference vs ratio not given [L]. Nifty 50 historical example not shown in HC docs [U]. |
| R-105 | 1Y Return vs Nifty | Stock 1-year return minus (or relative to) Nifty 1-year return | HC stock `close`; HC Nifty 50 index `close` | Daily | Exact (given definition) | Difference vs ratio not given [L]. Nifty 50 historical example not shown in HC docs [U]. |
| R-106 | % Away From 52W High | Last price / `year_high` minus 1 | FQ `last_price`, `year_high`; or HC | Snapshot | Exact | FQ `year_high` basis (intraday vs close) undocumented [L]. |
| R-107 | % Away From 52W Low | Last price / `year_low` minus 1 | FQ `last_price`, `year_low`; or HC | Snapshot | Exact | FQ `year_low` basis undocumented [L]. |
| R-108 | 1D Change in Volume | Volume today versus previous session | HC candle `[5]` | Daily | Exact |  |
| R-109 | 1W Change in Volume | Volume versus 5 sessions ago (or weekly aggregate) | HC candle `[5]` | Daily | Exact | Point vs aggregate not given [L]. |
| R-110 | 1M Average Volume | Mean daily volume over ~21 sessions | HC candle `[5]` | Daily | Exact |  |
| R-111 | 3M Average Volume | Mean daily volume over ~63 sessions | HC candle `[5]` | Daily | Exact |  |
| R-112 | Face value | Dividend `Amount` divided by (`Dividend %` / 100), from any dividend event | CA `event_details[name="Amount"]`, `event_details[name="Dividend %"]` | Event | Exact (conditional) | No face-value field exists. Sample: 5.5 / 0.55 = 10. Works only where a dividend event exists; verified on one sample event. |
| R-113 | 5Y CAGR | (Close now / close 5 years ago)^(1/5) minus 1 (assumed to be price CAGR from its section) | HC daily/weekly/monthly `close` (history from Jan 2000) | Daily/monthly | Approx | Adjusted-price status undocumented; CA split/bonus `ratio` could inform adjustment [L]. Interpretation as price CAGR is assumed. |

### 4.3 Derivable fields grouped by what they need

| Group | Count | Fields |
|---|---|---|
| Candle data only (HC) | 41 | R-007 Stochastic %K; R-008 Stochastic %D; R-009 1W Change in On Balance Volume; R-010 1W Change in AD Line; R-011 Super Trend; R-012 10D EMA; R-013 10D SMA; R-014 50D EMA; R-015 50D SMA; R-016 100D EMA; R-017 200D EMA; R-018 100D SMA; R-019 200D SMA; R-020 1Y Max Loss; R-021 VWAP; R-022 % Price above 1M SMA; R-023 % Price above 1Y SMA; R-024 % Price above 1M EMA; R-028 Relative Volume; R-029 Volatility; R-031 RSI – 14D; R-032 RSI Exponential – 14D; R-033 ADX Rating – Trend Strength; R-034 MACD Line 1 – Trend Indicator; R-035 MACD Line 2 – Signal Line Comp; R-036 % From Upper Bollinger Band; R-037 % From Lower Bollinger Band; R-038 % From Parabolic SAR; R-039 William %R; R-097 1D Return; R-098 1W Return; R-099 1M Return; R-100 6M Return; R-101 1Y Return; R-106 % Away From 52W High; R-107 % Away From 52W Low; R-108 1D Change in Volume; R-109 1W Change in Volume; R-110 1M Average Volume; R-111 3M Average Volume; R-113 5Y CAGR |
| Candle data plus index candles (Nifty) or external rate | 8 | R-025 Beta; R-026 Sharpe Ratio; R-027 Alpha; R-030 Volatility vs Nifty; R-102 1W Return vs Nifty; R-103 1M Return vs Nifty; R-104 6M Return vs Nifty; R-105 1Y Return vs Nifty |
| Option chain and futures data (OC, FQ, HC open interest) | 24 | R-040 1D Change in Call OI; R-041 1D Change in Put OI; R-042 1W Change in Call OI; R-043 1W Change in Put OI; R-044 Highest Call OI Strike; R-045 Highest Put OI Strike; R-046 Highest 1D OI Change CE Strike; R-047 Highest 1D OI Change PE Strike; R-048 Highest 1W OI Change CE Strike; R-049 Highest 1W OI Change PE Strike; R-050 Put Call Ratio; R-051 Fair Value; R-052 1D Change in Future OI; R-053 1W Change in Future OI; R-054 1D Change in Future Volume; R-055 1W Change in Future Volume; R-056 Basis; R-057 Fair Value Spread; R-058 Cash & Carry Profit; R-059 Rollover Cost; R-060 Percentage Rollover; R-061 Calendar Spread; R-062 Call Open Interest; R-063 Put Open Interest |
| Fundamentals statements only (BS, CF, IS) | 15 | R-004 Book Value; R-082 5Y Historical Revenue Growth; R-083 1Y Historical EPS Growth; R-084 5Y Historical EPS Growth; R-085 5Y Hist Op. Cash Flow Growth; R-087 Current Ratio; R-088 Asset Turnover Ratio; R-089 Working Capital Turnover Ratio; R-090 Net Income / Liabilities; R-091 5Y Avg Return on Equity; R-092 5Y Avg Return on Assets; R-093 Net Profit Margin; R-094 5Y Avg Net Profit Margin; R-095 Cash Flow Margin; R-096 5Y Avg Cash Flow Margin |
| Shareholding history only (SH) | 9 | R-064 MF Holding Change – 3M; R-065 MF Holding Change – 6M; R-066 Promoter Holding Change – 3M; R-067 Promoter Holding Change – 6M; R-068 Domestic Institutional Holding; R-069 DII Holding Change – 3M; R-070 DII Holding Change – 6M; R-071 FII Holding Change – 3M; R-072 FII Holding Change – 6M |
| Key ratios only (KR) | 4 | R-073 PE Premium vs Sub-sector; R-074 PB Premium vs Sub-sector; R-076 PE Premium vs Sector; R-077 PB Premium vs Sector |
| Corporate actions (CA), alone or with price | 5 | R-005 Dividend Per Share; R-006 Payout Ratio; R-078 Dividend Yield; R-086 3Y Historical Dividend Growth; R-112 Face value |
| Estimated share count or market cap (B1, B2) | 7 | R-001 Market Capital; R-002 Common Shares Outstanding; R-003 Total Cash Dividend Paid; R-075 TTM PE Ratio; R-079 PS Ratio; R-080 Price / CFO; R-081 Price / Sales |

---

## 5. Missing data fields

### 5.1 TickerTape fields with no Upstox source (93)

**Summary by theme**

| Theme | Count | Fields |
|---|---|---|
| Instrument reference | 2 | Name; Lot Size |
| ETF-specific | 2 | Expense Ratio; Tracking Error |
| Cash-flow items | 2 | Capital Expenditure; Free Cash Flow |
| Balance-sheet line items | 18 | Deferred Tax Assets (Net); Other Current Assets; Other Assets; Other Current Liabilities; Other Liabilities; Total Receivables; Total Inventory; Loans & Advances; Net Property, Plant & Equipment; Goodwill & Intangibles; Long Term Investments; Accounts Payable; Total Deposits – Banks; Deferred Tax Liabilities (Net); Minority Interest; Share Capital; Additional Paid–in Capital; Reserves & Surplus |
| Debt | 2 | Long Term Debt; Total Debt |
| Income-statement detail | 14 | EBITDA; Depreciation & Amortization; PBIT; Interest & Other Items; Raw Materials; Power & Fuel Cost; Employee Cost; Selling & Administrative Expenses; Operating & Other expenses; EBITDA (Q); EPS (Q); Operating and Other Expenses (Q); PBIT (Q); Cost of Goods Sold |
| Proprietary scores | 5 | Value Momentum Rank; Price Momentum Rank; Earnings Quality Rank; Price to Intrinsic Value Rank; Fundamental Score |
| Insider and bulk deals | 6 | Insider Trades - 1M Cumulative; Insider Trades - 3M Cumulative; Insider Trades - 6M Cumulative; Bulk Deals - 1M Cumulative; Bulk Deals - 3M Cumulative; Bulk Deals - 6M Cumulative |
| Ownership extras | 3 | No. of Shareholders; Pledged Promoter Holdings; Insurance Firms Holding |
| Forward estimates and analyst data | 12 | Percentage Buy Reco’s; Percentage Upside; No. of analysts with buy reco; Percentage Hold Reco's; Percentage Sell Reco's; Total no. of analysts; Forward PE Ratio; Forward PS Ratio; 1Y Forward Revenue Growth; 1Y Forward EBITDA Growth; 1Y Forward EPS Growth; 1Y Fwd Op. Cash Flow Growth |
| Sector benchmarks not provided | 5 | PS Premium vs Sub-sector; Dividend Yield vs Sub-sector; Sector Dividend Yield; Dividend Yield vs Sector; PS Premium vs Sector |
| Ratios blocked by missing inputs | 22 | Enterprise Value; EV / EBIT Ratio; EV / Revenue Ratio; EV / Invested Capital; EV / Free Cash Flow; Price / Free Cash Flow; 1Y Historical EBITDA Growth; 5Y Historical EBITDA Growth; Long Term Debt to Equity; Debt to Equity; Quick Ratio; Cash Conversion Cycle; Interest Coverage Ratio; Inventory Turnover Ratio; Days of Inventory Outstanding; Days of Sales Outstanding; Days Payable Outstanding; Earning Power; Return on Investment; 5Y Avg Return on Investment; EBITDA Margin; 5Y Avg EBITDA Margin |

**Full list**

| # | TickerTape section | Field | Why missing | Nearest Upstox field (not equivalent) |
|---|---|---|---|---|
| 1 | ETF Metrics | Name | No company or instrument name field in any reviewed endpoint (profile and competitors return a description, not a name). Instruments file not reviewed [U]. | None |
| 2 | ETF Metrics | Expense Ratio | No ETF-specific data in any reviewed endpoint. | None |
| 3 | ETF Metrics | Tracking Error | Requires ETF NAV versus index; NAV not provided. | ETF price candles (HC) and index candles would give a price-based proxy, not NAV-based tracking error |
| 4 | Cash Flow & Equity Metrics | Capital Expenditure | No capex line; investing cash flow is a single aggregate. | CF investing (not equivalent) |
| 5 | Cash Flow & Equity Metrics | Free Cash Flow | Requires capex, which is not provided. | CF operating plus investing (not equivalent; investing includes non-capex flows) |
| 6 | Cash Flow & Equity Metrics | Deferred Tax Assets (Net) | Not a line in the 8-item balance-sheet summary. | None |
| 7 | Cash Flow & Equity Metrics | Other Current Assets | Only the `Current Assets` total is provided. | None |
| 8 | Cash Flow & Equity Metrics | Other Assets | Only Non-Current and Current totals are provided. | None |
| 9 | Cash Flow & Equity Metrics | Other Current Liabilities | Only the `Current Liabilities` total is provided. | None |
| 10 | Cash Flow & Equity Metrics | Other Liabilities | Only Non-Current and Current totals are provided. | None |
| 11 | Balance Sheet Metrics | Total Receivables | Not provided. | None |
| 12 | Balance Sheet Metrics | Total Inventory | Not provided. | None |
| 13 | Balance Sheet Metrics | Loans & Advances | Not provided; no bank/NBFC-specific statement template is documented. | None |
| 14 | Balance Sheet Metrics | Net Property, Plant & Equipment | Not provided. | None |
| 15 | Balance Sheet Metrics | Goodwill & Intangibles | Not provided. | None |
| 16 | Balance Sheet Metrics | Long Term Investments | Not provided. | None |
| 17 | Balance Sheet Metrics | Accounts Payable | Not provided. | None |
| 18 | Balance Sheet Metrics | Total Deposits – Banks | Not provided; no bank-specific template documented. | None |
| 19 | Balance Sheet Metrics | Long Term Debt | No borrowings line. | BS `Non-Current Liabilities` (aggregate, not equivalent) |
| 20 | Balance Sheet Metrics | Total Debt | No borrowings line; `total_liability` is all liabilities, not debt. | BS `total_liability` (not equivalent) |
| 21 | Balance Sheet Metrics | Deferred Tax Liabilities (Net) | Not provided. | None |
| 22 | Balance Sheet Metrics | Minority Interest | Not provided as a separate line. | None |
| 23 | Balance Sheet Metrics | Share Capital | Not provided. "Equity Capital" appears to be total equity, not paid-up capital [L]. | None |
| 24 | Balance Sheet Metrics | Additional Paid–in Capital | Not provided. | None |
| 25 | Balance Sheet Metrics | Reserves & Surplus | Not provided; cannot be backed out without Share Capital. | None |
| 26 | Income Statement & Earnings Metrics | EBITDA | No D&A or finance-cost lines. The `operating_profit` category has an unconfirmed definition (sample equals PBT) [L]. | None |
| 27 | Income Statement & Earnings Metrics | Depreciation & Amortization | Not provided. | None |
| 28 | Income Statement & Earnings Metrics | PBIT | No interest line, so PBT cannot be bridged to PBIT. | None |
| 29 | Income Statement & Earnings Metrics | Interest & Other Items | Not provided; `Total Expenses` is a single aggregate (sample: Total Revenue minus Total Expenses = PBT). | None |
| 30 | Income Statement & Earnings Metrics | Raw Materials | Expense breakdown not provided. | None |
| 31 | Income Statement & Earnings Metrics | Power & Fuel Cost | Expense breakdown not provided. | None |
| 32 | Income Statement & Earnings Metrics | Employee Cost | Expense breakdown not provided. | None |
| 33 | Income Statement & Earnings Metrics | Selling & Administrative Expenses | Expense breakdown not provided. | None |
| 34 | Income Statement & Earnings Metrics | Operating & Other expenses | Only the aggregate `Total Expenses` is provided. | IS `Total Expenses` (includes interest and D&A; not equivalent) |
| 35 | Income Statement & Earnings Metrics | EBITDA (Q) | No D&A or finance-cost data; quarterly summary has only revenue, operating_profit, net_profit. | None |
| 36 | Income Statement & Earnings Metrics | EPS (Q) | Quarterly summary has no EPS; `full_statement` is annual-only. | None |
| 37 | Income Statement & Earnings Metrics | Operating and Other Expenses (Q) | Not provided. | None |
| 38 | Income Statement & Earnings Metrics | PBIT (Q) | No interest data. | None |
| 39 | Futures & Derivatives Metrics | Lot Size | Not present in any reviewed page; instrument master file not reviewed [U]. | None |
| 40 | Valuation & Fundamental Scores | Value Momentum Rank | Proprietary ranking; no Upstox equivalent. | None |
| 41 | Valuation & Fundamental Scores | Price Momentum Rank | Proprietary ranking; no Upstox equivalent. | None |
| 42 | Valuation & Fundamental Scores | Earnings Quality Rank | Proprietary ranking; no Upstox equivalent. | None |
| 43 | Valuation & Fundamental Scores | Price to Intrinsic Value Rank | Proprietary ranking; intrinsic value needs forecasts that are not provided. | None |
| 44 | Valuation & Fundamental Scores | Fundamental Score | Proprietary score; no Upstox equivalent. | None |
| 45 | Insider Trading & Shareholder Activity | Insider Trades - 1M Cumulative | No insider-trade data in any reviewed endpoint. Market Information and News pages not reviewed [U]. | None |
| 46 | Insider Trading & Shareholder Activity | Insider Trades - 3M Cumulative | No insider-trade data in any reviewed endpoint. Market Information and News pages not reviewed [U]. | None |
| 47 | Insider Trading & Shareholder Activity | Insider Trades - 6M Cumulative | No insider-trade data in any reviewed endpoint. Market Information and News pages not reviewed [U]. | None |
| 48 | Insider Trading & Shareholder Activity | Bulk Deals - 1M Cumulative | No bulk-deal data in any reviewed endpoint. Market Information and News pages not reviewed [U]. | None |
| 49 | Insider Trading & Shareholder Activity | Bulk Deals - 3M Cumulative | No bulk-deal data in any reviewed endpoint. Market Information and News pages not reviewed [U]. | None |
| 50 | Insider Trading & Shareholder Activity | Bulk Deals - 6M Cumulative | No bulk-deal data in any reviewed endpoint. Market Information and News pages not reviewed [U]. | None |
| 51 | Insider Trading & Shareholder Activity | No. of Shareholders | SH returns percentages only, not holder counts. | None |
| 52 | Institutional & Ownership Metrics | Pledged Promoter Holdings | No pledge data in SH. | None |
| 53 | Institutional & Ownership Metrics | Insurance Firms Holding | Not reported separately; any insurer holding is inside `other_dii` (composition undocumented). | SH `other_dii` (aggregate, not equivalent) |
| 54 | Analyst Recommendations | Percentage Buy Reco’s | No analyst or consensus data in any of the 8 Fundamentals endpoints. | None |
| 55 | Analyst Recommendations | Percentage Upside | No analyst or consensus data in any of the 8 Fundamentals endpoints. | None |
| 56 | Analyst Recommendations | No. of analysts with buy reco | No analyst or consensus data in any of the 8 Fundamentals endpoints. | None |
| 57 | Analyst Recommendations | Percentage Hold Reco's | No analyst or consensus data in any of the 8 Fundamentals endpoints. | None |
| 58 | Analyst Recommendations | Percentage Sell Reco's | No analyst or consensus data in any of the 8 Fundamentals endpoints. | None |
| 59 | Analyst Recommendations | Total no. of analysts | No analyst or consensus data in any of the 8 Fundamentals endpoints. | None |
| 60 | Sub-Sector Valuation Comparison | PS Premium vs Sub-sector | KR has no P/S ratio and no sector P/S benchmark. | None |
| 61 | Sub-Sector Valuation Comparison | Dividend Yield vs Sub-sector | No sector dividend-yield benchmark. | None |
| 62 | Sub-Sector Valuation Comparison | Sector Dividend Yield | No sector dividend-yield benchmark; COMP gives a peer list but not a full sector constituent list. | None |
| 63 | Valuation Ratios | Forward PE Ratio | Requires forward EPS estimates; none provided. | None |
| 64 | Valuation Ratios | Dividend Yield vs Sector | No sector dividend-yield benchmark. | None |
| 65 | Valuation Ratios | Forward PS Ratio | Requires forward revenue estimates; none provided. | None |
| 66 | Valuation Ratios | PS Premium vs Sector | No sector P/S benchmark. | None |
| 67 | Valuation Ratios | Enterprise Value | Needs market cap, total debt and cash; no debt line is provided (market cap itself is derived). | None |
| 68 | Valuation Ratios | EV / EBIT Ratio | Needs EV and EBIT; neither available. | None |
| 69 | Valuation Ratios | EV / Revenue Ratio | Needs EV; not available. | None |
| 70 | Valuation Ratios | EV / Invested Capital | Needs EV and invested capital (debt); not available. | None |
| 71 | Valuation Ratios | EV / Free Cash Flow | Needs EV and FCF (capex); not available. | None |
| 72 | Valuation Ratios | Price / Free Cash Flow | Needs FCF (capex); not available. | None |
| 73 | Growth Metrics | 1Y Forward Revenue Growth | No forward estimates. | None |
| 74 | Growth Metrics | 1Y Forward EBITDA Growth | No forward estimates. | None |
| 75 | Growth Metrics | 1Y Historical EBITDA Growth | EBITDA not available. | None |
| 76 | Growth Metrics | 5Y Historical EBITDA Growth | EBITDA not available. | None |
| 77 | Growth Metrics | 1Y Forward EPS Growth | No forward estimates. | None |
| 78 | Growth Metrics | 1Y Fwd Op. Cash Flow Growth | No forward estimates. | None |
| 79 | Financial Health & Efficiency Ratios | Long Term Debt to Equity | Long-term debt not available. | None |
| 80 | Financial Health & Efficiency Ratios | Debt to Equity | Debt not available. | BS `total_liability` / `Equity Capital` is liabilities-to-equity, not debt-to-equity |
| 81 | Financial Health & Efficiency Ratios | Quick Ratio | Inventory not available. | None |
| 82 | Financial Health & Efficiency Ratios | Cash Conversion Cycle | Needs DIO, DSO, DPO; none computable. | None |
| 83 | Financial Health & Efficiency Ratios | Interest Coverage Ratio | No interest or EBIT data. | None |
| 84 | Financial Health & Efficiency Ratios | Inventory Turnover Ratio | Inventory and COGS not available. | None |
| 85 | Financial Health & Efficiency Ratios | Days of Inventory Outstanding | Inventory and COGS not available. | None |
| 86 | Financial Health & Efficiency Ratios | Days of Sales Outstanding | Receivables not available. | None |
| 87 | Financial Health & Efficiency Ratios | Days Payable Outstanding | Payables and COGS not available. | None |
| 88 | Financial Health & Efficiency Ratios | Earning Power | Definition not given; the standard form (EBIT / total assets) needs EBIT, which is not available. | None |
| 89 | Profitability, Returns & Performance Metrics | Return on Investment | Definition not given; a ROIC-type measure needs EBIT and debt, which are not available. ROCE exists in KR but is not identical. | KR ROCE (not equivalent) |
| 90 | Profitability, Returns & Performance Metrics | 5Y Avg Return on Investment | Same as above. | None |
| 91 | Profitability, Returns & Performance Metrics | EBITDA Margin | EBITDA not available. | None |
| 92 | Profitability, Returns & Performance Metrics | 5Y Avg EBITDA Margin | EBITDA not available. | None |
| 93 | Profitability, Returns & Performance Metrics | Cost of Goods Sold | Not provided. | None |

### 5.2 Documentation gaps and items needing validation

These are open questions about what Upstox returns. They do not change the classifications above except where flagged.

| ID | Gap | Detail | Fields affected |
|---|---|---|---|
| G1 | History depth | Every Fundamentals sample shows only 4 periods (annual or quarterly). The docs state no maximum. Affects all 5Y metrics and 3Y dividend growth. | 5Y Hist Revenue/EPS/OCF Growth; 5Y Avg ROE, ROA, Net Margin, Cash Flow Margin; 3Y Dividend Growth |
| G2 | Quarterly balance sheet and cash flow | Responses list `time_period` as yearly or quarterly, but only the income statement documents a `time_period` query parameter. | Any quarterly balance-sheet or cash-flow metric |
| G3 | Statement templates by company type | `particular` is documented as examples. No documentation of bank, NBFC or insurer templates, although TickerTape lists Loans & Advances and Total Deposits. | Sector coverage for financial companies |
| G4 | `operating_profit` definition | Category label only. In the annual sample its values equal Profit Before Tax for three periods. | PBT (Q) |
| G5 | PAT attribution | Not stated whether `Profit After Tax` and `net_profit` include minority interest. | Net Income; B1; ROE; margins |
| G6 | `Equity Capital` meaning | Sample behaves as total equity, not paid-up share capital. | Total Equity; Share Capital; Book Value |
| G7 | Consolidated vs standalone for non-statement endpoints | KR, SH, PROF, CA and COMP have no `type` parameter and do not state their basis. | All KR-derived fields |
| G8 | Key-ratio basis and benchmark method | No period, price date, TTM vs annual flag, refresh cadence, or sector-benchmark method (mean/median, constituents). Values are strings. | PE, PB, ROE, ROA, ROCE, EV/EBITDA, premiums |
| G9 | Candle adjustment | Not stated whether HC candles are adjusted for splits, bonuses and dividends. | Returns, technicals, 5Y CAGR, volume averages |
| G10 | Index history | HC examples do not show the Nifty 50 index key; India VIX and global indices are mentioned. | Beta, Alpha, Volatility vs Nifty, returns vs Nifty |
| G11 | Stock-option and futures coverage | OC sample is an index; futures and option-contract keys depend on pages not opened (Instruments, Option Contracts, Expired Instruments). | Options & OI metrics; Futures metrics; Lot Size |
| G12 | Point-in-time data | No filing date, result announcement date or restatement flag on any financial statement. Only a period label is given. | Any backtest or screen that must avoid look-ahead bias |
| G13 | ETF coverage | Fundamentals are by company ISIN. No ETF fields were found. | Section 1 ETF metrics |
| G14 | Corporate-action detail | Only a Dividend sample is shown. Detail labels and ratio format for Bonus, Split and Rights are undocumented. | Price and EPS adjustment |
| G15 | Insider, bulk-deal and analyst data | Not found in the 8 Fundamentals endpoints. Market Information and News pages were not opened. | Insider/bulk deals; analyst fields |
| G16 | Rate limits and error codes | Rate Limits page not opened; the error-code table was absent from the retrieved pages. | Operational scale of per-ISIN calls |

### 5.3 Internal inconsistencies seen in the documentation's sample responses

The samples are illustrative, but they show patterns worth validating against live responses.

| # | Observation | Evidence in samples |
|---|---|---|
| 1 | Summary and full statement can disagree for earlier periods | IS summary `net_profit` for Mar 2023 is 74,088; IS full statement `Profit After Tax` for Mar 2023 is 73,670. |
| 2 | Cash-flow PBT differs from income-statement PBT for earlier periods | Mar 2023: CF 94,801 vs IS 94,046. Mar 2022: CF 84,142 vs IS 82,154. Mar 2025 and Mar 2024 match. |
| 3 | Summary series can extend further than the full statement | In the IS sample, the summary runs to Mar 2026 while `full_statement` ends at Mar 2025. |
| 4 | Summary `revenue` equals Total Revenue, not Revenue | Mar 2025: summary 982,671 = Total Revenue; `Revenue` line is 964,693. |
| 5 | `operating_profit` equals Profit Before Tax | Mar 2025 106,017; Mar 2024 104,340; Mar 2023 94,046, identical to PBT. |
| 6 | EPS series discontinuity | EPS - Basic: 98.59 (Mar 2023) then 51.45 (Mar 2024) while PAT rose; suggests a share-count change whose adjustment is undocumented. |
| 7 | PAT divided by EPS may not equal shares | 80,787 / 51.47 = approx 1,570 Cr; this overstates the share count if PAT includes minority interest. |

---

## 6. Final coverage summary

### 6.1 Overall result (249 TickerTape entries)

| Class | Count | Share |
|---|---|---|
| Directly available | 43 | 17.3% |
| Derivable from Upstox data | 113 | 45.4% |
| Not available | 93 | 37.3% |
| **Total** | **249** | 100.0% |

Directly available plus derivable: **156 of 249 (62.7%)**.

Derivable fields by fidelity: **66 Exact**, **33 Exact given a stated condition** (window, definition, external rate, history depth or contract data), **7 Approx**, **7 Approx-Low**.

Of the Direct fields, **1** rests on an unconfirmed label mapping (PBT (Q) via `operating_profit`).

### 6.2 Coverage by TickerTape section

| # | Section | Fields | Direct | Derivable | Not available | Covered (D+R) |
|---|---|---|---|---|---|---|
| 1 | ETF Metrics | 6 | 2 | 1 | 3 | 50.0% |
| 2 | Cash Flow & Equity Metrics | 18 | 8 | 3 | 7 | 61.1% |
| 3 | Balance Sheet Metrics | 20 | 5 | 0 | 15 | 25.0% |
| 4 | Income Statement & Earnings Metrics | 23 | 8 | 2 | 13 | 43.5% |
| 5 | Technical Indicators | 14 | 0 | 14 | 0 | 100.0% |
| 6 | Price, Momentum & Risk Metrics | 19 | 0 | 19 | 0 | 100.0% |
| 7 | Options & Open Interest Metrics | 11 | 0 | 11 | 0 | 100.0% |
| 8 | Futures & Derivatives Metrics | 17 | 3 | 13 | 1 | 94.1% |
| 9 | Valuation & Fundamental Scores | 6 | 1 | 0 | 5 | 16.7% |
| 10 | Insider Trading & Shareholder Activity | 7 | 0 | 0 | 7 | 0.0% |
| 11 | Institutional & Ownership Metrics | 15 | 4 | 9 | 2 | 86.7% |
| 12 | Analyst Recommendations | 6 | 0 | 0 | 6 | 0.0% |
| 13 | Sub-Sector Valuation Comparison | 8 | 2 | 3 | 3 | 62.5% |
| 14 | Valuation Ratios | 19 | 3 | 6 | 10 | 47.4% |
| 15 | Growth Metrics | 13 | 2 | 5 | 6 | 53.8% |
| 16 | Financial Health & Efficiency Ratios | 14 | 0 | 4 | 10 | 28.6% |
| 17 | Profitability, Returns & Performance Metrics | 14 | 3 | 6 | 5 | 64.3% |
| 18 | Price & Return Performance | 19 | 2 | 17 | 0 | 100.0% |
|  | **All sections** | 249 | 43 | 113 | 93 | 62.7% |

### 6.3 Coverage by source family

| Source family | Direct | Derivable | Total covered (or missing) |
|---|---|---|---|
| Fundamentals API only | 37 | 34 | 71 |
| Market-data endpoints (HC, FQ, OC) | 6 | 73 | 79 |
| Fundamentals plus market data | 0 | 6 | 6 |
| No Upstox source | 0 | 0 | 93 |

Fundamentals-only coverage is **71 fields (28.5%)** of the list. Fields that need market-data endpoints, alone or combined, add **85**.

### 6.4 Coverage of the raw financial-statement fields identified earlier (53)

This is the 53-field raw statement tier from the earlier TickerTape analysis (balance sheet, annual and quarterly income statement, cash flow).

| Class | Count | Share of 53 |
|---|---|---|
| Directly available | 20 | 37.7% |
| Derivable | 3 | 5.7% |
| Not available | 30 | 56.6% |

Raw statement fields not available: Total Receivables; Total Inventory; Loans & Advances; Net Property, Plant & Equipment; Goodwill & Intangibles; Long Term Investments; Deferred Tax Assets (Net); Other Current Assets; Other Assets; Accounts Payable; Total Deposits – Banks; Long Term Debt; Deferred Tax Liabilities (Net); Other Current Liabilities; Other Liabilities; Minority Interest; Share Capital; Additional Paid–in Capital; Reserves & Surplus; Raw Materials; Power & Fuel Cost; Employee Cost; Selling & Administrative Expenses; Operating & Other expenses; Cost of Goods Sold; Depreciation & Amortization; Interest & Other Items; Operating and Other Expenses (Q); EPS (Q); Capital Expenditure.

### 6.5 What Upstox covers well and what it does not

| Area | Finding |
|---|---|
| Price, volume, 52-week range, technical indicators | All computable from candle and quote data. Candle history from Jan 2000 (daily). Adjusted-price status undocumented. |
| Headline financial statements | Present: total revenue, PBT, tax, PAT, basic and diluted EPS; total assets, total liabilities, current and non-current totals, equity; operating, investing and financing cash flow, working-capital change, net change in cash. Annual, with consolidated or standalone selectable. |
| Statement detail | Absent: debt, receivables, inventory, payables, PP&E, goodwill, minority interest, share capital, reserves, expense lines, D&A, interest, capex, dividends paid. |
| Quarterly financials | Only revenue, operating_profit and net_profit. No quarterly EPS, balance sheet or cash flow documented. |
| Ownership | Promoter, FII, MF, other DII and retail-and-other percentages quarterly. Pledges, insurer split and holder counts absent. |
| Valuation | Six current ratios with sector benchmarks. EV, EBITDA, debt and forward estimates absent; market cap and share count absent (only estimable). |
| Growth | Historical revenue, EPS, operating cash flow growth computable subject to history depth. All forward growth and all EBITDA growth absent. |
| Derivatives | Option-chain OI and futures OI and volume available. Lot size, contract keys and expired-contract history depend on unreviewed pages. |
| Analyst, insider, bulk-deal data and proprietary scores | Not present. |

### 6.6 Answer to the objective

**Does Upstox provide sufficient raw data to build a TickerTape-style screener and analytics platform?**

- **Not on its own, for full parity with the master list.** 93 of 249 entries (37.3%) have no Upstox source, and only 43 (17.3%) are returned directly.
- **Raw financial-statement depth is the main shortfall.** Of the 53 raw statement fields, 20 are directly available and 30 are not. The missing set includes all debt, working-capital, fixed-asset, expense-breakdown, D&A, interest, capex and minority-interest data, so EBITDA, enterprise value, leverage, working-capital cycle and free-cash-flow metrics cannot be built from Upstox data.
- **Market, technical, F&O and shareholding-trend metrics are well covered**, because they derive from candle, quote, option-chain and quarterly shareholding data. Most of this depends on endpoints outside the Fundamentals group.
- **Several covered fields carry caveats that matter for a screener**: unknown history depth, no point-in-time dates for financials, undocumented candle adjustment, string-typed key ratios of undocumented basis, and an estimated rather than reported share count.
- **Categories with no Upstox data at all** in the reviewed pages: analyst recommendations (6), insider trades and bulk deals (6), pledged holdings, holder counts, ETF-specific fields, forward estimates, and the five proprietary scores and ranks.

*End of analysis.*
