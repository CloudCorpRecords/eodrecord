# BUILD REPORT — eodrecord

Rebuild of `com.jbpscapital.tideline/tideline` v1.3.0 (remote-only)
as `@reneturcios/eodrecord` 0.1.0 / `io.github.CloudCorpRecords/eodrecord`.
**Full surface built: all 4 captured tools (4/4).**

## Fixture conflicts

**None.** The spec, the brief, and the fixture agree on the four tool
names, titles, descriptions, and schemas. Required fields as served,
taken from the fixture (`harness/fixtures/tideline-tools-original.json`):

- `get_public_record` — NO `required` key in its schema (optional `days` only)
- `get_definitions` — NO `required` key in its schema (empty `properties`)
- `get_verification` — NO `required` key in its schema (empty `properties`)
- `get_current_state` — NO `required` key in its schema (empty `properties`)

All four captured schemas carry exactly `type`, `properties`, and
`additionalProperties` (false) keys and no `required` key; they are
served verbatim. The fixture's tool literals (name, title, description,
inputSchema, in fixture order) are served verbatim from
`src/forward.ts`; the literals were generated programmatically from
the fixture JSON (parsed and re-serialised by
`/tmp/b39-eodrecord-gen.py`, spliced by `/tmp/b39-eodrecord-splice.py`,
never retyped) and verified by the registration-integrity test (served
tools/list deep-equals the fixture's 4 tool objects, in order) and
again by the parity harness.

Judgment call recorded in the spec, followed here: the surface is
entirely read-only published data. `get_current_state` without a key
returns how Core access works and where the delayed public record is;
it is not a checkout, signup, or key-minting tool, and no tool in the
surface buys, subscribes, mints, or sends anything. The surface is
built in full.

## What was built

All 19 factory files under `builds/eodrecord/`:

- `package.json` (`@reneturcios/eodrecord` 0.1.0, bin
  `eodrecord` → `dist/cli.js`, deps exactly `@modelcontextprotocol/sdk ^1`
  + `zod ^3`; devDeps `@types/node ^20`, `tsx ^4`, `typescript ^5`)
- `server.json` (description exactly the prescribed 77-char string
  "Read the published end-of-day state of a rules-based SPY and QQQ index model.";
  npm entry carries `"version": "0.1.0"`; OCI entry
  `ghcr.io/cloudcorprecords/eodrecord:0.1.0` carries **no** version field)
- `Dockerfile` (node:20-alpine both stages; server-name label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/eodrecord"`
  matches `server.json`)
- `tsconfig.json` (fleet-identical, 318 bytes), `.gitignore`,
  `package-lock.json`, `README.md` (fresh prose), `LICENSE` (MIT,
  Copyright (c) 2026 Rene Turcios), `CHANGELOG.md` (single 0.1.0 entry,
  2026-10-28)
- `src/index.ts`, `src/cli.ts`, `src/mcp.ts`, `src/types.ts`,
  `src/forward.ts` (core; exports `VERSION = "0.1.0"`)
- `test/forward.test.ts`
- `.github/workflows/{ci.yml, publish.yml, publish-container.yml,
  registry-publish.yml}` (container workflow: IMAGE
  `ghcr.io/cloudcorprecords/eodrecord`, smoke assertion on server name
  `eodrecord`)

Plus `BUILD-REPORT.md` (this file) and the `parity-out/` harness output.

## Design

- Forwarding client to `https://tideline.jbpscapital.com/mcp` (pinned),
  overridable via `TIDELINE_MCP_URL` (LABELLED RECONSTRUCTION).
  tools/call is forwarded as a streamable-HTTP JSON-RPC client:
  initialize (protocolVersion `2025-03-26`, clientInfo `eodrecord`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; session established lazily and reused per client.
  Plain-JSON and SSE replies both parsed; the matching-id message
  wins; upstream results pass through verbatim (including `isError`
  and extra fields such as `structuredContent`).
- When `TIDELINE_API_KEY` is set, an `Authorization: Bearer <value>`
  header is sent on every upstream request (header NAME pinned by the
  registry record's own remote declaration, which declares a secret
  `Authorization` header not marked required; the env name, the Bearer
  form, and the env forwarding are LABELLED RECONSTRUCTIONS — the
  record pins no package env name and the capture was keyless). When
  unset (or blank), no key header is sent. No `X-API-Key` header is
  sent in either case. Both branches are covered by tests.
- Upstream failures (JSON-RPC error, non-2xx status, malformed body,
  throwing/timing-out fetch) surface as `eodrecord error:` text results
  (LABELLED RECONSTRUCTION).
- Registration via the MCP SDK **low-level** `Server` API
  (`ListToolsRequestSchema` / `CallToolRequestSchema` handlers)
  serving the fixture literals exactly; no `McpServer` + zod
  registration (per the task brief, overriding TEMPLATE.md §2 item 12).
  `src/mcp.ts` also exports `createServer` / `startMcpServer` /
  `handleCallTool` so the MCP layer is testable without a transport;
  the CLI's `mcp` subcommand starts it over stdio.
- Core takes an injectable fetch (`ForwardDeps.fetchFn`), so all tests
  run against a stubbed JSON-RPC upstream. No live network calls in
  tests; no live call claimed beyond the coordinator's tools/list
  captures.

## Labelled reconstructions (from the spec's Wire section)

- `TIDELINE_MCP_URL` env override of the pinned endpoint
  `https://tideline.jbpscapital.com/mcp`.
- `TIDELINE_API_KEY` env name and the forwarding of its value as
  `Authorization: Bearer <value>` (header name pinned by the registry
  record; env name pinned by nothing, capture keyless).
- The `eodrecord error:` failure rendering for upstream failures.

## Lockfile (Route 3, clean from birth)

Generated in the build dir with no `node_modules` present and no
pre-existing lockfile:
`npm install --package-lock-only --ignore-scripts --offline`.
Verified: parses, `name == @reneturcios/eodrecord`,
**128 package entries**, sha256
`e424dd45952ce16fafea2d79563100b1bd6c7d16dca53ae4ea9de1a726ee74c9`.
Then `npm install --offline --ignore-scripts` in the build dir for
`node_modules` (lockfile sha256 re-verified unchanged afterwards).

## Verification (all in the build dir unless noted)

- `npx tsc --noEmit` → exit 0
- `npm run build` → real `dist/` (cli.js, forward.js, mcp.js, all
  non-empty, with `.d.ts` + source maps)
- `node dist/cli.js --version` → `0.1.0`; unknown subcommand → usage on
  stderr, exit 2
- `npm test` → **60 tests, 60 pass, 0 fail** (8 suites: registration
  integrity — served tools deep-equal the fixture's full 4 tool
  objects, names/titles in order, exactly the fixture's keys per tool,
  no `required` key in any served schema, schema key set and
  `additionalProperties: false`, days bounds/wording, description
  wording, isKnownTool, pinned constants; session
  establishment/reuse/session-id handling, clientInfo/protocol
  version; env URL override, explicit url override, explicit-over-env
  precedence, blank-env fallback; Authorization Bearer present on
  every request when the key is set / absent when unset or blank /
  no X-API-Key header in either case; verbatim argument forwarding
  for all four tools incl. days at the 260 bound; verbatim result
  pass-through incl. upstream isError and SSE, distinct request ids;
  JSON-RPC/init/malformed/no-result/HTTP/throw/timeout failures →
  `eodrecord error:`; parseRpcBody JSON + SSE + matching-id + [DONE]
  cases; MCP-layer MethodNotFound / InvalidParams validation incl.
  null arguments; CLI --version / -v / unknown exit 2 / bad-JSON
  exit 2 / `mcp extra` exit 2)
- Parity: `python3 harness/parity.py --original
  harness/fixtures/tideline-tools-original.json --rebuild
  "stdio:node builds/eodrecord/dist/cli.js mcp" --out
  builds/eodrecord/parity-out` (run from `~/workspace/mcp-rebuild-100`)
  → **PASS, parity 100.0, 4/4 identical, no missing/extra/schema
  diffs** (see `parity-out/parity.json`)
- Consistency sweep: package.json version = server.json top-level =
  npm-entry version = OCI tag = core `VERSION` = CHANGELOG heading
  (0.1.0); package.json `mcpName` = server.json `name` = Dockerfile
  label; server.json description measured at 77 chars; OCI entry
  confirmed to have no `version` field.

Clean-room: all prose and code written fresh for eodrecord; only the
fleet mechanics were patterned on the reference forwarding-client
build, and only the fixture's tool literals are served verbatim.
