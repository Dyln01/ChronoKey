# ChronoKey

Reliable timestamps and UUIDs for AI agents, with x402 USDC settlement.

## What This Is

ChronoKey is a paid MCP server running on Cloudflare Workers. It provides two utility tools that AI agents need on nearly every run: reliable time context and unique identifier generation. Payment is handled via the x402 protocol — a machine-native payment standard using the HTTP `402 Payment Required` status code.

## Endpoint

    https://cloudflare-mcp-worker.dylanrenovos.workers.dev/mcp

- Transport: Streamable HTTP
- Method: POST
- Required header: Accept: application/json, text/event-stream
- Content-Type: application/json

## Tools

### health (free)

Returns "OK". Use it to verify the server is reachable.

### timestamp ($0.001 USDC)

Returns the current time in multiple formats.

Input: { "timezone": "America/New_York" }

timezone is optional (IANA name). Defaults to UTC.

Output:

    {
      "iso8601": "2026-09-26T15:30:00.000Z",
      "unixSeconds": 1790427000,
      "unixMilliseconds": 1790427000000,
      "timezone": "America/New_York",
      "humanReadable": "Friday, September 26, 2026 at 11:30:00 AM EDT"
    }

### uuid ($0.001 USDC)

Generates 1-100 UUIDs.

Input: { "count": 5, "version": "v7" }

- count (optional): 1-100, defaults to 1
- version (optional): "v4" (random) or "v7" (time-ordered), defaults to "v4"

Output:

    {
      "version": "v7",
      "count": 5,
      "uuids": [
        "018f3c8a-1e2b-7c4d-8f1a-2b3c4d5e6f70",
        "..."
      ]
    }

## Pricing

| Tool | Price | Currency |
|------|-------|----------|
| health | Free | - |
| timestamp | $0.001 | USDC |
| uuid | $0.001 | USDC |

## Supported Networks

| Network | CAIP-2 | USDC Contract |
|---------|--------|---------------|
| Base | eip155:8453 | 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 |
| Polygon PoS | eip155:137 | 0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359 |
| Arbitrum One | eip155:42161 | 0xaf88d065e77c8cC2239327C5EDb3A432268e5831 |

All three settle to: 0xAb59e91c7A4e280914681FA8eA2015f2e1826b4f

## How x402 Payment Works

1. Send the request without payment.
2. Receive a 402 with a PAYMENT-REQUIRED header. Decode (Base64) to see the accepts[] array.
3. Sign a USDC transferWithAuthorization (EIP-3009). Gasless - the facilitator pays the gas.
4. Retry with the X-PAYMENT header.
5. Receive the tool result once the facilitator verifies and settles.

x402 clients handle steps 2-4 automatically.

## Discovery

- /.well-known/x402 - discovery manifest
- /.well-known/mcp-pricing - tool pricing
- /openapi.json - OpenAPI 3.1 spec
- /llms.txt - plain-text summary for agent frameworks

## Connecting an MCP Client

Claude Desktop / Cursor config:

    {
      "mcpServers": {
        "chronokey": {
          "url": "https://cloudflare-mcp-worker.dylanrenovos.workers.dev/mcp"
        }
      }
    }

## Local Development

    npm install
    npx wrangler dev
    npx wrangler tail
    npx wrangler deploy

## Project Structure

    .
    ├── src/
    │   └── index.ts
    ├── public/
    │   ├── favicon.ico
    │   ├── llms.txt
    │   ├── openapi.json
    │   └── .well-known/
    │       ├── x402
    │       └── mcp-pricing
    ├── package.json
    ├── package-lock.json
    ├── tsconfig.json
    └── wrangler.jsonc

## Stack

- Cloudflare Workers
- Cloudflare Agents SDK
- x402 protocol
- Coinbase CDP Facilitator
- Model Context Protocol

## License

MIT
'@

$content | Out-File -FilePath README.md -Encoding utf8
Write-Host "README.md created at $(Get-Location)\README.md"
