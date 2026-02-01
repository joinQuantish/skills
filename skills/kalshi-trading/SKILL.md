---
name: kalshi-trading
version: 1.0.0
description: Trade prediction markets on Kalshi via Quantish MCP (Solana/DFlow)
author: Quantish
tags: [trading, kalshi, prediction-markets, solana]
---

# Kalshi Trading Skill

Complete guide to trading on Kalshi through the Quantish MCP server using Solana and DFlow.

## Prerequisites

Run `/quantish-mcp-setup` first to connect to the MCP servers.

## Key Differences from Polymarket

| Aspect | Polymarket | Kalshi |
|--------|------------|--------|
| Chain | Polygon | Solana |
| Gas Token | MATIC | SOL |
| Token Type | ERC-1155 | SPL Token-2022 |
| Order Routing | CLOB direct | DFlow DEX |
| Regulation | Crypto-native | CFTC-regulated |

## Trading Workflow

### 1. Check Wallet Status

```tool
mcp__quantish-kalshi__kalshi_get_wallet_status
```

**Response fields:**
- `publicKey`: Your Solana wallet address
- `walletType`: "generated" or "imported"
- `status`: Must be "READY"
- `balances`: SOL and USDC amounts

### 2. Check Balances

```tool
mcp__quantish-kalshi__kalshi_get_balances
```

**Response:**
```json
{
  "publicKey": "3yeJBPgGDvU82vReuNhtJekzarnGg5TeSK8WNPvebGnF",
  "balances": {
    "sol": 0.07,   // For gas fees (~0.004 per trade)
    "usdc": 8.0    // Available to trade
  }
}
```

**Important:** Need SOL for transaction fees (minimum ~0.01 SOL recommended).

### 3. Find Markets

```tool
mcp__quantish-kalshi__kalshi_search_markets
  query: "Fed Chair"
  limit: 5
  marketStatus: "active"  // "active", "inactive", "finalized", "all"
```

**Key fields:**
- `ticker`: Market identifier (e.g., "KXFEDCHAIRNOM-29-KW")
- `yesMint`: YES outcome token address
- `noMint`: NO outcome token address
- `yesAsk`: Current ask price for YES
- `noBid`: Current bid price for NO

### 4. Get Market Details

```tool
mcp__quantish-kalshi__kalshi_get_market
  ticker: "KXFEDCHAIRNOM-29-KW"
```

**Response:**
```json
{
  "market": {
    "ticker": "KXFEDCHAIRNOM-29-KW",
    "title": "Will Trump next nominate Kevin Warsh as Fed Chair?",
    "status": "active",
    "yesBid": "0.9700",
    "yesAsk": "0.9800",
    "noBid": "0.0200",
    "noAsk": "0.0300",
    "accounts": {
      "CASHx9KJ...": {
        "yesMint": "3ojpNwqx2RVYpTCXLfKknvSXWWJFuVT2pdzotZ5PG1wF",
        "noMint": "GkC4HPRbTFdEwiU7XiZeaHyH779oBVrRWrbXbB6QDFXC",
        "isInitialized": true
      }
    }
  }
}
```

**Note:** Use the `yesMint` or `noMint` from the `CASHx9KJ...` account (CASH token).

### 5. Get Live Pricing

```tool
mcp__quantish-kalshi__kalshi_get_live_data
  marketTicker: "KXFEDCHAIRNOM-29-KW"
```

**Response:**
```json
{
  "liveData": {
    "ticker": "KXFEDCHAIRNOM-29-KW",
    "yesPrice": "0.9700",
    "noPrice": "0.0200",
    "yesBid": "0.9700",
    "yesAsk": "0.9800",
    "status": "active"
  }
}
```

### 6. Get Quote (Optional)

Preview a trade before executing:

```tool
mcp__quantish-kalshi__kalshi_get_quote
  inputMint: "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"  // USDC
  outputMint: "3ojpNwqx2RVYpTCXLfKknvSXWWJFuVT2pdzotZ5PG1wF"  // YES token
  amount: 1000000  // 1 USDC (6 decimals)
```

### 7. Buy YES Shares

```tool
mcp__quantish-kalshi__kalshi_buy_yes
  marketTicker: "KXFEDCHAIRNOM-29-KW"
  yesOutcomeMint: "3ojpNwqx2RVYpTCXLfKknvSXWWJFuVT2pdzotZ5PG1wF"
  usdcAmount: 2
  slippageBps: 100  // optional, default 1%
```

**Response:**
```json
{
  "message": "Buy YES order executed",
  "txSignature": "5TFFSF3UbaZWvz...",
  "status": "pending"
}
```

