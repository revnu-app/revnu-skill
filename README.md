# Revnu Agent Skill

Sell software, SaaS, subscriptions, and digital products from your AI coding agent. [Revnu](https://revnu.app) is the Shopify for selling software — hosted checkout, storefronts, subscriptions, license key delivery, and growth tools — all managed through a CLI that any coding agent can use.

This skill teaches AI coding agents how to manage your [Revnu](https://revnu.app) store: create products, track revenue, manage license keys, run promotions, set up affiliates, and more.

## Install

Works with **40+ coding agents** including Claude Code, Cursor, GitHub Copilot, Windsurf, Gemini CLI, OpenAI Codex, and more.

```bash
npx skills add revnu-app/revnu-skill
```

### Install for a specific agent

```bash
# Claude Code
npx skills add revnu-app/revnu-skill -a claude-code

# Cursor
npx skills add revnu-app/revnu-skill -a cursor

# GitHub Copilot
npx skills add revnu-app/revnu-skill -a github-copilot

# All agents at once
npx skills add revnu-app/revnu-skill --all
```

### Global install (available across all projects)

```bash
npx skills add revnu-app/revnu-skill -g
```

## Prerequisites

You need a [Revnu](https://revnu.app) account and the CLI authenticated:

```bash
npx @revnu/setup auth login
```

## What can your agent do with this skill?

| Category | Examples |
|----------|----------|
| **Products** | Create, update, delete, list products with pricing and delivery features |
| **Subscriptions** | Monthly and one-time billing, metered/usage-based pricing |
| **License Keys** | Generate, activate, revoke, and manage per-device license keys |
| **Analytics** | MRR, ARR, active subscribers, revenue timeseries, purchase history |
| **Coupons** | Percentage off, fixed amount, free trials, expiration, usage limits |
| **Affiliates** | Invite affiliates, set commission rates, manage payouts |
| **A/B Tests** | Create pricing experiments, track conversions, pick winners |
| **Store Management** | Update store name, description, currency |

## Example conversations

Once installed, just talk to your agent naturally:

- "What's my MRR this month?"
- "Create a product called Pro Plan at $29/month with license key delivery"
- "Show me all active subscribers"
- "Create a 20% off coupon code LAUNCH20 that expires March 31"
- "Set up an affiliate program with 15% commission"
- "Which A/B test variant is winning?"
- "Revoke the license key for customer X"

## What is Revnu?

[Revnu](https://revnu.app) is a full commerce platform for software sellers. Instead of stitching together Stripe, a license server, a storefront builder, and analytics yourself, Revnu gives you everything in one platform:

- **Hosted checkout** with Stripe Connect
- **Storefronts** with custom subdomains
- **AI-generated landing pages**
- **License key delivery** with per-device activation
- **RevnuAuth SDK** for web app access control
- **Subscriptions and one-time payments**
- **Usage-based / metered billing**
- **A/B testing** for pricing experiments
- **Affiliate system** with automatic payouts
- **Coupon engine** with percentage, fixed, and free trial support
- **Analytics dashboard** with MRR, ARR, churn, and more

Whether you're an indie hacker, vibe coder, or dev tool company, Revnu handles the commerce infrastructure so you can focus on building.

[Get started at revnu.app](https://revnu.app)

## Skill structure

```
revnu-skill/
├── skills/
│   └── revnu/
│       └── SKILL.md    # Agent instructions and CLI reference
├── README.md
└── LICENSE
```

This skill follows the open [Agent Skills specification](https://agentskills.io).

## Supported agents

This skill works with any agent that supports the [Agent Skills](https://agentskills.io) format, including:

Claude Code, Cursor, GitHub Copilot, VS Code, OpenAI Codex, Windsurf, Gemini CLI, Goose, Roo Code, OpenHands, Junie, Kiro, Amp, OpenCode, Cline, TRAE, Mux, and [many more](https://agentskills.io).

## Links

- [Revnu](https://revnu.app) — The Shopify for selling software
- [Agent Skills spec](https://agentskills.io) — The open standard this skill follows
- [skills.sh](https://skills.sh) — Browse and discover agent skills
- [@revnu/setup on npm](https://www.npmjs.com/package/@revnu/setup) — CLI setup tool

## License

MIT
