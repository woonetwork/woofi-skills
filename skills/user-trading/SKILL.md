---
description: Query WOOFi user trading volume history for swap and perpetual trades
triggers:
  - "user trading volume"
  - "my trading volume"
  - "user volume history"
  - "my volume"
  - "user perp volume"
  - "my perp volume"
  - "user swap volume"
  - "trading history"
---

# WOOFi User Trading Volume Skill

## Overview

This skill queries the WOOFi user trading volume endpoints for both swap and perpetual trading activity.

**Base URL**: `https://api.woofi.com`
**Authentication**: None required
**Rate Limit**: 5 requests/second
**Caching**: 5 seconds

---

## Endpoints

### 1. GET /user_trading_volumes

Returns the user's swap trading volume history across **all** supported networks for the specified period.

**Parameters**:

| Parameter | Required | Values |
|-----------|----------|--------|
| `period` | Yes | `7d`, `14d`, `30d` |
| `user_address` | Yes | User's EVM address |

**Logic**: Calculates UTC day boundaries based on the period, then aggregates `get_network_trading_volumes()` across all supported networks.

---

### 2. GET /user_perp_volumes

Returns the user's perpetual trading volume on WOOFi DEX for the given period.

**Parameters**:

| Parameter | Required | Values |
|-----------|----------|--------|
| `period` | Yes | `7d`, `14d`, `30d` |
| `user_address` | Yes | User's EVM address |

**Logic**: Reads from Redis and looks up the user's volume by address.

---

## Example Queries

### User swap volume (last 30 days)
```
GET https://api.woofi.com/user_trading_volumes?period=30d&user_address=0x1234...
```

### User perp volume (last 7 days)
```
GET https://api.woofi.com/user_perp_volumes?period=7d&user_address=0x1234...
```

---

## Notes

- **Address format**: All EVM addresses should be checksummed. The backend converts via `Web3.toChecksumAddress()`.
- **Period values**: Only `7d`, `14d`, `30d` are supported (different from the `/stat` endpoint which uses `1d`, `1w`, `1m`, etc.).
- **Cross-chain**: `/user_trading_volumes` automatically aggregates across all supported networks — no need to query per-chain.

---

## Common Use Cases

1. **User activity report**: Combine swap and perp volumes for a complete trading summary
2. **Loyalty/tier tracking**: Use 30-day volume to determine user tier or eligibility
3. **Period comparison**: Query different periods to identify trading trends
4. **Swap vs perps breakdown**: Compare `/user_trading_volumes` and `/user_perp_volumes` for product usage insights
