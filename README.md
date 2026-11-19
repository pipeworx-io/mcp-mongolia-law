# @pipeworx/mongolia-law

Mongolian legislation: search the laws and other legal acts of Mongolia by title (Mongolian Cyrillic) and read a law or a single article of it, with the register's status, citation and `data_as_of`. Keyless; every tool is a live call per request.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2015+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `mn_law_search(query, category?, match?, limit?)` -- find acts by words in the title, for example `{"query":"Хөдөлмөрийн тухай","category":"law"}` for the Labour Law. `category` narrows to laws, the constitution, treaties, presidential decrees, parliament and government resolutions, court decisions, ministerial orders and local council decisions. Returns `law_id`, title, category, approval date and effective date. Lists only acts the register marks valid.
- `mn_law_get(law_id, article?)` -- an act's title, type, adoption date, status and list of articles (зүйл); with `article` (for example `"5"`), the Mongolian text of that article. `found:false` with a reason when the act or article is not found.

## Auth

Keyless.

## Data sources

- Ministry of Justice and Home Affairs of Mongolia, Unified Legal Information System (Эрх зүйн мэдээллийн нэгдсэн систем) -- the official register of Mongolian laws and legal acts (public, no login).

Every call is read live; nothing is stored. Text is in Mongolian; the system's English translations are not read.

Traps worth knowing:

- Search is by title words in Cyrillic, case-insensitive; a word in another form (an inflected ending) may not match. `match:"any"` widens it.
- Status: the act page states none. Search lists only acts the register's validity filter marks valid; its other value returns nothing, so a repealed act cannot be found through search. `mn_law_get` looks the act's title up under that same filter: listed means `status:"in_force"` with the statement quoted, otherwise `status` is `null` and the note says so. Nothing else is derived. An article the register has marked removed keeps its own note in the text.
- Long acts are large pages (the Labour Law is about 2 MB), so `mn_law_get` takes a few seconds.
- `data_as_of` is the day of the read; the register publishes no per-act refresh date.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "mongolia-law": {
      "url": "https://gateway.pipeworx.io/mongolia-law/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/mongolia-law/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2015+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/mn_law_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"Хөдөлмөрийн тухай","category":"law","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/mn_law_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-mongolia-law` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/mongolia-law/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "mongolia-law": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-mongolia-law"]
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
ask_pipeworx({ question: "your question about Mongolia Law data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
