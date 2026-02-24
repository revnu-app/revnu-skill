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

The user must have the Revnu CLI installed and authenticated:

```bash
# Install globally
npm install -g @revnu/cli

# Authenticate (opens browser for device code flow)
revnu auth login
```

**Always check auth status first** before running any other command if you're unsure whether the user is logged in.

## Agent Guidelines

1. **Always use `--json` flag** on every command so you can parse structured output
2. **Check auth once** — run `revnu auth status --json` at the start of a session to confirm the user is logged in. Tokens last 30 days, so you don't need to re-check before every command. If a command returns a "Not authenticated" error, then prompt the user to re-login.
3. **Parse JSON responses** — all commands with `--json` return structured JSON; errors return `{"error": "message"}`
4. **Prices are in cents** — when the user says "$9", pass `--price 900`; when displaying, convert cents to dollars
6. **IDs are Convex IDs** — they look like `k57abc123def456` (not UUIDs)
7. **Date format** — use `YYYY-MM-DD` for all date flags (`--from`, `--to`, `--expires`)
8. **Don't guess IDs** — if a command needs an ID, list resources first to find it
9. **Confirm destructive actions** — ask before delete, revoke, or stop operations

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

Check current authentication status.

```bash
revnu auth status --json
```

**Example output:**
```json
{
  "authenticated": true,
  "apiBaseUrl": "https://revnu.app",
  "tokenPrefix": "abc123def456...",
  "expiresAt": "2027-02-15T00:00:00.000Z",
  "daysUntilExpiry": 365,
  "creatorId": "k57abc123def456"
}
```

**Unauthenticated output:**
```json
{
  "authenticated": false,
  "apiBaseUrl": "https://revnu.app"
}
```

---

### 2. store — Store Management

#### `revnu store get`

Get current store details. Also works as just `revnu store`.

```bash
revnu store get --json
```

**Example output:**
```json
{
  "store": {
    "_id": "k57store123",
    "name": "My Awesome Store",
    "subdomain": "awesome",
    "description": "The best software store",
    "currency": "usd",
    "createdAt": 1708123456789
  }
}
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

**Example output:**
```json
{
  "success": true,
  "store": {
    "_id": "k57store123",
    "name": "New Store Name",
    "subdomain": "awesome",
    "description": "Now in euros",
    "currency": "eur"
  }
}
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

**Example output:**
```json
[
  {
    "_id": "k57prod001",
    "name": "Pro Plan",
    "priceCents": 900,
    "interval": "monthly",
    "status": "active",
    "features": ["license_key"],
    "maxDevices": 3,
    "description": "Full access to all features"
  },
  {
    "_id": "k57prod002",
    "name": "Lifetime License",
    "priceCents": 4900,
    "interval": "one-time",
    "status": "active",
    "features": ["license_key", "discord"],
    "maxDevices": 5
  }
]
```

#### `revnu products get <productId>`

Get details for a specific product.

```bash
revnu products get k57prod001 --json
```

**Example output:**
```json
{
  "_id": "k57prod001",
  "name": "Pro Plan",
  "priceCents": 900,
  "interval": "monthly",
  "status": "active",
  "features": ["license_key"],
  "maxDevices": 3,
  "description": "Full access to all features",
  "createdAt": 1708123456789
}
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

**Example output:**
```json
{
  "success": true,
  "product": {
    "_id": "k57prod003",
    "name": "Starter Plan",
    "priceCents": 500,
    "interval": "monthly",
    "status": "active",
    "features": []
  }
}
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

**Example output:**
```json
{
  "success": true,
  "product": {
    "_id": "k57prod001",
    "name": "Pro Plan v2",
    "priceCents": 1200,
    "interval": "monthly",
    "status": "active"
  }
}
```

#### `revnu products delete <productId>`

Delete a product. **Destructive — confirm with user first.**

```bash
revnu products delete k57prod001 --json
```

