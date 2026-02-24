---
name: revnu
description: Manage Revnu stores, products, licenses, analytics, coupons, affiliates, purchases, and A/B tests via the Revnu CLI
---

# Revnu CLI Agent Skill

## When to Use This Skill

Use this skill when the user wants to:

- **Store info**: "what's my store?", "show my store details", "update my store name", "change my currency"
- **Products**: "list my products", "what products do I have?", "create a new product", "update product price", "delete a product"
- **Licenses**: "show my license keys", "revoke a license", "how many active licenses?", "reactivate that license"
- **Analytics**: "what's my revenue?", "show my KPIs", "how many active subscribers?", "revenue this month", "recent purchases"
- **Purchases**: "list purchases", "show purchase details", "who bought my product?", "cancelled purchases"
- **Coupons**: "create a discount code", "list my coupons", "pause a coupon", "delete that coupon"
- **Affiliates**: "show my affiliates", "invite an affiliate", "update affiliate settings", "set commission rate"
- **A/B Tests**: "create an A/B test", "show test results", "start the test", "pause the test", "which variant is winning?"
- **Auth**: "am I logged in?", "check my auth status"

## Prerequisites

```bash
npm install -g @revnu/cli
revnu auth login  # Opens browser for device code auth
```

## Agent Guidelines

1. **Always use `--json` flag** on every command so you can parse structured output
2. **Check auth once** — run `revnu auth status --json` at the start of a session to confirm the user is logged in. Tokens last 30 days, so you don't need to re-check before every command. If a command returns a "Not authenticated" error, then prompt the user to re-login.
3. **Parse JSON responses** — all commands with `--json` return structured JSON; errors return `{"error": "message"}`
4. **Prices are in cents** — when the user says "$9", pass `--price 900`; when displaying, convert cents to dollars
5. **IDs are Convex IDs** — they look like `k57abc123def456` (not UUIDs)
6. **Date format** — use `YYYY-MM-DD` for all date flags (`--from`, `--to`, `--expires`)
7. **Don't guess IDs** — if a command needs an ID, list resources first to find it
8. **Confirm destructive actions** — ask before delete, revoke, or stop operations

## Global Flags

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON (always use this as an agent) |

---

## Command Reference

### 1. auth — Authentication

#### `revnu auth login`

Log in via browser-based device code flow. Opens the browser automatically.

```bash
revnu auth login
```

> Note: This is interactive and opens a browser. Only run if user explicitly asks to log in.

#### `revnu auth logout`

Clear saved credentials.

```bash
revnu auth logout
```

#### `revnu auth status`

Check current authentication status. Returns `authenticated`, `creatorId`, `daysUntilExpiry`.

```bash
revnu auth status --json
```

---

### 2. store — Store Management

#### `revnu store get`

Get current store details. Also works as just `revnu store`.

```bash
revnu store get --json
```

#### `revnu store update`

Update store settings. At least one flag required.

| Flag | Required | Description |
|------|----------|-------------|
| `--name <name>` | No | Store name |
| `--description <desc>` | No | Store description |
| `--currency <currency>` | No | Store currency (e.g., "usd", "eur") |

```bash
revnu store update --name "New Store Name" --json
revnu store update --currency eur --description "Now in euros" --json
```

---

### 3. products — Product Management

#### `revnu products list`

List all products.

| Flag | Required | Description |
|------|----------|-------------|
| `--status <status>` | No | Filter: "active" or "draft" |

```bash
revnu products list --json
revnu products list --status active --json
```

#### `revnu products get <productId>`

Get details for a specific product.

```bash
revnu products get k57prod001 --json
```

#### `revnu products create`

Create a new product.

| Flag | Required | Description |
|------|----------|-------------|
| `--name <name>` | **Yes** | Product name |
| `--price <cents>` | **Yes** | Price in cents (e.g., 900 = $9.00) |
| `--interval <interval>` | No | "monthly" (default) or "one-time" |
| `--description <desc>` | No | Product description |
| `--features <features>` | No | Comma-separated: "license_key", "web_app", "discord" |
| `--max-devices <n>` | No | Max device activations for license keys |
| `--currency <currency>` | No | Currency override (defaults to store currency) |

```bash
revnu products create --name "Starter Plan" --price 500 --json
revnu products create --name "Pro License" --price 2900 --interval one-time --features license_key --max-devices 3 --json
revnu products create --name "Team Plan" --price 4900 --interval monthly --features license_key,web_app --description "For teams up to 10" --json
```

