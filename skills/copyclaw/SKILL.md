---
name: copyclaw
version: 1.0.0
description: "CopyClaw: Set up a Polymarket MCP server, create a wallet, fund it, discover top traders, and auto-mirror their trades in under 2 seconds"
author: Quantish
tags: [trading, polymarket, copy-trading, prediction-markets, automation]
tools:
  - mcp__quantish__request_api_key
  - mcp__quantish__setup_wallet
  - mcp__quantish__set_approvals
  - mcp__quantish__get_wallet_status
  - mcp__quantish__get_balances
  - mcp__quantish__get_top_traders
  - mcp__quantish__get_trader_profile
  - mcp__quantish__copy_trade_follow
  - mcp__quantish__copy_trade_unfollow
  - mcp__quantish__copy_trade_list
  - mcp__quantish__copy_trade_update
  - mcp__quantish__copy_trade_history
  - mcp__quantish__copy_trade_status
  - mcp__quantish__place_order
  - mcp__quantish__get_positions
---

# CopyClaw — Polymarket Copy Trading

Mirror any Polymarket trader's buys and sells automatically. Detection via real-time WebSocket, execution in under 2 seconds.

---

## Step 0: Connect the MCP Server

Before anything works, you need the Polymarket MCP server connected in your Claude Code config.

