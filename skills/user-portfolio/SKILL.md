---
description: Query WOOFi user portfolio data including token balances, Supercharger positions, staking info, and boosted APR status
triggers:
  - "user balance"
  - "user portfolio"
  - "my balance"
  - "my position"
  - "user supercharger"
  - "user staking"
  - "user staking info"
  - "boosted apr"
  - "my staking"
  - "my earn"
  - "wallet balance"
---

# WOOFi User Portfolio Skill

## Overview

This skill queries the WOOFi user-specific API endpoints to retrieve token balances, Supercharger vault positions, staking information, and boosted APR eligibility.

**Base URL**: `https://api.woofi.com`
**Authentication**: None required
**Rate Limit**: 5 requests/second
**Caching**: 5 seconds

---

## Endpoints

### 1. GET /user_balances

Returns user's native and ERC20 token balances for all WOOFi-supported tokens on a network.

**Parameters**:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `network` | Yes | Network name (EVM only) |
| `user_address` | Yes | User's EVM address |

**Response**: List of token balance objects with address, symbol, decimals, and balance.

**Implementation detail**: Uses `Multicall3` to batch all balance queries in a single RPC call.

---

### 2. GET /user_supercharger_infos

Returns user's positions in WOOFi Supercharger vaults (deposits, pending withdrawals, etc.).

**Parameters**:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `user_address` | Yes | User's EVM address |

---

### 3. GET /user_stakingv2_infos

Returns user's Staking V2 information including claimed rewards and expected earnings.

**Parameters**:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `user_address` | Yes | User's EVM address |
| `network` | No | Default: `arbitrum` |
| `days` | No | Days for earnings calculation. Default: `30` |

**Response includes**:
- Claimed rewards (USDC/ARB)
- Compounded WOO
- Expected yearly earnings

---

### 4. GET /boosted_apr_info

Checks if a user is eligible for boosted APR (new stakers within 60 days of first stake).

**Parameters**:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `user_address` | Yes | User's EVM address |
| `network` | No | Network for staking lookup |

**Response Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `is_boosted` | boolean | Whether user is eligible for boosted APR |
| `deadline` | integer | Unix timestamp (first_stake_time + 60 days) |

---

## Supported Networks

**EVM only**: `bsc`, `avax`, `fantom`, `polygon`, `arbitrum`, `optimism`, `zksync`, `polygon_zkevm`, `linea`, `base`, `mantle`, `sonic`, `berachain`, `hyperevm`, `monad`

**Note**: All EVM addresses should be checksummed. The backend converts inputs via `Web3.toChecksumAddress()`.

---

## Example Queries

### User token balances on Arbitrum
```
GET https://api.woofi.com/user_balances?network=arbitrum&user_address=0x1234...
```

### User Supercharger positions
```
GET https://api.woofi.com/user_supercharger_infos?user_address=0x1234...
```

### User staking info (30-day earnings)
```
GET https://api.woofi.com/user_stakingv2_infos?user_address=0x1234...&network=arbitrum&days=30
```

### Check boosted APR eligibility
```
GET https://api.woofi.com/boosted_apr_info?user_address=0x1234...
```

---

## Common Use Cases

1. **Portfolio overview**: Combine `/user_balances`, `/user_supercharger_infos`, and `/user_stakingv2_infos` for a complete user portfolio
2. **Earnings report**: Use `/user_stakingv2_infos` with different `days` values to project earnings
3. **Boost status check**: Use `/boosted_apr_info` to inform users about their boosted APR window
4. **Multi-chain balances**: Query `/user_balances` across multiple networks for a full balance sheet
