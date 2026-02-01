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

Then restart Claude Code and run `/quantish-mcp-setup`.

## What Gets Set Up

| Server | Platform | What You Can Do |
|--------|----------|-----------------|
| quantish-discovery | All | Search 50k+ markets, get prices, find arbitrage |
| quantish | Polymarket | Place trades, manage positions, transfer funds |
| quantish-kalshi | Kalshi | Trade US-regulated markets via Solana/DFlow |

## Links

- [Quantish](https://quantish.live) - Web app
- [Docs](https://docs.quantish.live) - Full documentation
- [Discord](https://discord.gg/quantish) - Community

## License

MIT
