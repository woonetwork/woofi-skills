---
description: Query WOOFi Supercharger vault summary with APR rankings across all networks
triggers:
  - "earn summary"
  - "supercharger"
  - "supercharger vault"
  - "earn apr"
  - "best earn vault"
  - "vault ranking"
  - "earn opportunities"
  - "vault apr"
---

# WOOFi Earn Summary Skill

## Overview

This skill queries the WOOFi API `/earn_summary` endpoint to retrieve a summary of active Supercharger vaults across all networks, sorted by APR.

**Base URL**: `https://api.woofi.com`
**Endpoint**: `GET /earn_summary`
**Authentication**: None required
**Rate Limit**: 5 requests/second
**Caching**: 5 seconds

---

## Parameters

None.

---

## Response Fields

Returns a list of vault objects sorted by APR (highest first). Each vault contains APR, network, and vault details.

The APR is a weighted average calculated from:
- Loan vault and reserve vault balances
- Interest rates
- Debt from lending strategies

### Excluded Data

- Networks in paused state are excluded: `fantom`, `zksync`, `polygon_zkevm`
- Deprecated vaults are excluded
- Vaults with 0% APR are excluded

---

## Example Query

### Get all active Supercharger vaults
```
GET https://api.woofi.com/earn_summary
```

---

## Response Example

```json
{
  "status": "ok",
  "data": [
    {
      "network": "arbitrum",
      "token": "USDC",
      "apr": 12.5,
      "vault_address": "0x..."
    },
    {
      "network": "base",
      "token": "ETH",
      "apr": 8.3,
      "vault_address": "0x..."
    }
  ]
}
```

---

## Common Use Cases

1. **Best yield discovery**: Results are pre-sorted by APR — the first vault is the highest-yielding opportunity
2. **Network comparison**: Compare vault APRs across different networks to find the best chain for earning
3. **Vault availability**: Check which networks currently have active Supercharger vaults
4. **Yield monitoring**: Track APR changes over time by polling this endpoint periodically

### Difference from `/yield` Endpoint

- `/earn_summary` returns a cross-chain overview of Supercharger vaults sorted by APR
- `/yield` (used by the `earn-tvl` skill) returns detailed per-vault TVL, APY breakdown, and strategy composition for a specific network
- Use `/earn_summary` for discovery and ranking; use `/yield` for deep-dive analysis
