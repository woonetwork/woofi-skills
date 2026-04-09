---
description: Get price quotes for cross-chain swaps via Stargate bridge.
triggers:
  - "cross chain quote"
  - "bridge quote"
  - "cross chain price"
  - "bridge price"
  - "cross chain swap quote"
  - "how much across chains"
---

# WOOFi Cross-Chain Quote Skill

## Overview

This skill queries the WOOFi `v2/cross_chain/quote` API to get a price quote for a cross-chain swap. Cross-chain swaps use the Stargate bridge (LayerZero) to move tokens between different blockchain networks.

**Base URL**: `https://sapi.woofi.com`
**Endpoint**: `POST /v2/cross_chain/quote`
**Content-Type**: `application/json`

---

## Parameters

All parameters are passed in the JSON body.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `src_chain` | `int \| str` | Yes | - | Source chain ID (`42161`) or name (`"arbitrum"`) |
| `dst_chain` | `int \| str` | Yes | - | Destination chain ID (`8453`) or name (`"base"`) |
| `src_token` | `str` | Yes | - | Source token address or symbol (resolved against `src_chain`) |
| `dst_token` | `str` | Yes | - | Destination token address or symbol (resolved against `dst_chain`) |
| `src_amount` | `str` | Yes | - | Amount to sell (human-readable, e.g., `"100"`) |
| `slippage_pct` | `float` | No | `1.0` | Slippage tolerance (%) |
| `extra_fee_pct` | `float` | No | `0.0` | Additional fee (%) |

### Cross-Chain Token Resolution

**Important**: `src_token` resolves against `src_chain`, and `dst_token` resolves against `dst_chain`. The same symbol `"USDC"` maps to different contract addresses on different chains.

---

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `src_chain` | `int` | Source chain ID |
| `dst_chain` | `int` | Destination chain ID |
| `src_token` | `str` | Resolved source token address |
| `dst_token` | `str` | Resolved destination token address |
| `src_bridge_token` | `str` | Bridge token address on source chain |
| `dst_bridge_token` | `str` | Bridge token address on destination chain |
| `src_amount` | `str` | Input amount |
| `net_src_amount` | `str` | Amount after fees |
| `bridge_amount_in` | `str` | Amount entering the bridge |
| `bridge_amount_out` | `str` | Amount exiting the bridge |
| `dst_amount` | `str` | Final output amount on destination chain |
| `price_f` | `str` | Effective cross-chain price |

---

## Example Usage

### Quote for cross-chain USDC transfer from Arbitrum to Base
```bash
curl -X POST "https://sapi.woofi.com/v2/cross_chain/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "src_chain": "arbitrum",
    "dst_chain": "base",
    "src_token": "USDC",
    "dst_token": "USDC",
    "src_amount": "100"
  }'
```

### Response Example
```json
{
  "src_chain": 42161,
  "dst_chain": 8453,
  "src_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "dst_token": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "src_bridge_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "dst_bridge_token": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "src_amount": "100",
  "net_src_amount": "100",
  "bridge_amount_in": "100",
  "bridge_amount_out": "99.95",
  "dst_amount": "99.95",
  "price_f": "0.9995"
}
```

---

## Common Use Cases

1. **Cross-chain price checking**: Compare costs of bridging tokens between chains.
2. **Bridge fee estimation**: Check `src_amount` vs `dst_amount` to understand total bridge costs.
3. **Pre-trade verification**: Verify cross-chain swap expectations before executing.

## Notes

- Default slippage for cross-chain is `1.0%` (higher than single-chain `0.5%`) due to bridge latency.
- The bridge uses Stargate (LayerZero) — bridge fees are deducted from the transfer amount.
- `price_f` represents the effective exchange rate including all bridge fees and slippage.