**Example output:**
```json
{
  "success": true
}
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

**Example output:**
```json
[
  {
    "_id": "k57lic001",
    "key": "REVNU-XXXX-XXXX-XXXX",
    "productName": "Pro Plan",
    "buyerEmail": "buyer@example.com",
    "status": "active",
    "activationCount": 2,
    "maxDevices": 3,
    "createdAt": 1708123456789
  },
  {
    "_id": "k57lic002",
    "key": "REVNU-YYYY-YYYY-YYYY",
    "productName": "Lifetime License",
    "buyerEmail": "user2@example.com",
    "status": "revoked",
    "activationCount": 1,
    "maxDevices": 5,
    "createdAt": 1707123456789
  }
]
```

#### `revnu licenses get <licenseKeyId>`

Get license details including device activations.

```bash
revnu licenses get k57lic001 --json
```

**Example output:**
```json
{
  "_id": "k57lic001",
  "key": "REVNU-XXXX-XXXX-XXXX",
  "productName": "Pro Plan",
  "buyerEmail": "buyer@example.com",
  "status": "active",
  "activationCount": 2,
  "maxDevices": 3,
  "activations": [
    {
      "deviceId": "device_abc123",
      "deviceName": "MacBook Pro",
      "activatedAt": 1708200000000
    },
    {
      "deviceId": "device_def456",
      "deviceName": "Windows Desktop",
      "activatedAt": 1708300000000
    }
  ],
  "createdAt": 1708123456789
}
```

#### `revnu licenses revoke <licenseKeyId>`

Revoke an active license key. **Destructive — confirm with user first.**

```bash
revnu licenses revoke k57lic001 --json
```

**Example output:**
```json
{
  "success": true
}
```

#### `revnu licenses reactivate <licenseKeyId>`

Reactivate a previously revoked license key.

```bash
revnu licenses reactivate k57lic002 --json
```

**Example output:**
```json
{
  "success": true
}
```

---

### 5. analytics — KPIs & Revenue

#### `revnu analytics kpis`

View key performance indicators. Also works as just `revnu analytics`.

```bash
revnu analytics kpis --json
```

**Example output:**
```json
{
  "activeMembers": 142,
  "totalMembers": 387,
  "mrrCents": 127800,
  "arrCents": 1533600,
  "totalSalesCents": 892400,
  "productsCount": 5,
  "statusBreakdown": {
    "active": 142,
    "cancelled": 89,
    "past_due": 12,
    "awaiting_join": 144
  }
}
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

**Example output:**
```json
[
  {
    "date": "2026-01-15",
    "revenueCents": 4500,
    "purchaseCount": 3,
    "uniqueBuyers": 3
  },
  {
    "date": "2026-01-16",
    "revenueCents": 9900,
    "purchaseCount": 5,
    "uniqueBuyers": 4
  }
]
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

**Example output:**
```json
[
  {
    "_id": "k57purch001",
    "productName": "Pro Plan",
    "buyerEmail": "buyer@example.com",
    "status": "active",
    "createdAt": 1708123456789
  }
]
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

**Example output:**
```json
[
  {
    "_id": "k57purch001",
    "productName": "Pro Plan",
    "buyerUsername": "cooldev",
    "buyerEmail": "cool@example.com",
    "status": "active",
    "priceCents": 900,
    "createdAt": 1708123456789
  },
  {
    "_id": "k57purch002",
    "productName": "Lifetime License",
    "buyerUsername": "hackerx",
    "buyerEmail": "hacker@example.com",
    "status": "cancelled",
    "priceCents": 4900,
    "createdAt": 1707123456789
  }
]
```

#### `revnu purchases get <purchaseId>`

Get full details for a specific purchase.

```bash
revnu purchases get k57purch001 --json
```

**Example output:**
```json
{
  "_id": "k57purch001",
  "productName": "Pro Plan",
  "buyerUsername": "cooldev",
  "buyerEmail": "cool@example.com",
  "status": "active",
  "priceCents": 900,
  "interval": "monthly",
  "stripeSubscriptionId": "sub_xxx",
  "createdAt": 1708123456789
}
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

**Example output:**
```json
[
  {
    "_id": "k57coup001",
    "code": "LAUNCH20",
    "type": "percentage",
    "value": 20,
    "status": "active",
    "redemptions": 15,
    "maxRedemptions": 100,
    "createdAt": 1708123456789
  },
  {
    "_id": "k57coup002",
    "code": "FREETRIAL7",
    "type": "free_trial",
    "value": 7,
    "status": "active",
    "redemptions": 42,
    "createdAt": 1707123456789
  }
]
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

**Example output:**
```json
{
  "success": true,
  "coupon": {
    "_id": "k57coup003",
    "code": "LAUNCH20",
    "type": "percentage",
    "value": 20,
    "status": "active",
    "scope": "all_products",
    "redemptions": 0
  }
}
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

**Example output:**
```json
{
  "success": true,
  "coupon": {
    "_id": "k57coup001",
    "code": "LAUNCH20",
    "maxRedemptions": 200
  }
}
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

