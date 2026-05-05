# Alfred — Nordic Due Diligence MCP

Alfred is a due diligence research assistant for Nordic listed companies (Norway, Sweden, Denmark, Finland, Iceland). It exposes a single MCP tool — `due_diligence_report(company)` — and handles all search strategy internally.

**Live endpoint:** `https://alfred.aidatanorge.no/mcp`

## What Alfred does

Calling `due_diligence_report("Mowi")` triggers a multi-phase pipeline:

1. **Probe** — looks up the confirmed exchange ticker, available fiscal years, report types, data sources, and business description excerpts from the database before planning
2. **Haiku planning** — Claude Haiku generates a structured search plan (financial periods, operational sections, competitors, news) informed by what data actually exists
3. **Section generation** — Alfred automatically adds macro (commodity prices, FX rates) and power price sections aligned to the financial periods
4. **Parallel search** — all sections execute simultaneously, with source filters that ensure XBRL annual report data isn't crowded out by press releases

The caller receives structured JSON grouped by section. No query formulation needed — just synthesize the returned data.

## Tool

### `due_diligence_report(company)`

| Parameter | Type | Description |
|---|---|---|
| `company` | string | Company name or ticker, e.g. `"Mowi"`, `"Equinor"`, `"EQNR"` |

**Returns:**

```json
{
  "company": "Mowi",
  "ticker_confirmed": "MOWI",
  "country": "NO",
  "sector": "aquaculture",
  "macro_factors": ["salmon_price", "NOKUSD=X"],
  "power_zones": [],
  "periods_covered": ["Q12026", "Q42025"],
  "generated_at": "2026-05-05T10:00:00Z",
  "sections": {
    "financials_q1_2026": [...],
    "risks": [...],
    "macro_salmon_price_q1_2026": [...],
    ...
  }
}
```

Each section is a list of document chunks with fields: `text`, `source`, `ticker`, `published_date`, `report_type`, `score`.

## Coverage

| Country | Exchange | Sources |
|---|---|---|
| Norway | Oslo Børs | Newsweb filings, XBRL annual reports, Cision press releases |
| Sweden | Nasdaq Stockholm, First North SE | MFN filings, XBRL annual reports |
| Denmark | Nasdaq Copenhagen, First North DK | Nasdaq DK filings, XBRL annual reports |
| Finland | Nasdaq Helsinki, First North FI | Nasdaq FI filings, XBRL annual reports |
| Iceland | Nasdaq Iceland | Newsweb filings |

Macro data: Brent crude (BZ=F), aluminium (ALI=F), natural gas (NG=F), NOK/USD, SEK/USD, DKK/USD, Atlantic salmon spot price. Power prices: ENTSO-E day-ahead prices for NO1–NO5, SE1–SE4, DK1, DK2, FI.

## Usage

Add to your MCP config:

```json
{
  "mcpServers": {
    "alfred": {
      "type": "http",
      "url": "https://alfred.aidatanorge.no/mcp"
    }
  }
}
```

Then ask your agent: *"Give me a due diligence report on Norsk Hydro."* Alfred returns the data; your agent writes the report.

## License

MIT
