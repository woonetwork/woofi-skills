---
description: Get token exchange rates and expected receive amounts without executing a trade.
triggers:
  - "quote"
  - "price"
  - "exchange rate"
  - "token price"
  - "how much will I get"
---

# WOOFi Swap Quote Skill

## Overview

This skill queries the WOOFi `v1/quote` API to get aggregated token exchange rates and the expected received token amount for a specific chain. It does not generate or execute any transaction data.

**Base URL**: `https://sapi.woofi.com`
**Endpoint**: `POST /v1/quote`
**Content-Type**: `application/json`

---

## Parameters (QuoteRequest)

All parameters are passed in the JSON body.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `chain_id` | `integer` | Yes | - | Network chain ID (e.g., Base: `8453`, Arbitrum: `42161`, BSC: `56`, Polygon: `137`) |
| `sell_token`| `string` | Yes | - | Address or symbol of token to sell (Native ETH/BNB/etc is `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`) |
| `buy_token` | `string` | Yes | - | Address or symbol of token to buy |
| `sell_amount`| `string` | Yes | - | Human-readable amount to sell (e.g., `"1.5"`) |
| `slippage_pct`| `float` | No | `0.5` | Max allowed slippage percentage |
| `woofi_only`| `boolean` | No | `false` | If `true`, only sources quotes directly from WOOFi |

---

## Response Fields (QuoteResponse)

| Field | Type | Description |
|-------|------|-------------|
| `chain_id` | `integer` | Chain ID |
| `sell_token` | `string` | Sell token address |
| `buy_token` | `string` | Buy token address |
| `sell_amount` | `string` | Sell amount requested |
| `buy_amount` | `string` | Expected buy amount |
| `price` | `string` | Current execution price |
| `guaranteed_price` | `string` | Minimum guaranteed price after considering slippage |

---

## Example Usage

### Getting a Quote for swapping USDC to WBTC on Arbitrum
```bash
curl -X POST "https://sapi.woofi.com/v1/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "chain_id": 42161,
    "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
    "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
    "sell_amount": "1000"
  }'
```

### Response Example
```json
{
  "chain_id": 42161,
  "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
  "sell_amount": "1000",
  "buy_amount": "0.01524300",
  "price": "0.000015243",
  "guaranteed_price": "0.000015167"
}
```

---

## Common Use Cases

1. **Price Checking**: Determine the current value of a token swap.
2. **Slippage Evaluation**: Check the `guaranteed_price` against the standard `price` based on the provided `slippage_pct`.
3. **Pre-trade verification**: Always perform a quote before initiating a `swap` execution to verify expectations.