**Example output (pause/activate/delete):**
```json
{
  "success": true
}
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

**Example output:**
```json
[
  {
    "_id": "k57aff001",
    "name": "Jane Creator",
    "email": "jane@example.com",
    "status": "active",
    "totalEarningsCents": 15000,
    "pendingBalanceCents": 3200
  },
  {
    "_id": "k57aff002",
    "name": "Bob Promoter",
    "email": "bob@example.com",
    "status": "invited",
    "totalEarningsCents": 0,
    "pendingBalanceCents": 0
  }
]
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

**Example output:**
```json
{
  "success": true,
  "affiliate": {
    "_id": "k57aff003",
    "name": "Jane Creator",
    "email": "jane@example.com",
    "status": "invited"
  }
}
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

**Example output:**
```json
{
  "success": true,
  "affiliate": {
    "_id": "k57aff001",
    "name": "Jane Smith",
    "email": "jane@example.com"
  }
}
```

#### `revnu affiliates delete <affiliateId>`

Remove an affiliate (soft delete). **Destructive — confirm with user first.**

```bash
revnu affiliates delete k57aff002 --json
```

**Example output:**
```json
{
  "success": true
}
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

**Example output (view):**
```json
{
  "enabled": true,
  "commissionType": "percentage",
  "commissionValue": 20,
  "discountType": "percentage",
  "discountValue": 10,
  "cookieDays": 30,
  "minPayoutCents": 5000
}
```

---

### 9. abtests — A/B Testing

#### `revnu abtests list`

List all A/B tests.

```bash
revnu abtests list --json
```

**Example output:**
```json
[
  {
    "_id": "k57ab001",
    "name": "Pricing Page Test",
    "status": "running",
    "variantCount": 2,
    "durationDays": 14,
    "createdAt": 1708123456789
  },
  {
    "_id": "k57ab002",
    "name": "CTA Button Color",
    "status": "draft",
    "variantCount": 3,
    "durationDays": 7,
    "createdAt": 1707123456789
  }
]
```

#### `revnu abtests create`

Create a new A/B test (starts as draft).

| Flag | Required | Description |
|------|----------|-------------|
| `--name <name>` | **Yes** | Test name |
| `--variants <json>` | **Yes** | JSON array of variants (see format below) |
| `--duration <days>` | No | Test duration in days (default: 7) |
| `--goal <event>` | No | Goal event (default: "checkout_completed") |

**Variants JSON format:**
```json
[
  {"deploymentId": "dep_abc123", "weight": 50, "name": "Control"},
  {"deploymentId": "dep_def456", "weight": 50, "name": "New Pricing"}
]
```

> Weights should sum to 100. Each variant needs a `deploymentId` (the landing page deployment to show).

```bash
revnu abtests create --name "Pricing Test" --variants '[{"deploymentId":"dep_abc","weight":50,"name":"A"},{"deploymentId":"dep_def","weight":50,"name":"B"}]' --json

revnu abtests create --name "Long Test" --variants '[{"deploymentId":"dep_abc","weight":50,"name":"A"},{"deploymentId":"dep_def","weight":50,"name":"B"}]' --duration 30 --json
```

**Example output:**
```json
{
  "success": true,
  "abTest": {
    "_id": "k57ab003",
    "name": "Pricing Test",
    "status": "draft",
    "variantCount": 2,
    "durationDays": 7
  }
}
```

#### `revnu abtests results <abTestId>`

View results for an A/B test.

```bash
revnu abtests results k57ab001 --json
```

**Example output:**
```json
{
  "_id": "k57ab001",
  "name": "Pricing Page Test",
  "status": "running",
  "variants": [
    {
      "name": "Control",
      "visitors": 523,
      "conversions": 42,
      "conversionRate": 8.03,
      "revenueCents": 37800
    },
    {
      "name": "New Pricing",
      "visitors": 519,
      "conversions": 61,
      "conversionRate": 11.75,
      "revenueCents": 48900
    }
  ],
  "winner": null,
  "daysRemaining": 5
}
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

**Example output (start/pause/stop/delete):**
```json
{
  "success": true
}
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

### Common Errors

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

### General Error Format

All errors with `--json` return:
```json
{
  "error": "Error description here"
}
```

Without `--json`, errors display as formatted text with a red indicator.

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

