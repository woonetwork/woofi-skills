---
description: Generate blockchain-ready transaction data for swaps, including necessary token approvals.
triggers:
  - "swap"
  - "trade"
  - "execute swap"
  - "build transaction"
  - "swap tokens"
---

# WOOFi Swap Execution Skill

## Overview

This skill queries the WOOFi `v1/swap` API to both obtain a token swap quote and generate the executable blockchain transaction data (`tx_steps`). It will also indicate if a prior token `Approve` transaction is needed.

**Base URL**: `https://sapi.woofi.com`
**Endpoint**: `POST /v1/swap`
**Content-Type**: `application/json`

---

## Parameters (SwapRequest)

All parameters are passed in the JSON body. This endpoint inherits all parameters from the `Quote` endpoint, plus execution-specific ones.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `chain_id` | `integer` | Yes | Network chain ID (e.g., Base: `8453`, Arbitrum: `42161`) |
| `sell_token`| `string` | Yes | Token address/symbol to sell (Native is `0xEeeeeE...`) |
| `buy_token` | `string` | Yes | Token address/symbol to buy |
| `sell_amount`| `string` | Yes | Human-readable amount to sell (e.g., `"1.5"`) |
| `to` | `string` | Yes | The wallet address that will receive the purchased tokens |
| `rebate_to` | `string` | Yes | Rebate receiving address (typically same as the `to` address) |
| `slippage_pct`| `float` | No | Max allowed slippage percentage (default: `0.5`) |
| `signer_address`| `string` | No | The user's wallet address initiating the swap. Used to check if an `Approve` is required. |

---

## Response Fields (SwapResponse)

| Field | Type | Description |
|-------|------|-------------|
| `needs_approve` | `boolean` | If `true`, the user must approve the router contract before swapping |
| `tx_steps` | `array` | List of transaction steps to be signed and submitted on-chain |
| `buy_amount` | `string` | Expected buy amount |
| `price` | `string` | Current execution price |
| `guaranteed_price` | `string` | Minimum guaranteed price |

### TxCall Structure
Each item in `tx_steps` represents an on-chain call:
- `to`: The target contract address for this step
- `data`: Hex string representing the transaction data payload
- `value`: Amount of native token to send in Wei
- `desc`: Human-readable description (e.g., "Approve USDC for WOOFi Router")

---

## Example Usage

### Building a Swap for 1000 USDC to WBTC on Arbitrum
```bash
curl -X POST "https://sapi.woofi.com/v1/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "chain_id": 42161,
    "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
    "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
    "sell_amount": "1000",
    "to": "0xYourWalletAddress",
    "rebate_to": "0xYourWalletAddress",
    "signer_address": "0xYourWalletAddress"
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
  "guaranteed_price": "0.000015167",
  "needs_approve": true,
  "tx_steps": [
    {
      "to": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
      "data": "0x095ea7b3...",
      "value": "0",
      "desc": "Approve USDC for WOOFi Router"
    },
    {
      "to": "0x4c4AF8DBc524681930a27b2F1Af5bcC8062E6fB7",
      "data": "0x...",
      "value": "0",
      "desc": "Swap 1000 USDC for WBTC via WOOFi"
    }
  ]
}
```

---

## Important Notes

1. **Sequential Execution**: If `tx_steps` contains multiple items, they MUST be submitted to the blockchain in the exact order provided (e.g., Approve transaction must confirm before the Swap transaction is sent).
2. **Native Asset Routing**: When selling or buying native assets (like ETH on Arbitrum or BNB on BSC), always use `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`.