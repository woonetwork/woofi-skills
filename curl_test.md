# WOOFi v1 API Curl Test Samples

This file contains readily executable `curl` commands for testing the WOOFi `v1/quote` and `v1/swap` APIs, as documented in our skills and API references.

## 1. Get Quote (Base Chain)
**Scenario**: Swapping 0.1 Native ETH for USDC on Base.

```bash
curl -s -X POST "https://sapi.woofi.com/v1/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "chain_id": 8453,
    "sell_token": "0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE",
    "buy_token": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    "sell_amount": "0.1"
  }' | jq .
```

---

## 2. Get Quote (Arbitrum Chain)
**Scenario**: Swapping 1000 USDC for WBTC on Arbitrum.

```bash
curl -s -X POST "https://sapi.woofi.com/v1/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "chain_id": 42161,
    "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
    "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
    "sell_amount": "1000"
  }' | jq .
```

---

## 3. Build Swap Transaction (Arbitrum Chain)
**Scenario**: Generating the on-chain transaction payload to swap 1000 USDC for WBTC on Arbitrum, including approval checks.

```bash
curl -s -X POST "https://sapi.woofi.com/v1/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "chain_id": 42161,
    "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
    "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
    "sell_amount": "1000",
    "to": "0x000000000000000000000000000000000000dEaD",
    "rebate_to": "0x000000000000000000000000000000000000dEaD",
    "signer_address": "0x000000000000000000000000000000000000dEaD"
  }' | jq .
```

---

## 4. Get Quote with Custom Slippage (Arbitrum Chain)
**Scenario**: Swapping 1000 USDC for WBTC on Arbitrum, strictly enforcing a maximum slippage of 0.1% (default is 0.5%).

```bash
curl -s -X POST "https://sapi.woofi.com/v1/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "chain_id": 42161,
    "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
    "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
    "sell_amount": "1000",
    "slippage_pct": 0.1
  }' | jq .
```

---

## 5. Build Swap Transaction (Base Chain - Native ETH to USDC)
**Scenario**: Generating swap data to trade 0.1 Native ETH for USDC on Base, with a specific receiver address. Since it's native ETH, `needs_approve` will be false.

```bash
curl -s -X POST "https://sapi.woofi.com/v1/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "chain_id": 8453,
    "sell_token": "0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE",
    "buy_token": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    "sell_amount": "0.1",
    "to": "0x1234567890123456789012345678901234567890",
    "rebate_to": "0x1234567890123456789012345678901234567890",
    "signer_address": "0x1234567890123456789012345678901234567890"
  }' | jq .
```

---

## 6. Build Swap Transaction (BSC - USDT to Native BNB)
**Scenario**: Swapping 500 USDT to Native BNB on Binance Smart Chain (BSC - Chain ID 56). This typically requires approving USDT first if the signer hasn't done so.

```bash
curl -s -X POST "https://sapi.woofi.com/v1/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "chain_id": 56,
    "sell_token": "0x55d398326f99059fF775485246999027B3197955",
    "buy_token": "0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE",
    "sell_amount": "500",
    "to": "0x1234567890123456789012345678901234567890",
    "rebate_to": "0x1234567890123456789012345678901234567890",
    "signer_address": "0x1234567890123456789012345678901234567890"
  }' | jq .
```

---

## 7. Build Swap Transaction with High Slippage Tolerance (Polygon)
**Scenario**: Swapping 100 USDC to Native MATIC on Polygon (Chain ID 137) with a high slippage tolerance of 2.5% during volatile market conditions.

```bash
curl -s -X POST "https://sapi.woofi.com/v1/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "chain_id": 137,
    "sell_token": "0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359",
    "buy_token": "0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE",
    "sell_amount": "100",
    "slippage_pct": 2.5,
    "to": "0x1234567890123456789012345678901234567890",
    "rebate_to": "0x1234567890123456789012345678901234567890",
    "signer_address": "0x1234567890123456789012345678901234567890"
  }' | jq .
```