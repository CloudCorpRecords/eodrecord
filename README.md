# eodrecord

eodrecord gives an agent a read window onto a
rules-based index model's published record. The model
tracks SPY and QQQ; what it held and how it was
allocated at each close is published as an end-of-day
record. The free view of that record runs 90 days
behind the live one, and this package is how an agent
reads it: the delayed record itself, the fixed
definitions behind every published field, the ledger
fingerprint that lets anyone check the record was not
rewritten after the fact, and a pointer to the current
published state.

This package holds no model data of its own. It answers
the tool list locally, exactly as captured from the
live endpoint, and forwards each tool call upstream,
returning whatever comes back untouched. All four
tools are read-only views over published material.
None of them buys, subscribes, mints a key, or sends
anything anywhere.

## Tools

- `get_public_record` — the published end-of-day model
  states and allocation changes, shown with a 90-day
  delay (the data behind the vendor's public record
  page). Published states only; nothing is recomputed.
  Optional: `days`, an integer from 1 to 260 naming how
  many of the most recent sessions to return (default
  30). No argument is required.
- `get_definitions` — the fixed definitions of every
  published field: market regime, allocation states,
  extension, distance to next signal, and signals,
  together with the publication schedule. Takes no
  arguments.
- `get_verification` — the latest public ledger
  fingerprint with its RFC 3161 and OpenTimestamps
  receipts, plus how to verify the record
  independently. Takes no arguments.
- `get_current_state` — where the current published
  state lives. Core members receive it the evening it
  is published; without a key this returns how Core
  access works and where the delayed public record is.
  Takes no arguments.

## Configuration

- `TIDELINE_MCP_URL` (optional) — override the
  upstream endpoint. Defaults to
  `https://tideline.jbpscapital.com/mcp`. (The override
  hook is this rebuild's own labelled reconstruction.)
- `TIDELINE_API_KEY` (optional) — when set, every
  upstream request carries an
  `Authorization: Bearer <key>` header, for the
  caller's own key. When it is not set, no key header
  is sent. (The header name is pinned by the original's
  registry record, which declares a secret
  `Authorization` header that is not marked required;
  the record pins no package env name, so the env name
  and the Bearer forwarding are labelled
  reconstructions — the original surface was captured
  keyless.)

### Forwarding mechanics

On the first tool call the client opens a streamable
HTTP session upstream: a JSON-RPC `initialize`
request (protocolVersion `2025-03-26`, clientInfo
name `eodrecord`), then a
`notifications/initialized` notification, then the
`tools/call` request carrying the caller's arguments
unchanged. When the upstream answers `initialize`
with an `Mcp-Session-Id` header, that value rides on
every later request of the session, which is reused
for the life of the process. Replies may arrive as
plain JSON or as server-sent events; both are read.

Successful upstream results pass through verbatim,
including any `isError` flag the upstream sets.
Failures — a JSON-RPC error, a non-2xx status, an
unreadable response, or a request that cannot be
completed — come back as an `isError` result whose
text starts with `eodrecord error:` (this rendering
is this rebuild's own labelled reconstruction).

## CLI

Each tool is also a subcommand taking one JSON
argument string:

```bash
npx @reneturcios/eodrecord get_public_record '{"days":30}'
npx @reneturcios/eodrecord get_definitions
npx @reneturcios/eodrecord get_verification
npx @reneturcios/eodrecord get_current_state
npx @reneturcios/eodrecord --version
```

Exit codes: `0` on success, `1` on an `isError`
result, `2` on a usage error.

## MCP server

```bash
npx -y @reneturcios/eodrecord mcp
```

MCP client config:

```json
{
  "mcpServers": {
    "eodrecord": {
      "command": "npx",
      "args": ["-y", "@reneturcios/eodrecord", "mcp"]
    }
  }
}
```

## Docker

```bash
docker run -i ghcr.io/cloudcorprecords/eodrecord
docker run --rm ghcr.io/cloudcorprecords/eodrecord --version
```

The container's default command is `mcp`, so the first
form is a stdio MCP server; any other arguments reach
the CLI normally.

## Development

```bash
npm install
npm run build
npm test
npm run dev
```

The test suite runs entirely against a stubbed
JSON-RPC upstream — no live network calls.

## License

MIT — see `LICENSE`.
