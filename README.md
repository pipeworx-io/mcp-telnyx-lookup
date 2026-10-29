# @pipeworx/telnyx-lookup

Carrier, caller-name (CNAM) and number-portability (LRN, ported status, line
type) data for one E.164 phone number, plus Telnyx's fraud flag, from the Telnyx
Number Lookup API.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `telnyx_number_lookup(phone_number, type?, _apiKey)` — one number in E.164
  form (`+18665552368`). Returns the carrier name and line type
  (mobile/landline/voip), the CNAM caller name, and portability details (LRN,
  ported status and date, OCN/SPID, city/state). `type` selects `carrier`
  and/or `caller-name`; both are requested by default and portability data is
  always included.

## Auth

BYO only. `_apiKey` is required in the tool schema, so the gateway refuses
with `error_class: auth_required` ("requires an API key") before any upstream
request is made. Pipeworx does not front a Telnyx key for this pack — see
`scripts/vendors/intent.json` → `telnyx-lookup` for the decision, the pay
trigger, and where the Telnyx key minted on 2026-10-01 actually lives (it
belongs to another product and is deliberately not shared).

Telnyx gates the API beyond registration: a new account must complete
identity verification and add a payment method before Number Lookup answers.
Until then every call returns an auth-shaped error that looks identical to a
wrong key. Create keys in the Mission Control portal under API Keys.

## Data sources

- <https://api.telnyx.com/v2/number_lookup/{phone_number}?type=carrier&type=caller-name>
  — the single endpoint. `type` is a repeatable query parameter with the
  hyphenated enum values `carrier` and `caller-name`. The phone number is a
  path segment and the leading `+` must be URL-encoded. Errors come back as
  `{ errors: [{ code, title, detail }] }`; the pack surfaces the first
  `detail`.
- Pricing (read live 2026-10-08 from <https://telnyx.com/ai/pricing.json>):
  LRN $0.0015, MCC/MNC $0.0025, CNAM $0.003 per query, prepaid, no minimum.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "telnyx-lookup": {
      "url": "https://gateway.pipeworx.io/telnyx-lookup/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/telnyx-lookup/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/telnyx_number_lookup`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-telnyx-lookup` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/telnyx-lookup/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "telnyx-lookup": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-telnyx-lookup"]
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
ask_pipeworx({ question: "your question about Telnyx Lookup data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