#### `revnu products update <productId>`

Update an existing product. At least one flag required.

| Flag | Required | Description |
|------|----------|-------------|
| `--name <name>` | No | New name |
| `--description <desc>` | No | New description |
| `--price <cents>` | No | New price in cents |
| `--active <bool>` | No | "true" or "false" |
| `--features <features>` | No | Comma-separated feature list |
| `--max-devices <n>` | No | New max device count |

```bash
revnu products update k57prod001 --price 1200 --json
revnu products update k57prod001 --name "Pro Plan v2" --active true --json
```

#### `revnu products delete <productId>`

Delete a product. **Destructive — confirm with user first.**

```bash
revnu products delete k57prod001 --json
```

---

### 4. licenses — License Key Management

#### `revnu licenses list`

List all license keys.

| Flag | Required | Description |
|------|----------|-------------|
| `--status <status>` | No | Filter: "active", "revoked", "expired" |
| `--limit <n>` | No | Max results (default: 100) |

```bash
revnu licenses list --json
revnu licenses list --status active --json
revnu licenses list --status revoked --limit 20 --json
```

#### `revnu licenses get <licenseKeyId>`

Get license details including device activations.

```bash
revnu licenses get k57lic001 --json
```

#### `revnu licenses revoke <licenseKeyId>`

Revoke an active license key. **Destructive — confirm with user first.**

```bash
revnu licenses revoke k57lic001 --json
```

#### `revnu licenses reactivate <licenseKeyId>`

Reactivate a previously revoked license key.

```bash
revnu licenses reactivate k57lic002 --json
```

---

### 5. analytics — KPIs & Revenue

#### `revnu analytics kpis`

View key performance indicators. Also works as just `revnu analytics`. Returns activeMembers, totalMembers, mrrCents, arrCents, totalSalesCents, productsCount, and statusBreakdown.

```bash
revnu analytics kpis --json
```

> When displaying to the user: convert `mrrCents` / `arrCents` / `totalSalesCents` to dollars by dividing by 100.

#### `revnu analytics revenue`

View revenue timeseries data.

| Flag | Required | Description |
|------|----------|-------------|
| `--from <date>` | No | Start date (YYYY-MM-DD) |
| `--to <date>` | No | End date (YYYY-MM-DD) |

```bash
revnu analytics revenue --json
revnu analytics revenue --from 2026-01-01 --to 2026-01-31 --json
```

#### `revnu analytics purchases`

View recent purchases summary.

| Flag | Required | Description |
|------|----------|-------------|
| `--limit <n>` | No | Number of purchases (default: 10) |

```bash
revnu analytics purchases --json
revnu analytics purchases --limit 25 --json
```

---

### 6. purchases — Purchase History

#### `revnu purchases list`

List all purchases with optional filtering.

| Flag | Required | Description |
|------|----------|-------------|
| `--status <status>` | No | Filter: "active", "cancelled", "past_due", "awaiting_join" |
| `--from <date>` | No | Start date (YYYY-MM-DD) |
| `--to <date>` | No | End date (YYYY-MM-DD) |
| `--limit <n>` | No | Max results (default: 100) |

```bash
revnu purchases list --json
revnu purchases list --status active --json
revnu purchases list --from 2026-01-01 --to 2026-01-31 --limit 50 --json
```

#### `revnu purchases get <purchaseId>`

Get full details for a specific purchase.

```bash
revnu purchases get k57purch001 --json
```

---

### 7. coupons — Discount Codes

#### `revnu coupons list`

List all coupons.

| Flag | Required | Description |
|------|----------|-------------|
| `--status <status>` | No | Filter: "active", "paused", "expired" |

```bash
revnu coupons list --json
revnu coupons list --status active --json
```

#### `revnu coupons create`

Create a new coupon.

| Flag | Required | Description |
|------|----------|-------------|
| `--code <code>` | **Yes** | 3-20 alphanumeric characters |
| `--type <type>` | **Yes** | "percentage", "fixed", or "free_trial" |
| `--value <value>` | **Yes** | Percentage (1-100), cents (fixed), or days (free_trial) |
| `--scope <scope>` | No | "all_products" (default) or "specific_products" |
| `--currency <currency>` | No | Currency for fixed-amount coupons |
| `--duration <duration>` | No | "once", "forever", or "repeating" |
| `--duration-months <n>` | No | Months for repeating duration |
| `--max-redemptions <n>` | No | Max number of uses |
| `--first-purchase-only` | No | Boolean flag, restricts to first purchase |
| `--expires <date>` | No | Expiration date (YYYY-MM-DD) |

