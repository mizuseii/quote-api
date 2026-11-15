# Quote API — a self-selling micro-agent on Base

A minimal autonomous paid API: one random inspirational quote for **$0.001 USDC per call**, priced and settled by the agent itself via **x402 v2** on Base. No accounts, no API keys, no human in the loop.

Submitted to the **Wildcard track, zkVerify Agent Games Season 01**.

- **Live endpoint:** https://steep-glitter-93db.vilaslan9.workers.dev
- **vAPI Network listing:** https://api.vapinetwork.ai/call/quote-api-287
- **Price:** $0.001/call (1,000 atomic USDC, exact scheme, Base `eip155:8453`)
- **Stack:** one Cloudflare Worker, zero dependencies, no database.

## Try it

```bash
# Free discovery
curl https://steep-glitter-93db.vilaslan9.workers.dev/
curl https://steep-glitter-93db.vilaslan9.workers.dev/openapi.json

# Payment challenge (402 — nothing is spent)
curl -i https://steep-glitter-93db.vilaslan9.workers.dev/quote

# Paid call — any x402 v2 client, $0.001 USDC on Base:
# GET /quote with an X-PAYMENT header  ->  200 {"quote": "..."}
```

## How it works

1. Caller hits `GET /quote` with no payment → the Worker answers `402` with an
   x402 v2 payment challenge (also in the `PAYMENT-REQUIRED` header and in
   `/openapi.json` as `x-payment-info`).
2. Caller pays 1,000 atomic USDC on Base and retries with the signed payment
   in the `X-PAYMENT` (or `PAYMENT-SIGNATURE`) header.
3. The Worker forwards the payload to the x402 facilitator
   (`facilitator.payai.network`) for `/verify`, then `/settle`.
4. Only on successful settlement does it return a random quote, with the
   settlement payload in the `PAYMENT-RESPONSE` / `X-PAYMENT-RESPONSE` headers.

Any invalid payment, failed verification, or failed settlement returns `402`
with the reason — the quote is never served unpaid.

## Deploy your own

```bash
# quote-worker.js is the whole agent. Deploy as a Cloudflare Worker:
npx wrangler deploy quote-worker.js --name quote-api
```

Change `PAY_TO` (the wallet that receives the $0.001), `AMOUNT`, and `QUOTES`
at the top of the file. Everything else stays the same.

## Honest scope

This agent does one thing: a random quote from a small built-in list, for
$0.001. The point being demonstrated is the **payment loop** — an agent that
prices itself, verifies payment on-chain, settles, and delivers, with no human
approving a tenth of a cent — not the depth of the catalogue.

## Links

- Live endpoint: https://steep-glitter-93db.vilaslan9.workers.dev
- OpenAPI spec: https://steep-glitter-93db.vilaslan9.workers.dev/openapi.json
- x402 discovery: https://steep-glitter-93db.vilaslan9.workers.dev/.well-known/x402
- vAPI Network listing: https://api.vapinetwork.ai/call/quote-api-287
- Agent Games Wildcard track: https://agentgames.zkverify.io/events/wildcard
