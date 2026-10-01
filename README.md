# cf-domain-hunter

English | [中文](README_CN.md)

A standard skill that finds, verifies, and prices domain names using the free **Cloudflare CLI (`cf`)** — no API keys, no web scraping, no third-party services. Works with Claude Code, Pi, Codex, and other skill-compatible agents.

Every availability result is an **authoritative real-time registry check** with Cloudflare's at-cost pricing. Only verified-available domains are ever presented to the user.

## Features

- 🔍 **Suggest** — generate brandable, short domain candidates from a business description (double-pinyin friendly for Chinese users)
- ✅ **Verify** — authoritative availability check via `cf registrar registrations check` (real registry, not cache)
- 💰 **Price** — Cloudflare at-cost USD pricing for registration and renewal
- 🛡️ **Dependency guard** — detects missing `cf` CLI or missing login and walks the user through setup before any query

## Requirements

- Any [Agent Skills](https://agentskills.io)–compatible agent (Claude Code, Pi, etc.)
- [Cloudflare CLI (`cf`)](https://developers.cloudflare.com/cloudflare-cli/) installed **and logged in**:

```bash
npm install -g @cloudflare/cli
cf auth login
```

> Domain availability/pricing endpoints are account-scoped. Without login they return `[9106] Authentication failed` — the skill detects this and guides you through `cf auth login` automatically.

## Installation

### Option 1: One command with the skills CLI (recommended)

The [skills CLI](https://github.com/vercel-labs/skills) installs directly from GitHub and auto-detects your agent (Claude Code, Pi, Codex, Cursor, and 80+ more):

```bash
npx skills add https://github.com/ifyour/cf-domain-hunter
```

Add `-g` for user-level (all projects) install; default is project-level.

### Option 2: Natural language

Just send your agent the repo link and ask:

> Install the skill from https://github.com/ifyour/cf-domain-hunter

The agent will run the install command for you and pick the right skills directory.

## Usage

### 1. Natural language (auto-trigger)

Just describe what you want — the agent matches the skill by its description:

```
帮我找几个适合小学生课外学习视频业务的域名，最好双拼
```

```
Find me 10 available domains for a kids' cartoon learning site, under $15/year
```

### 2. Explicit skill command

```
/cf-domain-hunter 免费图片托管相关的域名，.com 和 .xyz 都看看
```

(Claude Code uses `/skill:cf-domain-hunter …` syntax; other hosts may differ.)

Arguments after the command are appended to the skill as your request.

### 3. Manual reference

Mention the skill name in your prompt: "用 cf-domain-hunter 的流程帮我查一下 katong.tv 可不可以注册".

## What you get

```
## Recommendation

**Domain:** katong.tv
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

## How it works

| Step | Command | Notes |
|------|---------|-------|
| 0. Dependency check | `cf auth whoami` | Stops and guides install/login if missing |
| 1. Generate ideas | — | 10–20 short, brandable candidates |
| 2. Suggestions (optional) | `cf registrar registrations search` | Cached, non-authoritative |
| 3. Verify (required) | `cf registrar registrations check` | Real-time registry, max 20 domains/request, returns pricing |
| 4. Recommend | — | Only `registrable: true` domains, waits for your confirmation |

## Notes & Limitations

- Registration itself is done via the Cloudflare dashboard (`https://dash.cloudflare.com/?to=/:account/domains/register`) — the check/verify flow is fully CLI-based
- `domain_premium` domains cannot be registered via the API (dashboard only, higher price)
- Max 20 domains per check request
- Pricing is Cloudflare's at-cost USD price — usually the cheapest retail option, so no separate price comparison is needed

## License

MIT
