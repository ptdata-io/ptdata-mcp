# PtData MCP server

**US company fundamentals as they were known on any past date.** PtData is a remote MCP server (and REST API) for point-in-time US equity data: fundamentals, survivorship-free index constituents, SEC filing metadata and earnings events. Every value carries the filing it came from and the date it became public.

```
https://api.ptdata.io/mcp
```

Streamable HTTP. No install, no key needed for the free tier.

[![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=ptdata&config=eyJ1cmwiOiJodHRwczovL2FwaS5wdGRhdGEuaW8vbWNwIn0=)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_PtData-0098FF?logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=ptdata&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fapi.ptdata.io%2Fmcp%22%7D)

## Why point-in-time

Most financial datasets keep only the latest version of each number. When a company restates, the old figure is overwritten. When it goes bankrupt or gets acquired, it disappears from the index history. A backtest or an agent working on that data sees information that did not exist yet on the date it is simulating.

PtData stores every version. You pass an `as_of_date` and get what was public on that date.

**Example: GE, fiscal year 2017 revenue**

| As of | Revenue | Source filing (accession) |
|---|---|---|
| 2018-02-28 | **$122.09B** | Original FY2017 10-K, `0000040545-18-000014` |
| today | **$110.97B** | Restated in the FY2018 10-K, `0000040545-19-000014` |

```bash
curl "https://api.ptdata.io/v1/fundamentals?ticker=GE&as_of_date=2018-02-28&metrics=revenue&fiscal_year=2017&fiscal_period=FY"
```

## Tools

All four tools are read-only and idempotent.

| Tool | What it returns |
|---|---|
| `get_fundamentals` | Fundamentals for a ticker as known on `as_of_date`, each with source filing, knowledge time and lineage. Missing metrics come back under `unavailable` with a reason instead of being silently dropped. |
| `get_index_constituents` | S&P 500 (or another supported index) members on `as_of_date`, survivorship-free, with the ticker used at the time. |
| `get_filing_metadata` | SEC filings for a ticker available on `as_of_date`, newest first, with amendment linkage. |
| `get_earnings_event` | An earnings event by `ticker`, `fiscal_year` and `fiscal_period` (FY or Q1–Q4): 8-K acceptance timestamp, as-reported figures and the realized price reaction. |

## Install

### Claude Code

```bash
claude mcp add --transport http ptdata https://api.ptdata.io/mcp
```

With an API key:

```bash
claude mcp add --transport http ptdata https://api.ptdata.io/mcp \
  --header "Authorization: Bearer YOUR_PTDATA_API_KEY"
```

### Claude (web and Desktop)

Go to **Customize > Connectors**, click **+ Add**, then **Add custom connector**. Name it `PtData` and paste `https://api.ptdata.io/mcp` as the URL. Custom connectors are available on all Claude plans; the Free plan allows one.

### Cursor

Click **Add to Cursor** above, or add this to `~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project):

```json
{
  "mcpServers": {
    "ptdata": {
      "url": "https://api.ptdata.io/mcp"
    }
  }
}
```

### VS Code

Click **Install in VS Code** above, or run **MCP: Add Server** from the Command Palette, choose **HTTP**, and enter `https://api.ptdata.io/mcp`.

### Any other MCP client

```json
{
  "mcpServers": {
    "ptdata": {
      "url": "https://api.ptdata.io/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_PTDATA_API_KEY"
      }
    }
  }
}
```

The `headers` block is optional. Leave it out to use the free tier.

## Example prompts

- "What was Apple's revenue for fiscal 2015 as it was known on 2016-01-31, and which filing did it come from?"
- "Show GE's FY2017 revenue as of 2018-02-28 and as of today. Why are they different?"
- "List the S&P 500 members on 2008-09-01. Which of them were removed by the end of 2009?"
- "Which SEC filings did Enron have on record as of 2001-10-15, and were any of them later amended?"
- "When exactly did Tesla's Q4 2023 earnings 8-K hit EDGAR, and how did the stock react?"
- "Build me a list of S&P 500 constituents on 2012-06-30 with their net income for the latest fiscal year known on that date."

## Plans

| Plan | Price | Limits | Key |
|---|---|---|---|
| Free | $0 | 120 requests/min, 200/day per IP | Not needed |
| Pro | $49/month | 15,000 requests/day | Required |
| Scale | $199/month | 100,000 requests/day | Required |

The free tier is for evaluation. Paid plans cover commercial use. Get a key at [api.ptdata.io/account](https://api.ptdata.io/account).

## Coverage

- 8,300+ companies in the store, 7M+ SEC filings indexed, updated daily from EDGAR.
- XBRL fundamentals from FY2009 for about 6,300 companies.
- Pre-XBRL tier for FY2001–2008, about 1,500 companies so far (including Enron, WorldCom and Bear Stearns). These values are extracted from filing text and carry a confidence label.
- Survivorship-free index rosters: S&P 500 from 1996, DJIA, and DJT, DJU and NDX over shorter windows.
- Banks and insurers get the lines they actually report, not an invented "revenue".
- Not included: prices, analyst consensus, intraday data.

All data comes from SEC EDGAR.

## Links

- API docs: https://api.ptdata.io/docs
- Correctness test suite (run it against the live API): https://github.com/ptdata-io/ptdata-correctness
- Methodology and known gaps: https://github.com/ptdata-io/ptdata-methodology
- MCP Registry: `io.ptdata/ptdata`
- Website: https://ptdata.io
- Contact: info@ptdata.io

## License

The configuration files and documentation in this repository are MIT licensed. The PtData service itself is governed by its terms of service at https://ptdata.io.
