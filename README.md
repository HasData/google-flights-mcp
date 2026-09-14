# Google Flights MCP Server

<!-- mcp-name: com.hasdata/google-flights -->

A hosted Model Context Protocol (MCP) server that gives Claude, Cursor, Windsurf and any other MCP client one Google Flights tool. Search one-way, round-trip and multi-city itineraries with fares, flight legs, carbon emissions and price history, all as structured JSON, with no Google account and no retired travel API to work around.

**1,000 free credits every month, no card required**, which is about 66 flight searches.

```
https://mcp.hasdata.com/api/mcp?apis=google_travel_flights
```

[![Glama score](https://glama.ai/mcp/servers/HasData/google-flights-mcp/badges/score.svg)](https://glama.ai/mcp/servers/HasData/google-flights-mcp)
[![tool contract](https://github.com/HasData/google-flights-mcp/actions/workflows/contract.yml/badge.svg)](https://github.com/HasData/google-flights-mcp/actions/workflows/contract.yml)
[![MCP](https://img.shields.io/badge/MCP-remote%20%7C%20streamable%20HTTP-6366f1?style=flat-square)](https://modelcontextprotocol.io)
[![Tools](https://img.shields.io/badge/tools-1-10b981?style=flat-square)](#tools)
[![npm](https://img.shields.io/npm/v/@hasdata/google-flights-mcp?style=flat-square&logo=npm&label=npm&color=cb3837)](https://www.npmjs.com/package/@hasdata/google-flights-mcp)
[![PyPI](https://img.shields.io/pypi/v/hasdata-google-flights-mcp?style=flat-square&logo=pypi&logoColor=white&label=PyPI&color=3775a9)](https://pypi.org/project/hasdata-google-flights-mcp/)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## Contents

- [What you need](#what-you-need)
- [Quick start](#quick-start)
- [Example prompts](#example-prompts)
- [Tools](#tools)
- [Errors and failure paths](#errors-and-failure-paths)
- [Pricing, free tier and limits](#pricing-free-tier-and-limits)
- [Tool selection](#tool-selection)
- [How it compares](#how-it-compares)
- [FAQ](#faq)
- [HasData links](#hasdata-links)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## What you need

An MCP client and a HasData API key from the [dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-mcp), free to create with no card, and the free tier covers about 66 calls a month at the 15-credit rate. This is a remote server, so the simplest path is a URL and an `x-api-key` header, with no container to run and no Google account anywhere in the flow. A client that only speaks stdio reaches it through a thin launcher, published as `@hasdata/google-flights-mcp` on npm and `hasdata-google-flights-mcp` on PyPI, shown below.

## Quick start

The server URL is the same for every client. We run it hands-on in Claude Code and Claude Desktop. The other blocks follow each client's own documented format for a remote server.

| Field | Value |
| :--- | :--- |
| URL | `https://mcp.hasdata.com/api/mcp?apis=google_travel_flights` |
| Transport | HTTP, streamable |
| Auth header | `x-api-key: HASDATA_API_KEY` |

Clients with OAuth support can add the same URL as a connector and sign in without putting a key in a config file.

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add --transport http google-flights "https://mcp.hasdata.com/api/mcp?apis=google_travel_flights" \
  --header "x-api-key: HASDATA_API_KEY"
```

</details>

<details>
<summary><b>Claude Desktop</b></summary>

Settings, then Connectors, then Add custom connector, then paste `https://mcp.hasdata.com/api/mcp?apis=google_travel_flights` and sign in.

For the config-file route, Claude Desktop loads only local (stdio) servers, so it reaches a remote server through a stdio launcher. The `@hasdata/google-flights-mcp` package is that launcher, and it reads the key from the environment. Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "google-flights": {
      "command": "npx",
      "args": ["-y", "@hasdata/google-flights-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

For Python instead of Node, swap the launcher for the PyPI package, which `uvx` runs without a manual install:

```json
{
  "mcpServers": {
    "google-flights": {
      "command": "uvx",
      "args": ["hasdata-google-flights-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Cursor</b></summary>

`~/.cursor/mcp.json` for every project, or `.cursor/mcp.json` for one:

```json
{
  "mcpServers": {
    "google-flights": {
      "url": "https://mcp.hasdata.com/api/mcp?apis=google_travel_flights",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Windsurf</b></summary>

`~/.codeium/windsurf/mcp_config.json`. Windsurf calls the field `serverUrl`, not `url`:

```json
{
  "mcpServers": {
    "google-flights": {
      "serverUrl": "https://mcp.hasdata.com/api/mcp?apis=google_travel_flights",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>VS Code</b></summary>

`.vscode/mcp.json` in the workspace:

```json
{
  "servers": {
    "google-flights": {
      "type": "http",
      "url": "https://mcp.hasdata.com/api/mcp?apis=google_travel_flights",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

## Example prompts

Prompts, not code. Paste one in and the agent picks the tool itself. Each is annotated with the calls it takes, because every successful call costs 15 credits.

> Find one-way flights from JFK to London Heathrow on September 15, sorted by price, and give me the three cheapest with airline and carbon estimate.

*One call, 15 credits. Fares, legs and emissions all come back together.*

> Same route but non-stop only, in business class, and tell me which option has the lowest emissions.

*One call, 15 credits. Cabin and stops are filters on the one request.*

> Is $295 a good price for JFK to LHR right now, given the price history?

*One call, 15 credits. The response carries `priceInsights` with a typical range and a price level.*

> Round trip JFK to LHR, out September 15 back September 22, cheapest fare.

*Two calls, 30 credits. Google returns outbound options first, then the return leg is a second call keyed by the option you pick.*

A round trip is two calls by design. The first returns outbound itineraries each with a `departureToken`, and you pass that token back to get the matching return flights. One-way and the price check are a single call each.

## Tools

| Tool | Credits | What it returns |
| :--- | :--- | :--- |
| `hasdata_google_travel_flights_getGoogleFlights` | 15 | Per-itinerary price, currency, total duration, stops, flight legs with airline, flight number, aircraft, departure/arrival airports and times, CO2 emissions, plus… |

One tool, read-only. The sample below is trimmed from a real call, and fares move constantly. Read it as a shape. The tool name links to its endpoint reference, which carries the full parameter list.

The sample is the payload, not the whole response. A `tools/call` result carries one text block, and that text is itself JSON holding `url`, `status`, `text` and `json`, with the scraped data under `json`. From a raw JSON-RPC response the path is `result.content[0].text`, parsed, then `.json`. A chat client unwraps that for you and code talking to the endpoint directly does not.

### Get Google Flights results

[`hasdata_google_travel_flights_getGoogleFlights`](https://docs.hasdata.com/apis/google-travel/flights?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-mcp)

Itineraries for a route and date, with fares, legs, emissions and price history.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `departureId` | string | yes | IATA code such as `JFK`, or a location kgmid like `/m/02_286`. Comma-separate several airports |
| `arrivalId` | string | yes | Same format as `departureId` |
| `outboundDate` | string | yes | `YYYY-MM-DD` |
| `type` | string | | `roundTrip` by default, `oneWay`, or `multiCity` with `multiCityJson` |
| `returnDate` | string | | Required when `type` is `roundTrip` |
| `travelClass` | string | | `economy`, `premiumEconomy`, `business` or `first` |
| `stops` | string | | `nonStop`, `oneStopOrFewer` or `twoStopsOrFewer` |
| `sortBy` | string | | `topFlights` default, plus `price`, `duration`, `emissions`, `departureTime`, `arrivalTime` |
| `adults` / `children` / `infantsInSeat` / `infantsOnLap` | number | | Passenger mix |
| `maxPrice` / `maxDuration` / `bags` | number | | Ceilings and carry-on count |
| `includeAirlines` / `excludeAirlines` | string | | Comma-separated IATA airline codes, one or the other, not both |
| `departureToken` | string | | Select an outbound option and fetch its return or next leg |
| `bookingToken` | string | | Fetch booking options for a chosen itinerary |
| `currency` / `gl` / `hl` | string | | Currency and the country and language of the search |
| `deepSearch` | boolean | | Match what Google shows in a browser, slower to return |

The reference also documents `includeConnections`, `excludeConnections`, `layoverDuration`, `outboundTimes`, `returnTimes`, `showHidden`, `lessEmissions` and `multiCityJson`.

Results split into `bestFlights` and `otherFlights`. Each itinerary carries `price`, `type`, `totalDuration` in minutes, a `flights` array of legs, a `carbonEmissions` object, and a `bookingToken`. Each leg holds the `departureAirport` and `arrivalAirport` (each with `id`, `name` and local `time`), `duration`, `airline`, `flightNumber`, `airplane`, `legroom`, `travelClass`, an `extensions` array, and `oftenDelayedByOver30Min` on legs Google flags. A non-stop itinerary has one leg, a connection has several.

> `carbonEmissions` is in grams, not kilograms. `thisFlight: 433000` is 433 kg. `differencePercent` compares it to `typicalForThisRoute`, so a negative number is a greener-than-average flight.

```json
{
  "price": 295,
  "type": "One way",
  "totalDuration": 415,
  "flights": [
    {
      "departureAirport": { "id": "JFK", "name": "John F. Kennedy International Airport", "time": "2026-09-15 8:15" },
      "arrivalAirport": { "id": "LHR", "name": "Heathrow Airport", "time": "2026-09-15 20:10" },
      "duration": 415,
      "airline": "Virgin Atlantic",
      "flightNumber": "VS 26",
      "airplane": "Boeing 787",
      "travelClass": "Economy"
    }
  ],
  "carbonEmissions": { "thisFlight": 367000, "typicalForThisRoute": 419000, "differencePercent": -12 },
  "bookingToken": "W1t7..."
}
```

`priceInsights` sits alongside the itineraries with `lowestPrice`, a `typicalPriceRange`, a `priceLevel` such as `typical`, and a `priceHistory` of `[timestamp, price]` points. `airports` echoes the resolved departure and arrival airports with city and country.

## Errors and failure paths

Your client almost never sees an HTTP error code from a tool call. The MCP layer answers 200 and puts the failure inside the result, with `isError` set to `true` and the reason as text. The agent reads a message where you might expect a status line.

**A wrong key surfaces as tool output, not as a failed connection.** `tools/list` accepts any non-empty key and returns the tool, so the client completes its handshake and shows green. The first tool call then comes back with `isError: true` and the text `HasData API error: 401 Unauthorized`. Watch for that string, because nothing earlier in the flow reports the problem.

**A missing key is the one real HTTP error.** Authorization runs before any tool, and the connection itself fails with 401. CORS headers are present, and a browser client reads the status and not an opaque network failure.

**An argument that breaks the tool's schema is rejected before it becomes a scrape.** The server answers with `isError: true` and the text `MCP error -32602: Input validation error`, naming the offending field. A `roundTrip` without a `returnDate`, or `includeAirlines` together with `excludeAirlines`, is caught here.

**A route with no flights on the date returns a successful result with the itinerary arrays empty**, not an error. `requestMetadata.status` still reads `ok`. Test for the flights before you rank them.

**A bad airport code returns 400** with `requestMetadata.status` set to `error`. Use IATA codes or kgmids, not city names.

Results that carry data also carry a `requestMetadata.id` worth quoting in support.

## Pricing, free tier and limits

Every Google Flights call costs **15 credits per successful call**. Response size does not change the price, and the deep search costs the same as a standard one.

The free tier is **1,000 credits every month with no card**, which is about 66 flight searches. It renews with the billing cycle, so a low-volume agent runs on the free tier indefinitely.

Paid plans start at **$49 a month** for 200,000 credits, which is about 13,000 searches. The unit price falls with volume, from **$3.68 per 1,000 calls** on the entry plan to **$1.49** on Business, **$1.25** on Growth and **$1.12** on the largest [high-volume plans](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-mcp).

Your plan also sets concurrency. The free tier allows 1 request at a time, Startup 15, Business 30, Growth 50, and the high-volume plans run from 200 to 1,500. Handle the overflow case defensively in anything unattended.

A request that comes back non-200 is not billed. A round trip is two calls, so budget for it.

## Tool selection

The `apis` query parameter decides which tools your agent sees. Fewer tools means less context spent on tool definitions, and fewer chances for the model to reach for the wrong one.

```
?apis=google_travel_flights          the one tool in this repo
?apis=google_travel                   add Google Hotels
?apis=google_travel_flights,airbnb    flights plus Airbnb stays
```

The parameter takes provider names like `google_travel` and individual API names like `google_travel_flights`. Misspelled names are ignored. If every name is wrong the request fails with 400, and the body lists both what it did not recognise and every valid value. Drop the parameter and the same endpoint exposes all 57 HasData tools.

## How it compares

Google retired its QPX Express flight API in 2018 and never replaced it, so there is no official Google Flights API. The remaining routes are scraping the public results or licensing raw GDS fare data, which is heavy and expensive. This server reads the same results the site shows and returns them as JSON.

| | Official Google API | This server |
| :--- | :--- | :--- |
| Availability | None since QPX Express closed in 2018 | Maintained schema over the live results |
| Emissions data | Not offered | Per-itinerary, compared to the route average |
| Price history | Not offered | `priceInsights` with a typical range |
| Setup | Nothing to set up, because it does not exist | One key and one URL |
| Cost | Not applicable | Paid past the free tier, 15 credits a call |

**What this server does not do.** No booking and no payment. It reads fares, legs and the tokens Google itself uses to move to booking, and hands the booking step back to you.

## FAQ

### Is there an official Google Flights API?

No. Google closed QPX Express in 2018 and has not shipped a replacement. Every option reads the same public results the website serves. This one is maintained by HasData and returns them as structured JSON.

### What is a Google Flights MCP server?

A server that exposes Google Flights as a tool an AI client can call. The client sends a tool call over the Model Context Protocol, the server fetches the itineraries and returns structured JSON, and the model works with the result. This one exposes a single tool and runs remotely.

### Why is a round trip two calls?

Google returns outbound options first, each with a `departureToken`. You pick one and pass its token back to get the return flights that pair with it. That mirrors how the site works, and it is why a round trip costs 30 credits.

### Are the carbon numbers in kilograms?

No, grams. `thisFlight: 433000` means 433 kg, and `differencePercent` compares it to the route average.

### What is deep search?

A slower mode that returns exactly what Google Flights shows in a browser. Leave it off for speed, turn it on when you need parity with the site.

### Can I use this together with other HasData APIs?

Yes. The `apis` parameter takes a list, and `?apis=google_travel` adds Google Hotels alongside flights. [Drop the parameter](#tool-selection) and you get everything.

### Compliance and personal data

HasData accesses publicly available data only. A platform's terms may restrict automated access, and you are responsible for your own compliance.

## HasData links

| | |
| :--- | :--- |
| Product page and request builder | [Google Flights API](https://hasdata.com/apis/google-flights-api?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-mcp) |
| Server documentation | [MCP server docs](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-mcp) |
| All 57 tools in one server | [HasData/hasdata-mcp](https://github.com/HasData/hasdata-mcp) |
| Client walkthroughs | [MCP clients and integrations](https://hasdata.com/integrations/mcp?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-mcp) |
| Everything else we scrape | [Google Flights API and 54 more](https://hasdata.com/apis/?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-mcp) |
| Plans and credit costs | [Plans and credit costs](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-mcp) |
| Keys and usage | [HasData dashboard](https://app.hasdata.com?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-mcp) |
| Node launcher on npm | [@hasdata/google-flights-mcp](https://www.npmjs.com/package/@hasdata/google-flights-mcp) |
| Python launcher on PyPI | [hasdata-google-flights-mcp](https://pypi.org/project/hasdata-google-flights-mcp/) |

## Development

This repository is configuration and documentation for a remote server. There is no build step and nothing to containerize.

The tests in `test/` assert the tool contract, the part that can break without a commit here. They check that `?apis=google_travel_flights` returns exactly one tool, that it still declares its required parameters, that the name has not changed, and that the key in use is actually accepted. That last check calls the tool for real and costs 15 credits, which is the price of a canary that can fail for the right reason.

```bash
# macOS and Linux
HASDATA_API_KEY=your_key_here npm test

# Windows PowerShell
$env:HASDATA_API_KEY="your_key_here"; npm test
```

The same suite runs in CI on every push and once a week on a schedule, because the upstream tool list can change without anyone touching this repository. A failure means the tool list moved, the key stopped working, or the endpoint was unreachable, and the assertion message says which.

## Contributing

Corrections to the parameter table and the response sample are the most useful contribution, because those are the parts that drift. Include the call you made and the response you got. Pull requests from forks run the suite without a key, and the live checks skip instead of going red.

## License

MIT. See [LICENSE](LICENSE).
