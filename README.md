# grahamvalue

What is the highest price Benjamin Graham's formula would justify paying for a stock? grahamvalue is a local forwarding client for the hosted Graham Number calculator MCP service: it answers `tools/list` locally from the captured tool definition and forwards every call to the service, returning the answers exactly as the service sends them back.

The one tool computes the Graham Number — sqrt(22.5 × EPS × book value per share) — the fair-value ceiling from Graham's defensive-investor framework, together with the margin of safety against the current price. Pass a US-listed ticker and the service pulls EPS, book value per share, and price from SEC EDGAR filings and market data on its own. Or skip the ticker entirely and pass `eps`, `bvps`, and `price` directly to value a hypothetical company. Every argument is optional; there is no write, signup, or send tool in this surface.

## The tool

- **`compute_graham_number`** — The Graham Number fair-value ceiling and margin of safety. Give it a `ticker` (for example `AAPL`) for a live lookup, or pass `eps` / `bvps` / `price` numbers directly. No argument is required by the schema.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/grahamvalue mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "grahamvalue": {
      "command": "npx",
      "args": ["-y", "@reneturcios/grahamvalue", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/grahamvalue
docker run --rm ghcr.io/cloudcorprecords/grahamvalue compute_graham_number '{"ticker":"AAPL"}'
```

## CLI

One binary serves both the command line and the MCP server. The tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
grahamvalue compute_graham_number '{"ticker":"AAPL"}'
grahamvalue compute_graham_number '{"eps":4.5,"bvps":38.2,"price":95}'
grahamvalue --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `GRAHAMVALUE_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://www.edgethirteen.com/api/mcp/graham-number-calculator`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `grahamvalue error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the 1 tool, deep-equal, with no `title` key served — the capture carries none — and no `required` key on the input schema, exactly as captured), exercises the session handshake and its reuse, verifies that no `Authorization` or `X-API-Key` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
