# MCP Worker

A paid MCP (Model Context Protocol) server running on Cloudflare Workers, with x402 micropayments settled in USDC.

## What This Does

This Worker exposes MCP tools over Streamable HTTP. One tool is free (`health`), one requires payment (`lookup`). Payment is handled via the [x402 protocol](https://x402.org) — a machine-native payment standard that uses the HTTP `402 Payment Required` status code.

Any MCP-compatible client (Claude Desktop, Cursor, or a custom agent) can connect and call the tools. Paid tools follow the standard x402 handshake: send the request, receive a 402 with payment details, settle USDC on-chain, retry with proof of payment.

## Endpoint