```bash
# 20% off everything
revnu coupons create --code LAUNCH20 --type percentage --value 20 --json

# $5 off, max 50 uses, expires end of month
revnu coupons create --code SAVE5 --type fixed --value 500 --max-redemptions 50 --expires 2026-03-31 --json

# 7-day free trial
revnu coupons create --code FREETRIAL --type free_trial --value 7 --json

# 30% off forever, first purchase only
revnu coupons create --code VIP30 --type percentage --value 30 --duration forever --first-purchase-only --json
```

#### `revnu coupons update <couponId>`

Update coupon settings. At least one flag required.

| Flag | Required | Description |
|------|----------|-------------|
| `--max-redemptions <n>` | No | New max redemption count |
| `--first-purchase-only` | No | Toggle first-purchase restriction |
| `--expires <date>` | No | New expiration date (YYYY-MM-DD) |
| `--scope <scope>` | No | "all_products" or "specific_products" |

```bash
revnu coupons update k57coup001 --max-redemptions 200 --json
revnu coupons update k57coup001 --expires 2026-06-30 --json
```

#### `revnu coupons pause <couponId>`

Pause an active coupon (stops new redemptions).

```bash
revnu coupons pause k57coup001 --json
```

#### `revnu coupons activate <couponId>`

Reactivate a paused coupon.

```bash
revnu coupons activate k57coup001 --json
```

#### `revnu coupons delete <couponId>`

Delete a coupon. **Destructive — confirm with user first.**

```bash
revnu coupons delete k57coup001 --json
```

---

### 8. affiliates — Affiliate Program

#### `revnu affiliates list`

List all affiliates.

| Flag | Required | Description |
|------|----------|-------------|
| `--status <status>` | No | Filter: "invited", "active", "paused", "removed" |

```bash
revnu affiliates list --json
revnu affiliates list --status active --json
```

#### `revnu affiliates create`

Invite a new affiliate.

| Flag | Required | Description |
|------|----------|-------------|
| `--email <email>` | **Yes** | Affiliate's email address |
| `--name <name>` | **Yes** | Affiliate's display name |

```bash
revnu affiliates create --name "Jane Creator" --email jane@example.com --json
```

#### `revnu affiliates update <affiliateId>`

Update an affiliate's info.

| Flag | Required | Description |
|------|----------|-------------|
| `--name <name>` | No | New display name |
| `--email <email>` | No | New email address |

```bash
revnu affiliates update k57aff001 --name "Jane Smith" --json
```

#### `revnu affiliates delete <affiliateId>`

Remove an affiliate (soft delete). **Destructive — confirm with user first.**

```bash
revnu affiliates delete k57aff002 --json
```

#### `revnu affiliates settings`

View or update affiliate program settings. With no flags, returns current settings. With flags, updates settings.

| Flag | Required | Description |
|------|----------|-------------|
| `--enabled <bool>` | No | "true" or "false" — enable/disable affiliate program |
| `--min-payout <cents>` | No | Minimum payout threshold in cents |
| `--cookie-days <days>` | No | Referral cookie expiration in days |
| `--commission-type <type>` | No | "percentage" or "fixed" |
| `--commission-value <value>` | No | Commission amount (percent or cents) |
| `--discount-type <type>` | No | "percentage" or "fixed" — discount for referred buyers |
| `--discount-value <value>` | No | Discount amount (percent or cents) |

```bash
# View current settings
revnu affiliates settings --json

# Update commission to 25%
revnu affiliates settings --commission-type percentage --commission-value 25 --json

# Enable with full config
revnu affiliates settings --enabled true --commission-type percentage --commission-value 20 --discount-type percentage --discount-value 10 --cookie-days 30 --min-payout 5000 --json
```

---

### 9. abtests — A/B Testing

#### `revnu abtests list`

List all A/B tests.

```bash
revnu abtests list --json
```

#### `revnu abtests create`

Create a new A/B test (starts as draft).

| Flag | Required | Description |
|------|----------|-------------|
| `--name <name>` | **Yes** | Test name |
| `--variants <json>` | **Yes** | JSON array of variants — weights must sum to 100 |
| `--duration <days>` | No | Test duration in days (default: 7) |
| `--goal <event>` | No | Goal event (default: "checkout_completed") |

**Variants JSON format:** `[{"deploymentId": "dep_abc", "weight": 50, "name": "Control"}, {"deploymentId": "dep_def", "weight": 50, "name": "New Pricing"}]`

