---
description: Query WOOFi Earn vault TVL, APY, and yield farming metrics across supported networks
triggers:
  - "earn tvl"
  - "earn vault"
  - "yield farming"
  - "woofi earn"
  - "tvl"
  - "apy"
  - "apr"
  - "vault yield"
  - "how much tvl"
  - "earn statistics"
---

# WOOFi Earn TVL Skill

## Overview

This skill queries the WOOFi Swap API `/yield` endpoint to retrieve TVL and yield farming data for WOOFi Earn vaults.

**Base URL**: `https://api.woofi.com`
**Endpoint**: `GET /yield`
**Authentication**: None required
**Rate Limit**: 120 requests/minute

---

## Parameters

| Parameter | Required | Values |
|-----------|----------|--------|
| `network` | Yes | See supported networks below |

### Supported Networks (10 networks)

`bsc`, `avalanche`, `polygon`, `arbitrum`, `optimism`, `linea`, `base`, `mantle`, `sonic`, `berachain`

Note: Solana, HyperEVM, and Monad are **not** supported for the `/yield` endpoint.

---

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `token` | string | Vault token symbol (e.g., USDC, USDT, ETH) |
| `apy` | number | Total APY as a percentage |
| `tvl` | string | TVL in the token's smallest unit (e.g., wei for ETH, 6-decimal units for USDC) |
| `decimals` | integer | Token decimals for TVL conversion |
| `price` | number | Current USD price of the token |
| `loan_percentage` | number | Portion of APR from lending strategy |
| `reserve_percentage` | number | Portion of APR from reserve strategy |
| `lending_apr` | number | APR component from lending |
| `reserve_apr` | number | APR component from reserve |

---

## TVL Conversion

All `tvl` values are in the token's smallest unit. Convert to token amount with:
```
token_amount = tvl / 10^decimals
```

Then convert to USD:
```
usd_tvl = token_amount * price
```

**Example**: If `tvl = "5000000000"`, `decimals = 6`, `price = 1.0` (USDC):
```
token_amount = 5000000000 / 10^6 = 5000 USDC
usd_tvl = 5000 * 1.0 = $5,000
```

---

## Example Queries

### Earn vault data on Arbitrum
```
GET https://api.woofi.com/yield?network=arbitrum
```

### Multi-chain TVL summary
Query each supported network separately, convert `tvl` for each vault, and sum across all networks for a global TVL figure.

---

## Response Example

```json
{
  "data": [
    {
      "token": "USDC",
      "apy": 8.45,
      "tvl": "12500000000000",
      "decimals": 6,
      "price": 1.0,
      "loan_percentage": 60.0,
      "reserve_percentage": 40.0,
      "lending_apr": 5.07,
      "reserve_apr": 3.38
    }
  ]
}
```

Converted TVL: `12500000000000 / 10^6 = 12,500,000 USDC = $12,500,000`

---

## Common Use Cases

1. **Global TVL report**: Query all 10 supported networks, convert each vault TVL, sum for total USD TVL
2. **Best yield comparison**: Sort vaults by `apy` descending to identify highest-yielding opportunities
3. **Strategy breakdown**: Use `loan_percentage` and `reserve_percentage` to understand yield composition
4. **Token-specific TVL**: Filter by `token` across networks to find total USDC or ETH TVL across the protocol
5. **APR component analysis**: Compare `lending_apr` vs `reserve_apr` to track strategy performance
