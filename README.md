# @pipeworx/china-recalls

Product recall announcements published by China's State Administration for Market Regulation (SAMR): about 1,400 announcements since 2018, searchable by title, category and date, with the full notice text for each. Keyless, live per request.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1863+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `china_search_recalls(query?, category?, from?, to?, limit?, offset?)` — recall announcements newest first. Returns id, title, date and an inferred category (`vehicle`, `consumer_product`, `monthly_digest`). Answers "has this company or product been recalled in China, and when".
- `china_get_recall(id)` — one announcement in full: recalling company, units affected, production window, defect and remedy text (Chinese), issuing unit, publication time, and attachments such as the recall-plan PDF.

## Auth

Keyless.

## Data sources

- State Administration for Market Regulation (SAMR), China, product recall announcements (the recall column of the SAMR government-affairs pages). Public, served without credentials. The pack proxies per call and stores nothing.

## Things the next person would otherwise rediscover

- **Scope is mostly vehicles.** Of the listed announcements the large majority are cars and motorcycles. Consumer goods (e.g. a power bank, a vacuum flask) are a small minority. The Defective Product Administration Center site was not reachable from our vantage, so no separate consumer-goods list is covered.
- **Monthly "domestic and foreign recall information" digests** are listed but published as images; `china_get_recall` reports `image_only` for them rather than inventing text.
- **Titles are Chinese.** `query` matches title text; a short alias table maps common English brand and product words (tesla, xiaomi, power bank, ...) to their Chinese forms and the terms actually used are echoed in `query_terms_used`.
- **Category is inferred from the title**, not published by SAMR. Treat `consumer_product` as "not recognised as a vehicle".
- **Dates** are announcement dates, not recall start dates; many announcements are published in batches on one day. The production window and units affected are parsed from the text and are null when the notice words them differently.
- A non-empty upstream list is asserted on every call; an empty first page raises an error instead of reporting "no recalls".

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "china-recalls": {
      "url": "https://gateway.pipeworx.io/china-recalls/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/china-recalls/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1863+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/china_search_recalls \
  -H 'Content-Type: application/json' \
  -d '{"query":"充电宝","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/china_search_recalls`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-china-recalls` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/china-recalls/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "china-recalls": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-china-recalls"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about China Recalls data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