```bash
revnu abtests create --name "Pricing Test" --variants '[{"deploymentId":"dep_abc","weight":50,"name":"A"},{"deploymentId":"dep_def","weight":50,"name":"B"}]' --json

revnu abtests create --name "Long Test" --variants '[{"deploymentId":"dep_abc","weight":50,"name":"A"},{"deploymentId":"dep_def","weight":50,"name":"B"}]' --duration 30 --json
```

#### `revnu abtests results <abTestId>`

View results for an A/B test. Returns per-variant visitors, conversions, conversionRate, and revenueCents.

```bash
revnu abtests results k57ab001 --json
```

#### `revnu abtests start <abTestId>`

Start a draft or paused A/B test.

```bash
revnu abtests start k57ab002 --json
```

#### `revnu abtests pause <abTestId>`

Pause a running A/B test.

```bash
revnu abtests pause k57ab001 --json
```

#### `revnu abtests stop <abTestId>`

Stop a test and determine the winner. **Confirm with user first.**

```bash
revnu abtests stop k57ab001 --json
```

#### `revnu abtests delete <abTestId>`

Delete a draft or completed test. **Destructive — confirm with user first.**

```bash
revnu abtests delete k57ab002 --json
```

---

## Decision Trees

### User asks about revenue or metrics

1. Start with `revnu analytics kpis --json` for overview (MRR, ARR, active members)
2. If they want a time range breakdown → `revnu analytics revenue --from <date> --to <date> --json`
3. If they want to see specific transactions → `revnu analytics purchases --json` or `revnu purchases list --json`

### User wants to create a product

1. Ask for: name, price, billing interval (monthly or one-time)
2. Ask about delivery features: license keys? web app access? Discord roles?
3. If license keys: ask about max device activations
4. Run `revnu products create` with all gathered info

### User asks about customers or buyers

1. `revnu purchases list --json` — see all purchases/buyers
2. Filter by status if needed: `--status active`, `--status cancelled`
3. `revnu licenses list --json` — see license key distribution
4. `revnu analytics kpis --json` — get aggregate member counts

### User wants to run a promotion

1. Determine: percentage off, fixed amount off, or free trial?
2. `revnu coupons create` with appropriate `--type` and `--value`
3. Optionally set `--max-redemptions`, `--expires`, `--first-purchase-only`

### User asks about a specific customer issue

1. `revnu purchases list --json` — find the purchase
2. `revnu purchases get <purchaseId> --json` — get full details
3. If license related: `revnu licenses list --json` → `revnu licenses get <id> --json`
4. Can revoke/reactivate licenses as needed

### User wants to set up affiliates

1. Check current settings: `revnu affiliates settings --json`
2. Enable if needed: `revnu affiliates settings --enabled true --commission-type percentage --commission-value 20 --json`
3. Invite affiliates: `revnu affiliates create --name "Name" --email email@example.com --json`

---

## Error Handling

| Error | Cause | Recovery |
|-------|-------|----------|
| `{"error": "Not authenticated"}` | No valid credentials | Run `revnu auth login` |
| `{"error": "Token expired"}` | Auth token has expired | Run `revnu auth login` again |
| `{"error": "Store not found"}` | Creator hasn't set up a store yet | Direct user to Revnu dashboard |
| `{"error": "Product not found"}` | Invalid product ID | Run `revnu products list --json` to find correct ID |
| `{"error": "Invalid coupon code"}` | Code format violation | Code must be 3-20 alphanumeric characters |
| `{"error": "Coupon code already exists"}` | Duplicate code | Choose a different code |
| `{"error": "Cannot delete active product"}` | Product has active subscribers | Deactivate first or handle subscribers |
| Command not found | CLI not installed | Run `npm install -g @revnu/cli` |

---

## What's NOT Supported via CLI

The following operations require the Revnu web dashboard and cannot be done through the CLI:

- **Stripe Connect setup** — connecting/disconnecting Stripe accounts
- **Webhook configuration** — managing webhook endpoints
- **Storefront design** — editing storefront themes, layouts, custom domains
- **AI landing pages** — generating or editing AI-powered landing pages
- **Session replays** — viewing recorded user sessions
- **Email automation** — setting up email sequences and templates
- **Discord bot setup** — connecting Discord servers and configuring roles
- **Team management** — inviting team members, managing roles/permissions
- **Deployment management** — deploying or managing landing page versions
- **Billing/plan management** — changing the creator's Revnu subscription

