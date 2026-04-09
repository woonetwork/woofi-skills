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

This skill queries the WOOFi `v2/swap` API to both obtain a token swap quote and generate the executable blockchain transaction data (`tx_steps`). It will also indicate if a prior token `Approve` transaction is needed.

**Base URL**: `https://sapi.woofi.com`
**Endpoint**: `POST /v2/swap`
**Content-Type**: `application/json`

---

## Parameters (SwapRequest)

All parameters are passed in the JSON body. This endpoint inherits all parameters from the `Quote` endpoint, plus execution-specific ones.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `chain` | `int \| str` | Yes | - | Chain ID (`42161`) or chain name (`"arbitrum"`) |
| `sell_token`| `str` | Yes | - | Token address (`"0xaf88..."`) or symbol/name (`"USDC"`) |
| `buy_token` | `str` | Yes | - | Token address or symbol/name to buy |
| `sell_amount`| `str` | Yes | - | Human-readable amount to sell (e.g., `"1.5"`) |
| `to` | `str` | Yes | - | The wallet address that will receive the purchased tokens |
| `rebate_to` | `str` | Yes | - | Rebate receiving address (typically same as the `to` address) |
| `slippage_pct`| `float` | No | `0.5` | Max allowed slippage percentage |
| `signer_address`| `str \| null` | No | `null` | The user's wallet address initiating the swap. Used to check if an `Approve` is required. |
| `woofi_only`| `bool` | No | `false` | Restrict to WOOFi pools only |

---

## Response Fields (SwapResponse)

| Field | Type | Description |
|-------|------|-------------|
| `chain` | `int` | Chain ID |
| `sell_token` | `str` | Resolved sell token address |
| `buy_token` | `str` | Resolved buy token address |
| `sell_amount` | `str` | Input amount |
| `buy_amount` | `str` | Expected buy amount |
| `price` | `str` | Current execution price |
| `guaranteed_price` | `str` | Minimum guaranteed price |
| `needs_approve` | `bool` | If `true`, the user must approve the router contract before swapping |
| `tx_steps` | `TxCall[]` | Transaction list: `[approve?, swap]` |

### TxCall Structure
Each item in `tx_steps` represents an on-chain call:

| Field | Type | Description |
|-------|------|-------------|
| `to` | `str` | The target contract address for this step |
| `data` | `str` | Hex string representing the transaction data payload |
| `value` | `str` | Native token value (wei) |
| `chain` | `int` | Chain ID |
| `desc` | `str` | Human-readable description (e.g., `"approve"`, `"swap"`) |

---

## Example Usage

### Building a Swap for 1000 USDC to WBTC on Arbitrum
```bash
curl -X POST "https://sapi.woofi.com/v2/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "arbitrum",
    "sell_token": "USDC",
    "buy_token": "WBTC",
    "sell_amount": "1000",
    "to": "0xYourWalletAddress",
    "rebate_to": "0xYourWalletAddress",
    "signer_address": "0xYourWalletAddress"
  }'
```

### Response Example
```json
{
  "chain": 42161,
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
      "chain": 42161,
      "desc": "approve"
    },
    {
      "to": "0x4c4AF8DBc524681930a27b2F1Af5bcC8062E6fB7",
      "data": "0x...",
      "value": "0",
      "chain": 42161,
      "desc": "swap"
    }
  ]
}
```

---

## Important Notes

1. **Sequential Execution**: If `tx_steps` contains multiple items, they MUST be submitted to the blockchain in the exact order provided (e.g., Approve transaction must confirm before the Swap transaction is sent).
2. **Native Asset Routing**: When selling or buying native assets (like ETH on Arbitrum or BNB on BSC), use the symbol name (e.g., `"ETH"`, `"BNB"`) or address `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`.
3. **Flexible Input**: Both chain names and token symbols are accepted — no need to look up chain IDs or token addresses manually.
