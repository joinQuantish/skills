# Quantish Skills

Claude Code skills for prediction market trading.

## Available Skills

### quantish-mcp-setup

One command to set up complete prediction market trading:

```
/quantish-mcp-setup
```

Sets up 3 MCP servers:
- **Discovery** - Search markets across Polymarket, Kalshi, Limitless
- **Polymarket** - Trade on Polymarket (crypto, global)
- **Kalshi** - Trade on Kalshi (CFTC-regulated, US)

### polymarket-trading

Complete Polymarket trading guide with all tool calls:

```
/polymarket-trading
```

Covers:
- Wallet setup and balances
- Finding markets via Discovery
- Checking orderbook (best bid/ask)
- Placing and canceling orders
- Managing positions
- Claiming winnings

### kalshi-trading

Complete Kalshi trading guide (Solana/DFlow):

```
/kalshi-trading
```

Covers:
- Wallet and balance management
- Market search and live data
- Buying YES/NO shares
- Token holdings and positions
- Redemption after settlement
- SOL/USDC swaps

## Installation

### Option 1: Clone to skills directory

```bash
git clone https://github.com/joinQuantish/skills.git ~/.claude/skills/quantish
```

### Option 2: Copy individual skill

```bash
mkdir -p ~/.claude/skills/quantish-mcp-setup
curl -o ~/.claude/skills/quantish-mcp-setup/SKILL.md \
  https://raw.githubusercontent.com/joinQuantish/skills/main/skills/quantish-mcp-setup/SKILL.md
```

Then restart Claude Code and run the skill.

## What Gets Set Up

| Server | Platform | What You Can Do |
|--------|----------|-----------------|
| quantish-discovery | All | Search 50k+ markets, get prices, find arbitrage |
| quantish | Polymarket | Place trades, manage positions, transfer funds |
| quantish-kalshi | Kalshi | Trade US-regulated markets via Solana/DFlow |

## Quick Start

1. Run `/quantish-mcp-setup` to connect to servers
2. Restart Claude Code
3. Run `/polymarket-trading` or `/kalshi-trading` for trading guides

## Example Commands (After Setup)

```
"Search for government shutdown markets"
"Show my Polymarket balance"
"Buy $2 of YES on the Fed Chair market"
"Show my Kalshi positions"
```

## Links

- [Quantish](https://quantish.live) - Web app
- [Docs](https://docs.quantish.live) - Full documentation
- [Discord](https://discord.gg/quantish) - Community

## License

MIT
