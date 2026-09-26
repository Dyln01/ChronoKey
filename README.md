@'
# MCP Worker

A paid MCP (Model Context Protocol) server running on Cloudflare Workers, with x402 micropayments settled in USDC.

## What This Does

This Worker exposes MCP tools over Streamable HTTP. One tool is free (`health`), one requires payment (`lookup`). Payment is handled via the x402 protocol — a machine-native payment standard that uses the HTTP `402 Payment Required` status code.

Any MCP-compatible client (Claude Desktop, Cursor, or a custom agent) can connect and call the tools. Paid tools follow the standard x402 handshake: send the request, receive a 402 with payment details, settle USDC on-chain, retry with proof of payment.

## Endpoint

https://cloudflare-mcp-worker.dylanrenovos.workers.dev/mcp

- Transport: Streamable HTTP
- Method: POST
- Required header: Accept: application/json, text/event-stream
- Content-Type: application/json

## Tools

### health (free)

Returns "OK". Use it to verify the server is reachable.

curl -X POST https://cloudflare-mcp-worker.dylanrenovos.workers.dev/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"health","arguments":{}}}'

### lookup (paid - $0.02 USDC)

Returns a record for a given ID.

Input: { "id": "string" }

Example call:

curl -X POST https://cloudflare-mcp-worker.dylanrenovos.workers.dev/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"lookup","arguments":{"id":"test-123"}}}'

## Pricing

| Tool | Price | Currency |
|------|-------|----------|
| health | Free | - |
| lookup | $0.02 | USDC |

## Supported Payment Networks

| Network | CAIP-2 | USDC Contract |
|---------|--------|---------------|
| Base | eip155:8453 | 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 |
| Polygon PoS | eip155:137 | 0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359 |
| Arbitrum One | eip155:42161 | 0xaf88d065e77c8cC2239327C5EDb3A432268e5831 |

All three settle to: 0xAb59e91c7A4e280914681FA8eA2015f2e1826b4f

## How x402 Payment Works

1. Send the request without payment.
2. Receive a 402 response with a PAYMENT-REQUIRED header.
3. Sign a USDC transfer (gasless, EIP-3009).
4. Retry the request with the X-PAYMENT header.
5. Receive the tool result once settled.

x402 clients handle steps 2-4 automatically.

## Discovery

- /.well-known/x402 - discovery manifest
- /openapi.json - OpenAPI 3.1 spec with x-payment-info

## Connecting an MCP Client

Claude Desktop / Cursor config:

{
  "mcpServers": {
    "cloudflare-mcp-worker": {
      "url": "https://cloudflare-mcp-worker.dylanrenovos.workers.dev/mcp"
    }
  }
}

## Local Development

npm install
npx wrangler dev
npx wrangler tail
npx wrangler deploy

## Stack

- Cloudflare Workers
- Cloudflare Agents SDK
- x402 protocol
- Coinbase CDP Facilitator
- Model Context Protocol

## License

MIT
'@ | Out-File -FilePath README.md -Encoding utf8
