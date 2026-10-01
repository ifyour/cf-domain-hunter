---
name: cf-domain-hunter
description: Search domains, check availability, get real pricing, purchase recommendations. Use when user wants to buy a domain, check domain availability or prices, or search for available domain names. All checks use the free Cloudflare CLI (cf).
---

# Domain Hunter Skill

Help users find and purchase domain names. All availability and pricing checks run locally via the **free Cloudflare CLI (`cf`)** — no API keys, no web scraping.

## Workflow

### Step 0: Dependency Check (cf CLI installed and logged in)

All queries are account-scoped — they FAIL without authentication. Before anything else:

```bash
command -v cf >/dev/null 2>&1 || { echo "CF_CLI_MISSING"; }
cf auth whoami 2>&1 | grep -q '"authenticated": true' && echo OK || echo "CF_NOT_AUTHED"
```

- **CF_CLI_MISSING** → stop and guide the user:
  `npm install -g @cloudflare/cli` (or see `https://developers.cloudflare.com/cloudflare-cli/`), then re-run this check
- **CF_NOT_AUTHED** → stop and guide:
  `cf auth login` (opens browser OAuth flow), then re-run this check
- **OK** → proceed to Step 1

Never continue to domain checks on a failed dependency check.

### Step 1: Generate Domain Ideas

Based on the user's business/project description, generate 10-20 candidate domain names.

**Guidelines:**
- Keep names short (under 15 characters), prefer double pinyin for Chinese users
- Make them memorable and brandable
- Consider: `{noun}{noun}`, `{noun}{suffix}` (ke, tv, shop, vip), `{prefix}{keyword}`
- Cover both .com and video/commerce-flavored TLDs (.tv, .shop, .vip, .xyz, .icu)

### Step 2: Search Suggestions (optional, non-authoritative)

Use the suggestion endpoint to discover more variants with cached pricing:

```bash
cf registrar registrations search --query "<keyword or phrase>" --limit 20
# Optional: --extensions "com,tv,shop"  (unsupported extensions are silently ignored)
```

Note: results are **cached and non-authoritative**. Always confirm with Step 3.

### Step 3: Authoritative Availability Check (REQUIRED before presenting)

Real-time registry check, max **20 domains per request**:

```bash
# NOTE: the command requires --body; the positional arg is ignored but must not clash.
cf registrar registrations check "x" --body '{"domains":["example.com","katong.tv","xiaoxuetv.com"]}'
```

Response per domain:
- `registrable: true` → **available**, includes `pricing` (registration_cost / renewal_cost in USD)
- `reason: domain_unavailable` → taken
- `reason: domain_premium` → premium-priced, cannot register via API (dashboard only, higher price)
- `reason: extension_not_supported` → TLD unsupported by Cloudflare Registrar

**CRITICAL:**
- Only present domains with `registrable: true`
- Present suggestions and **wait for user confirmation** before any registration
- Registration itself: guide the user to the Cloudflare dashboard at `https://dash.cloudflare.com/?to=/:account/domains/register` (API registration of premium domains is unsupported; dashboard covers all cases). Confirm price with user first.

### Step 4: Recommend

Present final recommendation in this format:

```
## Recommendation

**Domain:** example.tv
**Price:** $25.00/year (same registration and renewal)
**Registrar:** Cloudflare (at-cost, free WHOIS privacy)
**Verified:** ✅ registrable: true

### Available Candidates (verified in real time)
| Domain | Year 1 | Renewal |
|--------|--------|---------|
| katong.tv | $25.00 | $25.00 |
| xiaoxuetv.com | $10.46 | $10.46 |

### Unavailable
- donghua.com — already registered
- xueke.vip — premium-priced domain
```

## Notes

- All registrar endpoints are **account-scoped** — an unauthenticated call returns `[9106] Authentication failed`; run Step 0 first and surface the login guidance instead of a raw error
- `cf registrar registrations check` **requires** `--body '{"domains":[...]}'` — a bare positional list of domains is parsed as subcommands and errors
- With `--body`, the positional argument is still required by yargs and its value IS sent to the API; pass a dummy like `"x"`, never a real domain
- Max 20 domains per check request
- `--quiet`/`-q` suppresses help noise; output is JSON
- Pricing returned is Cloudflare's at-cost USD price — usually the cheapest retail option, no separate price comparison needed