Add this to your `~/.claude.json` (or the project's `.claude.json`) under `"mcpServers"`:

```json
{
  "mcpServers": {
    "quantish": {
      "type": "sse",
      "url": "https://quantish-sdk-production.up.railway.app/mcp",
      "headers": {
        "x-api-key": "YOUR_API_KEY_HERE"
      }
    }
  }
}
```

If you don't have an API key yet, proceed to Step 1 first using a temporary key, or ask the user for one.

---

## Step 1: Get an API Key

Call `request_api_key` to create an account and get your key:

```
request_api_key(externalId: "user@example.com")
```

- `externalId` is required — use an email, user ID, or any unique identifier
- Returns: `{ apiKey: "pk_live_...", userId: "...", isNew: true/false }`
- **Save the apiKey** — you need it for the MCP server header above
- If the user already has a key, skip this step

After getting the key, update the `x-api-key` header in the MCP server config and reconnect.

---

## Step 2: Set Up Your Wallet

Run these two commands in order:

```
setup_wallet()
```

This deploys a Gnosis Safe wallet on Polygon, generates API credentials for the Polymarket CLOB, and configures everything. It takes ~10-30 seconds. Wait for it to complete.

Then set trading approvals:

```
set_approvals()
```

This approves the USDC, CTF (conditional token), and Neg Risk contracts so your wallet can trade. Gasless via relayer.

Verify everything worked:

```
get_wallet_status()
```

You should see:
- `safeDeployed: true`
- `hasApiCredentials: true`
- `approvals: { usdc: true, ctf: true, negRisk: true }`

---

## Step 3: Fund Your Wallet

Your wallet needs **USDC on the Polygon network** to trade.

Get your deposit address:

```
get_wallet_status()
```

Look for the `safeAddress` field — that's your Polygon deposit address (starts with `0x`).

**Send USDC to that address on the Polygon network.** Both USDC.e (bridged) and native USDC work. Even $5-10 is enough to start copy trading.

Verify the deposit landed:

```
get_balances()
```

Check the `safe.usdc` field shows your balance.

**Important**: This is a Polygon address. Do NOT send tokens on Ethereum, Base, Solana, or any other chain. USDC on Polygon only.

---

## Step 4: Discover Traders to Copy

Browse the Polymarket leaderboard:

```
get_top_traders(category: "OVERALL", timePeriod: "WEEK", orderBy: "PNL", limit: 10)
```

**Parameters:**
- `category`: OVERALL, POLITICS, SPORTS, CRYPTO, CULTURE, WEATHER, ECONOMICS, TECH, FINANCE
- `timePeriod`: DAY, WEEK, MONTH, ALL
- `orderBy`: PNL (profit and loss) or VOL (volume)
- `limit`: 1-50 (default 20)

Returns a ranked list with wallet addresses, PNL, volume, and number of markets traded.

Inspect a specific trader:

```
get_trader_profile(address: "0x...")
```

Returns: display name, bio, X/Twitter handle, profile image, verification status. Useful for deciding who to follow.

---

## Step 5: Start Copy Trading

Follow a trader:

```
copy_trade_follow(
  targetWallet: "0x1234...abcd",
  allocationMode: "FIXED_AMOUNT",
  allocationValue: 2,
  maxTradeSize: 10,
  minTradeSize: 1,
  copyBuys: true,
  copySells: true
)
```

**Parameters explained:**
- `targetWallet` (required): The Polygon wallet address of the trader to copy
- `allocationMode` (optional, default PERCENTAGE): How to size your copy trades
- `allocationValue` (optional, default 10): Value for the mode (see table below)
- `maxTradeSize` (optional): Safety cap in USDC per trade
- `minTradeSize` (optional, default $1): Ignore target trades below this USDC value
- `copyBuys` (optional, default true): Mirror BUY trades
- `copySells` (optional, default true): Mirror SELL trades

### Allocation Modes

| Mode | allocationValue meaning | Example |
|------|------------------------|---------|
| `FIXED_AMOUNT` | Dollars to spend per copy trade | `2` = spend $2 per trade, regardless of target size |
| `PERCENTAGE` | Percentage of target trade value | `50` = if target buys $100, you buy $50 |
| `MIRROR_SIZE` | (ignored) Exact share count match | Target buys 10 shares, you buy 10 shares |

**Recommended for beginners**: `FIXED_AMOUNT` with `allocationValue: 2` and `maxTradeSize: 10`.

---

## Step 6: Monitor and Manage

### Check engine health
```
copy_trade_status()
```
Shows: active monitors, RTDS WebSocket connection status, detection stats, concurrent capacity.

### List your subscriptions
```
copy_trade_list()
```
Shows all followed traders with: allocation settings, copy count, success/fail counts, active status.

### View execution history
```
copy_trade_history(limit: 20)
```
Every copied trade with: status (SUCCESS/FAILED/SKIPPED), target trade details, copy size, latency in ms, error message if failed.

Filter by trader:
```
copy_trade_history(targetWallet: "0x...", limit: 50)
```

### Update settings
```
copy_trade_update(
  targetWallet: "0x...",
  allocationValue: 5,
  maxTradeSize: 20
)
```
Change any parameter on an active subscription. To pause without unfollowing:
```
copy_trade_update(targetWallet: "0x...", isActive: false)
```
Resume:
```
copy_trade_update(targetWallet: "0x...", isActive: true)
```

### Stop copying
```
copy_trade_unfollow(targetWallet: "0x...")
```
Stops monitoring immediately. **Existing positions are NOT closed** — only new trades stop being copied.

---

## How It Works

1. **RTDS WebSocket** streams every Polymarket trade in real time
2. Engine filters for trades matching your target wallets
3. When a match is found, a FOK (Fill or Kill) order is placed at the same price
4. Typical end-to-end: **<2 seconds** from target fill to your fill
5. Data API polling runs as backup (every 3 seconds) in case WebSocket disconnects

### Execution types
- **SUCCESS**: Order filled. You now hold the same position as the target.
- **FAILED**: CLOB rejected (usually insufficient USDC). Check `get_balances()`.
- **SKIPPED**: Pre-check caught it (sell with no position, trade too small, etc.)

### Limits
- **Minimum order**: $1 (Polymarket enforces this — price x size must be >= $1)
- **Maximum targets**: 150 wallets monitored simultaneously
- **Concurrent orders**: 100 simultaneous CLOB submissions
- **Detection**: Sub-second via RTDS WebSocket

---

## Troubleshooting

**"not enough balance / allowance"**
→ Your Safe wallet needs more USDC. Run `get_balances()` to check. Fund the `safeAddress` on Polygon.

**"Order too small"**
→ The calculated copy trade is below $1. Increase `allocationValue` or lower `minTradeSize`.

**"No copied position to sell"**
→ The target sold but you never bought (joined after they entered the position). Normal, engine skips it.

**copy_trade_status shows `rtds.connected: false`**
→ WebSocket disconnected. It auto-reconnects. Polling still works as backup. Check again in 30 seconds.

**Trades detected but failing**
→ Check `copy_trade_history()` for error messages. Most common: insufficient USDC, stale approval.
→ Fix: fund wallet, then `set_approvals(force: true)`.

---

## Complete Example: Zero to Copy Trading

```
# 1. Get API key
request_api_key(externalId: "myemail@example.com")
# → Save the apiKey, configure MCP server with it

# 2. Set up wallet
setup_wallet()
set_approvals()

# 3. Get deposit address
get_wallet_status()
# → Send USDC to the safeAddress on Polygon

# 4. Verify funds arrived
get_balances()

# 5. Find a profitable trader
get_top_traders(category: "CRYPTO", timePeriod: "WEEK", orderBy: "PNL", limit: 5)
get_trader_profile(address: "0x...")

# 6. Start copying with $2 per trade
copy_trade_follow(targetWallet: "0x...", allocationMode: "FIXED_AMOUNT", allocationValue: 2, maxTradeSize: 10)

# 7. Check it's running
copy_trade_status()
copy_trade_list()

# 8. Later — check how it's going
copy_trade_history(limit: 10)
get_positions()
get_balances()
```
