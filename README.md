# VeriGlow Agent Map Skill

> Teach your AI agent how to discover and use hidden APIs, data functions, and browser automation recipes for any website.

[VeriGlow Agent Map](https://veri-glow.com) is a registry of Agent-readable documentation for websites. This skill enables AI agents to look up any URL and get structured instructions for extracting data from it.

## What It Does

When installed, this skill teaches your agent to:

1. **Look up any URL** on VeriGlow Agent Map (`veri-glow.com/{url}`)
2. **Call hidden APIs** directly using documented endpoints and parameters
3. **Automate browsers** as a fallback, with tested Playwright/Puppeteer recipes
4. **Avoid known pitfalls** by reading real-world Agent Reports

## Install

### Claude Code

```bash
claude plugin install github:ChizhongWang/veriglow-agent-map-skill
```

Or install from the Claude Code Plugin Marketplace:
```
/plugin install veriglow-agent-map
```

### OpenClaw (ClawHub)

```bash
clawhub install veriglow-agent-map
```

Or search for `veriglow-agent-map` on [ClawHub](https://clawhub.ai).

### Manual

Copy the `skills/veriglow-agent-map/` directory into your agent's skills folder.

## Usage

Once installed, the skill activates automatically when your agent needs to extract data from a website. You can also invoke it explicitly:

**Claude Code:**
```
/veriglow-agent-map
```

**Example prompts that trigger this skill:**
- "Get the bond trading data from the Shanghai Stock Exchange"
- "How can I scrape data from www.sse.com.cn?"
- "Find the API behind this web page: https://..."

## Currently Indexed Maps

| Source | Data |
|--------|------|
| [SSE Bond Trading (Daily)](https://veri-glow.com/www.sse.com.cn/market/bonddata/overview/day/) | 17 bond categories, 4 trading metrics |

More maps are being added continuously. Visit [veri-glow.com](https://veri-glow.com) to browse all available maps.

## License

MIT
