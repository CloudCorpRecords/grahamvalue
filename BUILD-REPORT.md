# BUILD REPORT — grahamvalue

Clean-room rebuild of `com.edgethirteen.www/graham-number-calculator`
v1.0.0 (remote-only MCP server at
`https://www.edgethirteen.com/api/mcp/graham-number-calculator`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/grahamvalue` 0.1.0, registry name
`io.github.CloudCorpRecords/grahamvalue`.

## Surface built

Full surface: **1 of 1 captured tools built**, none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `compute_graham_number` | *(none — the captured inputSchema carries NO `required` key at all; all four properties — `ticker`, `eps`, `bvps`, `price` — are optional, served verbatim)* |

The served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO
`title` key on the tool and none is served, and the inputSchema
carries NO `required` key and none was invented. The tool literal
was generated programmatically from
`harness/fixtures/graham-tools.json` by the script at
`/tmp/b51-grahamvalue-scratch/gen.py` (parse + re-serialise in
fixture key order; the generator's round-trip assertions confirmed
the emitted literal parses back to the fixture's tool object, and a
separate check confirmed the literal embedded in `src/forward.ts`
deep-equals the fixture's tool object) — never retyped — and the
registration-integrity test suite plus the parity harness
confirmed the served literal deep-equals the fixture's tool
object. Tools are registered via the MCP SDK low-level Server
API (`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tool's name,
description, property set/order and shapes, the absence of a
`title` key, and the absence of a `required` key on the
inputSchema.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned:
  `https://www.edgethirteen.com/api/mcp/graham-number-calculator`.
- Env override `GRAHAMVALUE_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization`
  header is sent under any environment — including with key-like
  env vars (`GRAHAMVALUE_API_KEY`, `API_KEY`, `AUTHORIZATION`)
  set — and that no `X-API-Key` header is ever sent; upstream
  requests carry only `Content-Type`, `Accept` and, when issued,
  `Mcp-Session-Id`.
- Error rendering `grahamvalue error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including
  `structuredContent`, `isError` and extra fields. tools/call is
  forwarded as streamable-HTTP JSON-RPC: initialize
  (protocolVersion `2025-03-26`) → notifications/initialized →
  tools/call, honouring `Mcp-Session-Id` when issued; plain-JSON
  and SSE replies are both parsed. The forwarding core takes an
  injectable fetch; tests stub the upstream — no live network
  calls in tests.

## Deviations

**None.** The offline lockfile route (Route 3) succeeded on the
first attempt, and the install / typecheck / build / test /
parity chain ran in-session with no write-guard interference.

## Lockfile

- Route: **Route 3 (offline)** —
  `npm install --package-lock-only --ignore-scripts --offline`,
  generated clean in the build dir with no `node_modules` and no
  pre-existing lockfile.
- Parses as JSON; `name` == `@reneturcios/grahamvalue`.
- Package entries: **128**.
- sha256: `8ebfb49883212604b6ae630c27e07801e5f1fc9125ffe5ecc81f58724a83f97e`
  (re-verified unchanged after the subsequent full `npm install`).

## server.json

- Description exactly: "Graham Number fair-value ceiling for a US ticker from SEC EDGAR EPS and book value per share." —
  **93 characters** (measured from the file).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/grahamvalue:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/grahamvalue"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/grahamvalue`
  and its smoke test asserts the server name `grahamvalue`.

## Verification (all run 2026-10-28)

1. `tsc --noEmit` → exit 0.
2. `npm run build` (tsc) → exit 0; real `dist/` produced (cli,
   forward, index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. Tests (`npm test` → `tsx --test test/forward.test.ts`,
   fake-fetch only) → **55 tests, 55 pass, 0 fail**
   (registration-integrity suite deep-equals the served
   tools/list against the fixture's tool, including the
   NO-`required`-key and NO-`title`-key assertions and the
   four-property order/shape checks, plus session
   handling/reuse, env override, the no-Authorization /
   no-X-API-Key assertions under any environment, verbatim
   argument forwarding — ticker and direct eps/bvps/price —
   and result pass-through including structuredContent and
   upstream isError, SSE parsing, error rendering for both the
   JSON-RPC-error and transport-failure paths, MCP-layer
   validation, and CLI behaviour including a working
   subcommand for the tool against a local stub server).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/graham-tools.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/grahamvalue/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/grahamvalue/parity-out` → **verdict PASS, parity 100.0**
   (1/1 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