### 8. Buy NO Shares

```tool
mcp__quantish-kalshi__kalshi_buy_no
  marketTicker: "KXFEDCHAIRNOM-29-KW"
  noOutcomeMint: "GkC4HPRbTFdEwiU7XiZeaHyH779oBVrRWrbXbB6QDFXC"
  usdcAmount: 2
```

### 9. Check Token Holdings (Positions)

```tool
mcp__quantish-kalshi__kalshi_get_token_holdings
```

**Response:**
```json
{
  "tokens": [
    {
      "mint": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
      "balance": "8000000",  // 8 USDC
      "decimals": 6
    },
    {
      "mint": "3ojpNwqx2RVYpTCXLfKknvSXWWJFuVT2pdzotZ5PG1wF",
      "balance": "2000000",  // 2 YES shares
      "decimals": 6
    }
  ]
}
```

**Note:** `get_token_holdings` is more reliable than `get_positions` for Kalshi.

### 10. Check Orders

```tool
mcp__quantish-kalshi__kalshi_get_orders
  status: "SUBMITTED"  // optional
```

### 11. Sell Position

```tool
mcp__quantish-kalshi__kalshi_sell_position
  outcomeMint: "3ojpNwqx2RVYpTCXLfKknvSXWWJFuVT2pdzotZ5PG1wF"
  tokenAmount: 2
  slippageBps: 100
```

### 12. Redeem Winnings (After Market Settles)

Check if market is settled:

```tool
mcp__quantish-kalshi__kalshi_check_redemption_status
  ticker: "KXFEDCHAIRNOM-29-KW"
```

Get redeemable positions:

```tool
mcp__quantish-kalshi__kalshi_get_redeemable_positions
```

Redeem all:

```tool
mcp__quantish-kalshi__kalshi_redeem_all_positions
```

Or redeem specific:

```tool
mcp__quantish-kalshi__kalshi_redeem_winnings
  outcomeMint: "3ojpNwqx2RVYpTCXLfKknvSXWWJFuVT2pdzotZ5PG1wF"
  tokenAmount: 2
```

## Token Swaps

### Get Swap Quote

```tool
mcp__quantish-kalshi__kalshi_get_swap_quote
  inputMint: "SOL"
  outputMint: "USDC"
  amount: 0.1
```

### Swap SOL to USDC

```tool
mcp__quantish-kalshi__kalshi_swap_sol_to_usdc
  solAmount: 0.1
```

### Swap USDC to SOL

```tool
mcp__quantish-kalshi__kalshi_swap_usdc_to_sol
  usdcAmount: 5
```

## Transfers

### Send USDC

```tool
mcp__quantish-kalshi__kalshi_send_usdc
  toAddress: "..."
  amount: 5
```

### Send SOL

```tool
mcp__quantish-kalshi__kalshi_send_sol
  toAddress: "..."
  amount: 0.1
```

### Get Deposit Address

```tool
mcp__quantish-kalshi__kalshi_get_deposit_address
```

## Important Notes

### Market Ticker Format

Kalshi tickers follow this pattern:
- `KXFEDCHAIRNOM-29-KW` = Fed Chair Nomination, 2029 expiry, Kevin Warsh

### Token Decimals

All Kalshi tokens use 6 decimals:
- `1000000` = 1 token/USDC
- `2000000` = 2 tokens/USDC

### Common Token Addresses

| Token | Address |
|-------|---------|
| USDC | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |
| CASH | `CASHx9KJUStyftLFWGvEVf59SGeG9sh5FfcnZMVPCASH` |

### Gas Costs

- Typical trade: ~0.004 SOL
- Keep at least 0.01 SOL for gas

## Best Practices

1. **Check wallet has SOL** - Trades fail without gas
2. **Use get_market first** - Get correct yesMint/noMint
3. **Verify with get_token_holdings** - More reliable than get_positions
4. **Set slippage** - Default 1% (100 bps) usually fine
5. **Check market status** - Only "active" markets can be traded

## Example: Complete Trade Flow

```
1. kalshi_get_wallet_status → Confirm ready
2. kalshi_get_balances → Check USDC and SOL
3. kalshi_search_markets → Find market
4. kalshi_get_market → Get yesMint/noMint
5. kalshi_get_live_data → Check current prices
6. kalshi_buy_yes → Execute trade
7. kalshi_get_token_holdings → Verify position
```

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| "Market not found" | Wrong ticker format | Use kalshi_search_markets |
| "Insufficient SOL" | No gas | Swap USDC to SOL or deposit SOL |
| "Transaction failed" | Various | Check tx on Solscan |
