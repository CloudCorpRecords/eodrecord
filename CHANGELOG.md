# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Tideline MCP server, serving the four captured tools locally for tools/list and forwarding every tools/call upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when issued).
- Tools: `get_public_record`, `get_definitions`, `get_verification`, and `get_current_state`. The surface is read-only published data only; no tool in it buys, subscribes, mints a key, or sends anything.
- Upstream results pass through verbatim; upstream failures surface as `eodrecord error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). Upstream endpoint overridable via `TIDELINE_MCP_URL`; when `TIDELINE_API_KEY` is set it is sent as an `Authorization: Bearer` header.
