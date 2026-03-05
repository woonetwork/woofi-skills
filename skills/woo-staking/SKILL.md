---
description: Query global WOO token staking statistics including APR, Multiplier Point boost, and total staked amount
triggers:
  - "woo staking"
  - "staking apr"
  - "staking stats"
  - "staked woo"
  - "multiplier points"
  - "woo stake"
  - "staking yield"
  - "woo token staking"
---

# WOO Staking Statistics Skill

## Overview

This skill queries the WOOFi Swap API `/stakingv2` endpoint to retrieve global WOO token staking statistics.

**Base URL**: `https://api.woofi.com`
**Endpoint**: `GET /stakingv2`
**Authentication**: None required
**Rate Limit**: 120 requests/minute

---

## Parameters

This endpoint is **chain-agnostic** — no `network` parameter is required. It returns global aggregate staking data across all chains.

---

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `avg_apr` | number | Combined average APR (base + Multiplier Point boost) as a percentage |
| `base_apr` | number | Base staking APR without any boost, as a percentage |
| `mp_boosted_apr` | number | Additional APR from Multiplier Point (MP) boost, as a percentage |
| `total_woo_staked` | string | Total WOO tokens staked globally, in wei |
| `decimals` | integer | WOO token decimals (typically 18) |

### APR Formula

```
avg_apr = base_apr + mp_boosted_apr
```

The Multiplier Point system rewards long-term stakers with an additional APR boost on top of the base rate.

---

## WOO Amount Conversion

`total_woo_staked` is returned in wei. Convert to WOO tokens with:
```
woo_amount = total_woo_staked / 10^decimals
```

**Example**: If `total_woo_staked = "850000000000000000000000000"` and `decimals = 18`:
```
woo_amount = 850000000000000000000000000 / 10^18 = 850,000,000 WOO
```

---

## Example Query

### Global staking statistics
```
GET https://api.woofi.com/stakingv2
```

---

## Response Example

```json
{
  "data": {
    "avg_apr": 18.7,
    "base_apr": 12.4,
    "mp_boosted_apr": 6.3,
    "total_woo_staked": "850000000000000000000000000",
    "decimals": 18
  }
}
```

Converted: `850,000,000 WOO` total staked; `18.7%` average APR (`12.4%` base + `6.3%` MP boost)

---

## Common Use Cases

1. **Current staking rate**: Read `avg_apr` for the all-in yield for WOO stakers
2. **Boost analysis**: Compare `base_apr` vs `mp_boosted_apr` to understand how much long-term staking rewards contribute
3. **Staking participation**: Convert `total_woo_staked` to get the aggregate WOO locked in the protocol
4. **Dashboard widget**: Combine `avg_apr` and converted `total_woo_staked` for a staking summary card
