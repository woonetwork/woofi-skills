---
description: Query WOOFi supported networks, DEXs, and tokens for swaps
triggers:
  - "swap support"
  - "supported networks"
  - "supported tokens"
  - "supported dex"
  - "which chains"
  - "which tokens"
  - "swap config"
  - "woofi networks"
---

# WOOFi Swap Support Skill

## Overview

This skill queries the WOOFi API `/swap_support` endpoint to retrieve supported networks, DEXs, and token information for swaps.

**Base URL**: `https://api.woofi.com`
**Endpoint**: `GET /swap_support`
**Authentication**: None required
**Rate Limit**: 5 requests/second
**Caching**: 3600 seconds (1 hour)

---

## Parameters

None.

---

## Response Fields

The response `data` is a dictionary keyed by network name (e.g., `bsc`, `avax`). Each network contains:

| Field | Type | Description |
|-------|------|-------------|
| `dexs` | array | List of supported DEXs on the network |
| `network_infos` | object | Network metadata: name, chain ID, RPC URL, explorer URL, etc. |
| `token_infos` | array | Tokens supported for swaps, each with address, symbol, decimals, and `swap_enable` status |

### Supported Networks

**EVM**: `bsc`, `avax`, `fantom`, `polygon`, `arbitrum`, `optimism`, `zksync`, `polygon_zkevm`, `linea`, `base`, `mantle`, `sonic`, `berachain`, `hyperevm`, `monad`

**SVM**: `solana`

**Other**: `sei` (hardcoded, chain ID `1329`, tokens: SEI, USDC)

---

## Example Query

### Get all supported swap configurations
```
GET https://api.woofi.com/swap_support
```

---

## Response Example

```json
{
  "status": "ok",
  "data": {
    "arbitrum": {
      "dexs": ["WOOFi", "1inch"],
      "network_infos": {
        "name": "Arbitrum",
        "chain_id": 42161,
        "explorer_url": "https://arbiscan.io"
      },
      "token_infos": [
        {
          "address": "0x...",
          "symbol": "ETH",
          "decimals": 18,
          "swap_enable": true
        }
      ]
    }
  }
}
```

---

## Common Use Cases

1. **Network discovery**: List all networks where WOOFi swaps are available
2. **Token lookup**: Find which tokens are supported for swapping on a specific chain
3. **DEX availability**: Check which DEXs are integrated on each network
4. **Chain metadata**: Retrieve chain IDs, RPC URLs, and explorer links for frontend integration
5. **Swap eligibility**: Check `swap_enable` to determine if a token is currently active for swaps
