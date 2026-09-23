---
name: farmdash-signal-architect
description: Prepare user-authorized EVM swap transactions through FarmDash.
version: 1.3.0
license: MIT
---

# FarmDash Signal Architect

A third-party skill for requesting EVM swap estimates and preparing a transaction for the user's own wallet. It is not a Binance service and does not place Binance exchange orders.

## Safety and scope

- Never ask for, accept, store, or transmit a seed phrase or private key. Signing must remain in the user's wallet.
- Never recommend a token or initiate a swap on the skill's own initiative.
- A request for information or an estimate is not permission to prepare or sign a transaction.
- Before any wallet action, show the exact asset contracts, networks, input amount, recipient, minimum output, slippage, expiry, approvals, and all quoted route, network, and service costs. Do not omit or minimize a cost returned by the quote.
- Require a fresh, explicit confirmation immediately before the wallet signs. The user decides whether to submit the signed transaction.
- Stop if the route, wallet, recipient, quote, simulation, or network does not match the user's request, or if required information is unavailable.

## Workflow

1. Ask for any missing details: source and destination network, token contract addresses, amount, recipient, and slippage tolerance. Do not infer an address from a token symbol alone.
2. For an exploratory estimate, use `GET /agents/quote`. Label its default `market_estimate` as indicative—not an executable provider quote, a liquidity guarantee, or an instruction to trade.
3. Only when the user explicitly asks to prepare a swap, create an execution intent with `POST /v1/agent/quote-intent`, using the exact parameters and a unique idempotency key. Follow the current API contract for required fields.
4. Retrieve the firm quote for that intent and compare every field with the user's request. Clearly present the expected output, slippage, quote freshness, route, approvals, and all costs returned by the API.
5. Run the required wallet-bound preflight using `POST /v1/simulate`. Do not continue unless the simulation succeeds, is fresh, and matches the same intent and wallet.
6. Show the complete transaction summary and ask for explicit confirmation. Only then use the wallet's local signing flow and the documented `POST /agents/swap` preparation endpoint.
7. FarmDash returns a prepared transaction; it does not sign or broadcast it. The user's wallet signs and submits it to the target network. Report the resulting status without claiming success until the chain confirms it.

## Stop conditions

Stop and explain the issue if an endpoint is unavailable, the quote or simulation is stale or mismatched, the route is unsupported, the API returns a halt/error, or the user has not confirmed the exact transaction. Never retry a failed or changed intent as if it were the same request.

## References

- Agent Hub and integration guidance: https://www.farmdash.one/agents
- OpenAPI contract: https://www.farmdash.one/agents/openapi.yaml
- Public MCP manifest: https://www.farmdash.one/.well-known/mcp.json

Authentication and request details can change; follow the current published contract. Do not infer access requirements from old examples.