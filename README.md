<div align="center">

<img src="logo.png" alt="ChronoKey logo" width="160" />

# ChronoKey

Timestamps, UUIDs, gas prices, domain intelligence, agent memory, and text-to-structure extraction for AI agents. Paid via x402 or MPP.

[View full pricing](https://cloudflare-mcp-worker.dylanrenovos.workers.dev/.well-known/mcp-pricing) · [Live stats](https://cloudflare-mcp-worker.dylanrenovos.workers.dev/stats)

</div>

---

## What This Is

ChronoKey is a hosted MCP server that provides 15 utility tools AI agents need on nearly every run. Payment is handled via the x402 protocol (USDC on Base, Polygon, or Arbitrum) or MPP (pathUSD on Tempo). Any MCP-compatible client can connect and call the tools.

## Endpoint

    https://cloudflare-mcp-worker.dylanrenovos.workers.dev/mcp

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

Output:

    {
      "iso8601": "2026-09-27T09:30:00.000Z",
      "unixSeconds": 1790488200,
      "unixMilliseconds": 1790488200000,
      "timezone": "America/New_York",
      "humanReadable": "Sunday, September 27, 2026 at 5:30:00 AM EDT"
    }

Pricing: First 10 calls per day per session free, then Pro subscription, then $0.001 USD.

### uuid

Generates 1-100 UUIDs.

Input: `{ "count": 5, "version": "v7" }`

- count (optional): 1-100, defaults to 1
- version (optional): "v4" (random) or "v7" (time-ordered), defaults to "v4"

Output:

    { "version": "v7", "count": 5, "uuids": ["018f3c8a-...", "..."] }

Pricing: First 10 calls per day per session free, then Pro subscription, then $0.001 USD.

### text.structure

Extracts structured entities from unstructured text into a strict, stable JSON schema.

Input:

    {
      "text": "Contact alice@example.com on 2026-09-26 about the $42 invoice. See https://example.com for details. #finance @team",
      "mode": "standard",
      "include_referrals": false
    }

- mode (optional): "standard" (default) or "deep"
- include_referrals (optional): append complementary service suggestions

Standard output:

    {
      "emails": ["alice@example.com"],
      "urls": ["https://example.com"],
      "phones": [],
      "dates": ["2026-09-26"],
      "numbers": [{ "raw": "42", "value": 42, "type": "integer", "negative": false }],
      "ipAddresses": [],
      "hashes": [],
      "mentions": ["@team"],
      "hashtags": ["#finance"],
      "stats": { "wordCount": 15, "charCount": 118, "lineCount": 1 },
      "mode": "standard"
    }

Deep output adds: currencies, organizations, persons, jsonBlobs.

Pricing: Standard $0.008 USD (Pro-covered), Deep $0.020 USD.

### text.structure.deep

Convenience alias for text.structure with mode "deep". Costs $0.020 USD.

### extract.and.fetch

Extracts entities then fetches the first URL found and returns its content.

Input: `{ "text": "..." }`

Output: `{ "entities": {...}, "fetched": { "url": "...", "ok": true, "status": 200, "contentType": "...", "content": "...", "error": null } }`

Content is truncated to 5,000 characters. Fetch times out after 10 seconds.

Pricing: $0.012 USD.

### extract.and.summarize

Extracts entities then returns a grouped summary with per-type counts, total entity count, and the dominant entity type.

Input: `{ "text": "..." }`

Output: `{ "summary": { "counts": {...}, "totalEntities": 23, "dominant": "emails" }, "sample": {...} }`

Pricing: $0.010 USD.

### extract.verified

Verified extraction. Returns standard entities plus a verification_token for the two-step confirmation flow.

Input: `{ "text": "..." }`

Output: `{ "entities": {...}, "verification_token": "uuid", "next_step": "Call extract.confirm..." }`

Pricing: $0.005 USD base. Total $0.015 USD when accuracy is confirmed.

### extract.confirm

Confirms whether a previous extract.verified result was accurate.

Input: `{ "verification_token": "...", "accurate": true }`

Pricing: $0.010 USD bonus on accurate=true. Free on accurate=false — misses are never charged.

### gas.price

Real-time EIP-1559 gas prices across five chains.

Input: `{ "chains": ["eip155:8453", "eip155:137"] }` — optional, defaults to all five.

Supported chains:

| Chain | CAIP-2 |
|---|---|
| Base | eip155:8453 |
| Polygon | eip155:137 |
| Arbitrum One | eip155:42161 |
| Ethereum | eip155:1 |
| Optimism | eip155:10 |

Output:

    {
      "chains": [
        {
          "caip2": "eip155:8453",
          "name": "Base",
          "nativeSymbol": "ETH",
          "slow": "0.0123",
          "standard": "0.0198",
          "fast": "0.0451",
          "baseFee": "0.0081",
          "priorityFee": { "p25": "0.0042", "p50": "0.0117", "p75": "0.0370", "p90": "0.0812" },
          "usdCost": {
            "nativeTransfer": "0.0013",
            "erc20Transfer": "0.0039",
            "swap": "0.0108"
          },
          "blockNumber": "23456789"
        }
      ],
      "cheapest": "eip155:8453",
      "updatedAt": "2026-09-27T09:30:00.000Z"
    }

Pricing: Pro-covered or $0.003 USD.

### domain.intel

One call returns WHOIS, DNS, and SSL certificate data for any domain. Aggregates RDAP, Cloudflare DNS-over-HTTPS, and Certificate Transparency logs.

Input:

    {
      "domain": "example.com",
      "records": ["A", "AAAA", "MX", "TXT", "NS", "CNAME", "SOA"],
      "include_ssl": true,
      "include_whois": true
    }

- domain (required): domain name to query
- records (optional): DNS record types to fetch. Defaults to all seven.
- include_ssl (optional): fetch SSL certificate history. Defaults to true.
- include_whois (optional): fetch WHOIS via RDAP. Defaults to true.

Output:

    {
      "domain": "example.com",
      "queriedAt": "2026-09-27T09:30:00.000Z",
      "dns": {
        "A": ["93.184.216.34"],
        "AAAA": ["2606:2800:220:1:248:1893:25c8:1946"],
        "MX": [{ "priority": 0, "exchange": "." }],
        "TXT": ["v=spf1 -all"],
        "NS": ["a.iana-servers.net", "b.iana-servers.net"],
        "SOA": { "mname": "a.iana-servers.net", "rname": "hostmaster.iana.org", "serial": 2026, "refresh": 3600, "retry": 600, "expire": 604800, "minimum": 300 }
      },
      "whois": {
        "registrar": "RESERVED-Internet Assigned Numbers Authority",
        "created": "1995-08-14T04:00:00Z",
        "expires": "2026-08-13T04:00:00Z",
        "updated": "2025-08-14T07:01:38Z",
        "status": ["clientDeleteProhibited", "clientTransferProhibited"],
        "nameservers": ["a.iana-servers.net", "b.iana-servers.net"],
        "dnssec": true
      },
      "ssl": {
        "certificates": [
          {
            "issuer": "DigiCert Inc",
            "commonName": "www.example.org",
            "notBefore": "2025-01-15T00:00:00Z",
            "notAfter": "2026-02-15T23:59:59Z",
            "san": ["www.example.org", "example.com", "example.net"],
            "serialNumber": "..."
          }
        ],
        "latestExpiry": "2026-02-15T23:59:59Z",
        "issuerCounts": { "DigiCert Inc": 4, "Let's Encrypt": 1 }
      },
      "summary": {
        "hasDns": true,
        "hasWhois": true,
        "hasSsl": true,
        "daysUntilDomainExpiry": 320,
        "daysUntilCertExpiry": 141,
        "dnssecEnabled": true,
        "emailSecurity": { "spf": true, "dmarc": false, "dkim": false },
        "warnings": []
      }
    }

Pricing: Pro-covered or $0.005 USD.

Upstream sources:

- DNS via Cloudflare DNS-over-HTTPS (`cloudflare-dns.com/dns-query`), with Google DoH (`dns.google/resolve`) as fallback
- WHOIS via RDAP through the `rdap.org` IANA bootstrap service
- SSL via Certificate Transparency logs through `crt.sh`

Partial failures are reported in `summary.warnings` — for example `["whois_unavailable"]` or `["ssl_ct_log_too_large"]`. The response is still returned with whatever data was successfully gathered.

### memory.store

Stores a value under a namespaced key. Namespaces are isolated — two callers using different namespaces never see each other's data.

Input:

    {
      "namespace": "my-agent",
      "key": "user_preference",
      "value": "prefers JSON output",
      "tags": ["preference"],
      "ttlSeconds": 86400
    }

- tags (optional): array of strings for filtering
- ttlSeconds (optional): time-to-live in seconds, max 365 days

Pricing: Pro-covered or $0.005 USD.

### memory.recall

Retrieves a value by key from a namespace. Returns null if not found or expired.

Input: `{ "namespace": "my-agent", "key": "user_preference" }`

Output: `{ "found": true, "key": "user_preference", "value": "prefers JSON output" }`

Pricing: Pro-covered or $0.005 USD.

### memory.search

Full-text search across a namespace using SQLite FTS5 with Porter stemming and BM25 ranking. Returns the top matches with relevance scores.

Input:

    {
      "namespace": "my-agent",
      "query": "preferences json",
      "limit": 10
    }

- limit (optional): 1-50, defaults to 10

Output:

    {
      "query": "preferences json",
      "namespace": "my-agent",
      "count": 3,
      "results": [
        { "key": "user_preference", "value": "prefers JSON output", "tags": ["preference"], "score": -1.42 }
      ]
    }

Pricing: $0.010 USD. Not covered by Pro subscription.

### subscribe

Subscribe to ChronoKey Pro for $5.00 USD for 30 days. Grants unlimited access to the covered tools.

Covered tools:

- timestamp
- uuid
- text.structure (standard mode only)
- gas.price
- domain.intel
- memory.store
- memory.recall

Not covered (always pay-per-call):

- text.structure.deep
- extract.and.fetch
- extract.and.summarize
- extract.verified / extract.confirm
- memory.search

Important: Subscriptions are bound to the MCP session ID. Reuse the same session across calls to benefit from the subscription.

Pricing: $5.00 USD for 30 days. Extending adds on top of existing expiry.

### subscription.status

Check the status of the Pro subscription bound to the current MCP session.

Output:

    {
      "active": true,
      "plan": "pro",
      "expiresAt": "2026-10-27T09:30:00.000Z",
      "daysRemaining": 28,
      "coveredTools": ["timestamp", "uuid", "text.structure", "gas.price", "domain.intel", "memory.store", "memory.recall"]
    }

Pricing: Free.

## Pricing Summary

| Tool | Price (USD) | Pro-covered | Free tier |
|---|---|---|---|
| health | Free | - | - |
| timestamp | $0.001 | yes | 10/day |
| uuid | $0.001 | yes | 10/day |
| text.structure (standard) | $0.008 | yes | - |
| text.structure (deep) | $0.020 | no | - |
| text.structure.deep | $0.020 | no | - |
| extract.and.fetch | $0.012 | no | - |
| extract.and.summarize | $0.010 | no | - |
| extract.verified | $0.005 | no | - |
| extract.confirm (accurate) | $0.010 | no | - |
| extract.confirm (inaccurate) | Free | - | - |
| gas.price | $0.003 | yes | - |
| domain.intel | $0.005 | yes | - |
| memory.store | $0.005 | yes | - |
| memory.recall | $0.005 | yes | - |
| memory.search | $0.010 | no | - |
| subscribe | $5.00 / 30d | - | - |
| subscription.status | Free | - | - |

## Freemium

The first 10 calls per day per MCP session are free for timestamp and uuid. No payment required. The remaining count is returned in the tool result's `_meta.freeCallsRemaining` field and in the `X-Free-Calls-Remaining` response header.

## Pro Subscription

$5.00 USD for 30 days. Covers timestamp, uuid, text.structure (standard mode), gas.price, domain.intel, memory.store, and memory.recall. Reuse the same MCP session across calls to benefit from the subscription. Active subscriptions are signaled via the X-Subscription-Active and X-Subscription-Expires response headers.

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

## Domain Intelligence

domain.intel aggregates three free, keyless upstream sources:

- DNS via Cloudflare DNS-over-HTTPS, with Google DoH as a fallback
- WHOIS via RDAP through the IANA bootstrap service at rdap.org
- SSL certificate history via Certificate Transparency logs at crt.sh

The tool returns partial data with a `summary.warnings` array when one of the upstreams fails. Warnings you may encounter:

| Warning | Cause |
|---|---|
| whois_unavailable | RDAP bootstrap down or TLD not supported |
| ssl_ct_log_unavailable | crt.sh unreachable or returned non-JSON |
| ssl_ct_log_too_large | Domain has too many certificates (response cap hit) |
| ssl_ct_log_parse_error | crt.sh returned malformed JSON |
| dns_no_records | Domain has no A, AAAA, MX, TXT, NS, CNAME, or SOA records |

## Memory Namespaces

Memory tools use isolated namespaces via Durable Object instances. Use a stable identifier (e.g., your agent's wallet address) to keep data separate across runs.

- Max value size: 50,000 characters
- Max key length: 256 characters
- Max search results: 50
- Max TTL: 365 days

Search uses SQLite FTS5 with Porter stemming and BM25 ranking. This means "running" matches "run", and results are ordered by relevance.

## Discovery

- /.well-known/x402 — discovery manifest
- /.well-known/mcp-pricing — tool pricing
- /.well-known/mcp/server-card.json — Smithery server card
- /openapi.json — OpenAPI 3.1 spec
- /llms.txt — plain-text summary for agent frameworks
- /referrals — complementary service catalog
- /stats — public call and revenue stats
- /verified-stats — verified extraction accuracy stats
- /subscription-stats — active subscription count

## Connecting an MCP Client

### Claude Desktop / Cursor

Add to your MCP config:

    {
      "mcpServers": {
        "chronokey": {
          "url": "https://cloudflare-mcp-worker.dylanrenovos.workers.dev/mcp"
        }
      }
    }

### MCP Inspector (for testing)

    npx @modelcontextprotocol/inspector@latest

Enter the endpoint URL and click Connect. The Inspector handles the initialize handshake and session management automatically.

## Example: Full Workflow

An agent checking a domain's trust signals, then checking gas prices before a transaction:

    // 1. Domain intelligence
    { "jsonrpc": "2.0", "id": 1, "method": "tools/call", "params": { "name": "domain.intel", "arguments": { "domain": "example.com" } } }

    // 2. Check gas before transacting
    { "jsonrpc": "2.0", "id": 2, "method": "tools/call", "params": { "name": "gas.price", "arguments": { "chains": ["eip155:8453"] } } }

    // 3. Store the decision
    { "jsonrpc": "2.0", "id": 3, "method": "tools/call", "params": { "name": "memory.store", "arguments": { "namespace": "agent-0xAb59", "key": "last_domain_check", "value": "example.com: dnssec=true, spf=true, cert expires 2026-02-15" } } }

    // 4. Search for related context
    { "jsonrpc": "2.0", "id": 4, "method": "tools/call", "params": { "name": "memory.search", "arguments": { "namespace": "agent-0xAb59", "query": "domain dnssec" } } }

## Support

For questions, issues, or custom integrations, contact the maintainer.

## Terms

ChronoKey is a proprietary hosted service. All rights reserved.
