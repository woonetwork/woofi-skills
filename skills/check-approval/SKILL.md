---
description: Check if a token approval is needed before swapping on WOOFi.
triggers:
  - "check approval"
  - "token approval"
  - "allowance"
  - "approve token"
  - "needs approval"
---

# WOOFi Check Approval Skill

## Overview

This skill queries the WOOFi `v2/check_approval` API to check if a token approval is needed before executing a swap. It returns the current allowance and whether the user needs to approve the router contract.

**Base URL**: `https://sapi.woofi.com`
**Endpoint**: `POST /v2/check_approval`
**Content-Type**: `application/json`

---

## Parameters

All parameters are passed in the JSON body.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `chain` | `int \| str` | Yes | - | Chain ID (`42161`) or chain name (`"arbitrum"`) |
| `sell_token` | `str` | Yes | - | Token address (`"0xaf88..."`) or symbol/name (`"USDC"`) |
| `owner` | `str` | Yes | - | User wallet address |
| `sell_amount` | `str` | Yes | - | Amount to check (human-readable, e.g., `"1000"`) |

---

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `chain` | `int` | Chain ID |
| `sell_token` | `str` | Resolved token address |
| `owner` | `str` | User wallet address |
| `spender` | `str` | Router address (approval target) |
| `current_allowance` | `str` | Current allowance (human-readable) |
| `needs_approval` | `bool` | Whether approval is needed |

---

## Example Usage

### Check if USDC approval is needed on Arbitrum
```bash
curl -X POST "https://sapi.woofi.com/v2/check_approval" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "arbitrum",
    "sell_token": "USDC",
    "owner": "0xYourWalletAddress",
    "sell_amount": "1000"
  }'
```

### Response Example
```json
{
  "chain": 42161,
  "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "owner": "0xYourWalletAddress",
  "spender": "0x4c4AF8DBc524681930a27b2F1Af5bcC8062E6fB7",
  "current_allowance": "0",
  "needs_approval": true
}
```

---

## Common Use Cases

1. **Pre-swap check**: Verify if approval is needed before calling `/v2/swap`.
2. **Allowance monitoring**: Check how much allowance remains for the WOOFi router.
3. **UX optimization**: Show users whether they need one or two transactions before swapping.

## Notes

- If using `/v2/swap` with `signer_address`, the swap endpoint already checks approval and includes the approve step in `tx_steps`. This endpoint is useful for standalone approval checks.
- Native tokens (ETH, BNB, etc.) never need approval.
