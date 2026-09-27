<div align="center">

<img src="logo.png" alt="ChronoKey logo" width="160" />

# ChronoKey

Timestamps, UUIDs, gas prices, domain intelligence, agent memory, and text-to-structure extraction for AI agents. Paid via x402 or MPP.

[View full pricing](https://bot.chronokey.workers.dev/.well-known/mcp-pricing)

</div>

---

## What This Is

ChronoKey is a hosted MCP server that provides 15 utility tools AI agents need on nearly every run. Payment is handled via the x402 protocol (USDC on Base, Polygon, or Arbitrum) or MPP (pathUSD on Tempo). Any MCP-compatible client can connect and call the tools.

## Endpoint

    https://bot.chronokey.workers.dev/mcp

- Transport: Streamable HTTP
- Method: POST
- Required header: Accept: application/json, text/event-stream
- Content-Type: application/json

## Tools

### health (free)

Returns `"OK"`. Use it to verify the server is reachable.

### timestamp

Returns the current time in multiple formats.

Input: `{ "timezone": "America/New_York" }` — timezone is optional (IANA name). Defaults to UTC.

Pricing: First 10 calls per day per session free, then Pro subscription, then $0.001 USD.

### uuid

Generates 1-100 UUIDs.

Input: `{ "count": 5, "version": "v7" }`

- count (optional): 1-100, defaults to 1
- version (optional): "v4" (random) or "v7" (time-ordered), defaults to "v4"

Pricing: First 10 calls per day per session free, then Pro subscription, then $0.001 USD.

### text.structure

Extracts structured entities from unstructured text into a strict, stable JSON schema.

Input:

    {
      "text": "Contact alice@example.com on 2026-09-26 about the $42 invoice.",
      "mode": "standard",
      "include_referrals": false
    }

- mode (optional): "standard" (default) or "deep"
- include_referrals (optional): append complementary service suggestions

Deep output adds currencies, organizations, persons, and embedded JSON blobs.

Pricing: Standard $0.008 USD (Pro-covered), Deep $0.020 USD.

### text.structure.deep

Convenience alias for text.structure with mode "deep". Costs $0.020 USD.

### extract.and.fetch

Extracts entities then fetches the first URL found and returns its content. $0.012 USD.

### extract.and.summarize

Extracts entities then returns a grouped summary. $0.010 USD.

### extract.verified

Verified extraction. Returns standard entities plus a verification_token. $0.005 USD base.

### extract.confirm

Confirms whether a previous extract.verified result was accurate. $0.010 USD on accurate=true. Free on accurate=false.

### gas.price

Real-time EIP-1559 gas prices across Base, Polygon, Arbitrum, Ethereum, and Optimism. Pro-covered or $0.003 USD.

### domain.intel

One call returns WHOIS, DNS, and SSL certificate data for any domain. Pro-covered or $0.005 USD.

### memory.store

Stores a value under a namespaced key with optional TTL. Pro-covered or $0.005 USD.

### memory.recall

Retrieves a value by key. Pro-covered or $0.005 USD.

### memory.search

Full-text search across a namespace using SQLite FTS5 with BM25 ranking. $0.010 USD.

### subscribe

Subscribe to ChronoKey Pro for $5.00 USD for 30 days. Covers timestamp, uuid, text.structure (standard mode), gas.price, domain.intel, memory.store, and memory.recall.

### subscription.status

Check the status of the Pro subscription bound to the current MCP session. Free.

## Pricing Summary

| Tool | Price (USD) | Pro-covered | Free tier |
|---|---|---|---|
| health | Free | - | - |
| timestamp | $0.001 | yes | 10/day |
| uuid | $0.001 | yes | 10/day |
| text.structure (standard) | $0.008 | yes | - |
| text.structure (deep) | $0.020 | no | - |
| extract.and.fetch | $0.012 | no | - |
| extract.and.summarize | $0.010 | no | - |
| extract.verified | $0.005 | no | - |
| extract.confirm | $0.010 | no | - |
| gas.price | $0.003 | yes | - |
| domain.intel | $0.005 | yes | - |
| memory.store | $0.005 | yes | - |
| memory.recall | $0.005 | yes | - |
| memory.search | $0.010 | no | - |
| subscribe | $5.00 / 30d | - | - |
| subscription.status | Free | - | - |

## Freemium

The first 10 calls per day per MCP session are free for timestamp and uuid.

## Pro Subscription

$5.00 USD for 30 days. Covers timestamp, uuid, text.structure (standard mode), gas.price, domain.intel, memory.store, and memory.recall. Reuse the same MCP session across calls to benefit.

## Supported Payment Protocols

**x402:** USDC on Base (eip155:8453), Polygon (eip155:137), or Arbitrum (eip155:42161).

**MPP:** pathUSD on Tempo.

## How Payment Works

1. Send the request without payment.
2. Receive a 402 with both an x402 challenge (PAYMENT-REQUIRED header) and an MPP challenge (WWW-Authenticate header).
3. Sign a payment authorization.
4. Retry with the credential.
5. Receive the tool result once the payment is settled.

## Discovery

- /.well-known/x402 — discovery manifest
- /.well-known/mcp-pricing — tool pricing
- /.well-known/mcp/server-card.json — Smithery server card
- /.well-known/agent-card.json — A2A agent card
- /openapi.json — OpenAPI 3.1 spec
- /llms.txt — plain-text summary for agent frameworks

## Connecting an MCP Client

### Claude Desktop / Cursor

    {
      "mcpServers": {
        "chronokey": {
          "url": "https://bot.chronokey.workers.dev/mcp"
        }
      }
    }

### MCP Inspector

    npx @modelcontextprotocol/inspector@latest

Enter the endpoint URL and click Connect.

## Support

For questions, issues, or custom integrations, contact the maintainer.

## Terms

ChronoKey is a proprietary hosted service. All rights reserved.
