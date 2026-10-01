# cf-domain-hunter

[English](README.md) | 中文

一个兼容 [Agent Skills](https://agentskills.io) 规范的技能，基于免费的 **Cloudflare CLI（`cf`）** 查找、验证域名并获取真实价格——无需 API Key、不爬网页、不依赖第三方服务。适用于任何遵循 Agent Skills 规范的智能体（Claude Code、Pi 等）。

所有可注册结果都是**注册局实时权威查询**，并附带 Cloudflare 成本价。技能只会向你展示确认可注册的域名。

## 功能特性

- 🔍 **智能推荐** — 根据业务描述生成简短、有品牌感的候选域名（对中文用户优先双拼）
- ✅ **权威验证** — 通过 `cf registrar registrations check` 实时查询注册局（非缓存）
- 💰 **真实价格** — Cloudflare 成本价（美元），含首年注册价与续费价
- 🛡️ **依赖自检** — 自动检测 `cf` 未安装或未登录，并引导你完成安装/登录后再查询

## 环境要求

- 任何兼容 [Agent Skills](https://agentskills.io) 规范的智能体（Claude Code、Pi 等）
- [Cloudflare CLI（`cf`）](https://developers.cloudflare.com/cloudflare-cli/) 已安装**且已登录**：

```bash
npm install -g @cloudflare/cli
cf auth login
```

> 域名可注册性/价格接口是账号级别的。未登录会返回 `[9106] Authentication failed`——技能会检测到并自动引导你执行 `cf auth login`。

## 安装

### 方式一：skills CLI 一条命令安装（推荐）

[skills CLI](https://github.com/vercel-labs/skills) 支持直接从 GitHub 安装，并自动检测你的智能体（Claude Code、Pi、Codex、Cursor 等 80+）：

```bash
npx skills add https://github.com/ifyour/cf-domain-hunter
```

加 `-g` 安装到用户级（所有项目可用）；默认安装到项目级。

### 方式二：自然语言安装

直接把仓库链接发给你的智能体：

> 帮我安装 https://github.com/ifyour/cf-domain-hunter 这个技能

智能体会替你执行安装命令，并选择正确的技能目录。

## 使用方法

### 方式一：自然语言（自动触发）

直接描述需求即可，智能体会根据技能描述自动匹配加载：

```
帮我找几个适合小学生课外学习视频业务的域名，最好双拼
```

```
Find me 10 available domains for a kids' cartoon learning site, under $15/year
```

### 方式二：显式技能命令

```
/cf-domain-hunter 免费图片托管相关的域名，.com 和 .xyz 都看看
```

（Claude Code 使用 `/skill:cf-domain-hunter …` 语法，其他宿主可能略有不同。）

命令后面的内容会作为你的具体需求附加给技能。

### 方式三：手动引用

在 prompt 中提及技能名称：「用 cf-domain-hunter 的流程帮我查一下 katong.tv 可不可以注册」。

## 输出效果

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

## 工作原理

| 步骤 | 命令 | 说明 |
|------|------|------|
| 0. 依赖检查 | `cf auth whoami` | 未安装/未登录时停下来并给出引导 |
| 1. 生成候选 | — | 10–20 个简短、有品牌感的候选名 |
| 2. 建议搜索（可选） | `cf registrar registrations search` | 缓存结果，非权威 |
| 3. 权威验证（必须） | `cf registrar registrations check` | 注册局实时查询，每次最多 20 个，返回实价 |
| 4. 输出推荐 | — | 只展示 `registrable: true` 的域名，等你确认后再操作 |

## 注意事项与限制

- 实际注册通过 Cloudflare 控制台完成（`https://dash.cloudflare.com/?to=/:account/domains/register`）——查询/验证流程完全基于 CLI
- `domain_premium`（溢价域名）无法通过 API 注册（仅控制台，价格更高）
- 每次验证请求最多 20 个域名
- 返回的是 Cloudflare 成本价（美元）——通常已是最低零售价，无需再单独比价

## 许可证

MIT
