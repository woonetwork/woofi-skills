---
description: Build transactions for cross-chain swaps via Stargate bridge with token approvals.
triggers:
  - "cross chain swap"
  - "bridge swap"
  - "cross chain trade"
  - "bridge tokens"
  - "swap across chains"
  - "transfer cross chain"
---

# WOOFi Cross-Chain Swap Skill

## Overview

This skill queries the WOOFi `v2/cross_chain/swap` API to build transaction(s) for a cross-chain swap. It uses the Stargate bridge (LayerZero) and returns executable transaction data including approval steps if needed.

**Base URL**: `https://sapi.woofi.com`
**Endpoint**: `POST /v2/cross_chain/swap`
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
| `src_amount` | `str` | Yes | - | Amount to sell (human-readable) |
| `to` | `str` | Yes | - | Recipient address on destination chain |
| `slippage_pct` | `float` | No | `1.0` | Slippage tolerance (%) |
| `extra_fee_pct` | `float` | No | `0.0` | Additional fee (%) |
| `signer_address` | `str \| null` | No | `null` | Signer address (for approve check) |
| `airdrop_native_amount` | `int` | No | `0` | Native token airdrop amount on destination chain (uint128) |

---

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `src_chain` | `int` | Source chain ID |
| `dst_chain` | `int` | Destination chain ID |
| `src_token` | `str` | Resolved source token address |
| `dst_token` | `str` | Resolved destination token address |
| `src_bridge_token` | `str` | Bridge token on source chain |
| `dst_bridge_token` | `str` | Bridge token on destination chain |
| `src_amount` | `str` | Input amount |
| `net_src_amount` | `str` | Amount after fees |
| `bridge_amount_in` | `str` | Amount entering the bridge |
| `bridge_amount_out` | `str` | Amount exiting the bridge |
| `dst_amount` | `str` | Final output amount |
| `price_f` | `str` | Effective cross-chain price |
| `native_fee` | `str` | LayerZero native fee (human-readable, in source chain native token) |
| `needs_approve` | `bool` | Whether token approval is needed |
| `tx_steps` | `TxCall[]` | Transaction list: `[approve?, crossSwap]` |

### TxCall Structure

| Field | Type | Description |
|-------|------|-------------|
| `to` | `str` | Contract address |
| `data` | `str` | Calldata (hex) |
| `value` | `str` | Native token value (wei) |
| `chain` | `int` | Chain ID |
| `desc` | `str` | Description (e.g., `"approve"`, `"crossSwap"`) |

---

## Example Usage

### Cross-chain swap USDC from Arbitrum to Base
```bash
curl -X POST "https://sapi.woofi.com/v2/cross_chain/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "src_chain": "arbitrum",
    "dst_chain": "base",
    "src_token": "USDC",
    "dst_token": "USDC",
    "src_amount": "100",
    "to": "0xYourWalletAddress",
    "signer_address": "0xYourWalletAddress"
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
  "price_f": "0.9995",
  "native_fee": "0.00025",
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
      "to": "0x...",
      "data": "0x...",
      "value": "250000000000000",
      "chain": 42161,
      "desc": "crossSwap"
    }
  ]
}
```

---

## Important Notes

1. **Sequential Execution**: `tx_steps` must be submitted in order — approve first, then crossSwap.
2. **Native Fee**: The cross-chain swap transaction requires sending `native_fee` as the `value` field in the crossSwap tx step. This pays for the LayerZero message.
3. **Destination Delivery**: Tokens arrive on the destination chain after the bridge confirms (typically a few minutes).
4. **Native Airdrop**: Use `airdrop_native_amount` to receive a small amount of native gas token on the destination chain (useful for new wallets).
5. **Token Resolution**: `src_token` resolves against `src_chain`, `dst_token` resolves against `dst_chain`. The same symbol maps to different addresses on different chains.
