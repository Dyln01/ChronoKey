# ChronoKey

Timestamps, UUIDs, and text-to-structure extraction for AI agents, with x402 and MPP payment support.

## What This Is

ChronoKey is a hosted MCP server that provides three utility tools AI agents need on nearly every run: reliable time context, unique identifier generation, and deterministic text-to-structure extraction. Payment is handled via x402 (USDC on Base, Polygon, or Arbitrum) or MPP (pathUSD on Tempo). Any MCP-compatible client can connect and call the tools.

## Endpoint

    https://cloudflare-mcp-worker.dylanrenovos.workers.dev/mcp

- Transport: Streamable HTTP
- Method: POST
- Required header: Accept: application/json, text/event-stream
- Content-Type: application/json

## Tools

### health (free)

Returns "OK". Use it to verify the server is reachable.

### timestamp ($0.001 USD)

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

### uuid ($0.001 USD)

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

### text.structure ($0.008 USD)

Extracts structured entities from unstructured text and returns a strict, stable JSON schema.

Input: { "text": "Contact alice@example.com on 2026-09-26 about the $42 invoice. See https://example.com for details. #finance @team" }

Output:

    {
      "emails": ["alice@example.com"],
      "urls": ["https://example.com"],
      "phones": [],
      "dates": ["2026-09-26"],
      "numbers": [
        { "raw": "42", "value": 42, "type": "integer", "negative": false }
      ],
      "ipAddresses": [],
      "hashes": [],
      "mentions": ["@team"],
      "hashtags": ["#finance"],
      "stats": {
        "wordCount": 15,
        "charCount": 118,
        "lineCount": 1
      }
    }

The output schema is deterministic and stable. Every field is always present — empty arrays when nothing matches. There are no external API calls, no LLM, and no rate limits. Input is capped at 20,000 characters.

Fields extracted:

- emails — deduplicated
- urls — http/https, trailing punctuation stripped
- phones — international and US formats
- dates — ISO 8601, with optional time component
- numbers — with type (integer/decimal), raw value, and sign
- ipAddresses — IPv4
- hashes — classified by length (md5/sha1/sha256)
- mentions — @handles
- hashtags — #tags
- stats — word count, char count, line count

## Pricing

| Tool | Price | Currency |
|------|-------|----------|
| health | Free | - |
| timestamp | $0.001 | USD |
| uuid | $0.001 | USD |
| text.structure | $0.008 | USD |

## Supported Payment Protocols

### x402

USDC on Base (eip155:8453), Polygon (eip155:137), or Arbitrum (eip155:42161).

### MPP

pathUSD on Tempo.

Both protocols are advertised on the same 402 response. The client picks one.

## How Payment Works

1. Send the request without payment.
2. Receive a 402 with both an x402 challenge (PAYMENT-REQUIRED header) and an MPP challenge (WWW-Authenticate header).
3. Sign a payment authorization using either protocol.
4. Retry with the credential.
5. Receive the tool result once the payment is verified and settled.

Payment clients handle steps 2-4 automatically.

## Discovery

- /.well-known/x402 — discovery manifest
- /.well-known/mcp-pricing — tool pricing
- /.well-known/mcp/server-card.json — Smithery server card
- /openapi.json — OpenAPI 3.1 spec
- /llms.txt — plain-text summary for agent frameworks

## Connecting an MCP Client

Claude Desktop / Cursor config:

    {
      "mcpServers": {
        "chronokey": {
          "url": "https://cloudflare-mcp-worker.dylanrenovos.workers.dev/mcp"
        }
      }
    }

## Support

For questions, issues, or custom integrations, contact the maintainer.

## Terms

ChronoKey is a proprietary hosted service. All rights reserved.
