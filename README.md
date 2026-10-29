# @pipeworx/globenewswire

Live press releases from GlobeNewswire's public-company RSS feed — headline,
issuer, tickers, publish time and excerpt, newest first.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `globenewswire_latest_releases(since_hours?, limit?)` — the current rolling
  window of releases, newest first. `newest_item_at` tells a caller whether
  the feed is quiet or stalled.
- `globenewswire_search_releases(query, limit?)` — keyword/company match over
  headline, issuer and excerpt, within the current window only.
- `globenewswire_get_release(url)` — one specific release, matched by the URL
  a prior call returned. Fails clearly (not silently empty) if the release has
  scrolled out of the current window.

## Auth

Keyless. No account, no clickthrough terms.

## Data sources

- <https://www.globenewswire.com/RssFeed/orgclass/1/feedTitle/GlobeNewswire%20-%20News%20About%20Public%20Companies>
  — GlobeNewswire's own channel description calls this "the last 20 releases".
  It is the **public-company** org class only (org class 1); GlobeNewswire
  also distributes private-company, nonprofit and mutual-fund releases under
  other org classes this pack does not read.

Every call re-fetches this same rolling ~20-item window live — it is **not**
an archive. A release typically scrolls off within about an hour at the
wire's normal volume, so `search_releases` only ever searches what is
currently in the window and `get_release` can miss something that has
already fallen out of it (the tool says so explicitly rather than returning
an empty result).

The issuer name is pulled from `<dc:contributor>`, not from a generic
`<author>`/byline heuristic — this feed carries no `<author>` tag, so a
generic byline reader would return nothing useful here.

PR Newswire sits behind Cloudflare bot management and occasionally answers a
request with a challenge redirect that 404s; this pack retries up to 3 times
before reporting the wire down (see `mcps/prnewswire/src/index.ts` for the
measured rate — the same defensive retry is applied here too, even though it
has not been observed on this feed specifically).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "globenewswire": {
      "url": "https://gateway.pipeworx.io/globenewswire/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/globenewswire/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/globenewswire_latest_releases \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/globenewswire_latest_releases`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-globenewswire` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/globenewswire/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "globenewswire": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-globenewswire"]
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
ask_pipeworx({ question: "your question about Globenewswire data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
