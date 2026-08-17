---
name: vectoree
version: 0.4.0
description: Use when connecting an app to Vectoree Cloud, listing or calling models through the Vectoree gateway, managing project database or storage via @vectoree/cli, migrating OpenAI SDK calls, or pointing Codex at Vectoree. Install with npx skills add VectoreeAI/vectoree-skills.
homepage: https://github.com/VectoreeAI/vectoree-skills
cli_package: "@vectoree/cli"
api_base_hint: Default API origin is https://vectoree.ai (override with VECTOREE_API_URL if needed)
last_updated: 2026-08-17
---

# Vectoree for AI Coding Agents

**One CLI (`vectoree`).** Skill = scene orchestration. CLI = atomic commands. Do not invent platform APIs; call `@vectoree/cli`, then change application code.

**Install:** `npx skills add VectoreeAI/vectoree-skills`

**Canonical files:** [`skills/vectoree/SKILL.md`](https://github.com/VectoreeAI/vectoree-skills/blob/main/skills/vectoree/SKILL.md) (what `npx skills add` copies). Raw fallback: `https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/SKILL.md`

This skill is for **developers and their coding agents** wiring Vectoree Cloud into an app (or into Codex). It is not Skills Hub, not a runtime tool catalog, and not a product the end user clicks.

Two surfaces. Do not mix them:

| Surface | What | Who |
|---------|------|-----|
| **CLI** | `npx @vectoree/cli …` | Agent ops: login, link, db, storage, probe models |
| **OpenAI-compatible HTTP** | `POST /api/v1/chat/completions` | App runtime inference |

```text
Developer → coding agent → this skill → @vectoree/cli + app code → Vectoree Cloud
```

---

## Onboarding (2 steps)

### 1) Install the skill

This is the [Agent Skills](https://skills.sh) install command. It copies `SKILL.md` into Cursor / Claude Code / Codex / etc.

```bash
npx skills add VectoreeAI/vectoree-skills
```

Project-local (default): `./.agents/skills/vectoree`. Global: add `-g`. Then tell the agent:

```text
Set up Vectoree as the backend for this project.
The Vectoree skill is installed. Follow it.
1. npx @vectoree/cli login
2. npx @vectoree/cli link
3. npx @vectoree/cli ai status
```

If the agent cannot load installed skills, paste this fallback:

```text
Set up Vectoree as the backend for this project.
Follow the skill at:
https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/SKILL.md

1. npx @vectoree/cli login
2. npx @vectoree/cli link
3. npx @vectoree/cli ai status
```

### 2) Credentials

After `link`, `.vectoree/config.json` holds the project API key. Add `.vectoree/` to `.gitignore`. Never commit real keys.

For CI / headless (no browser). Prefer this over `login --use-device-code`:

```bash
export VECTOREE_API_URL=https://vectoree.ai
export VECTOREE_API_KEY=sk-ve-v1-your_key_here
npx @vectoree/cli ai status
```

```env
# .env (project root). Never commit real keys.
VECTOREE_API_URL=https://vectoree.ai
VECTOREE_API_KEY=sk-ve-v1-your_key_here
```

`VECTOREE_API_URL` is the **origin** (no trailing `/api`). Staging: `https://vectoree.net`.

---

## Install the CLI

```bash
npx @vectoree/cli --help
# or
npm install -g @vectoree/cli
```

Prefer `npx @vectoree/cli` so the agent uses a current version.

---

## Decision table

Match what the developer said. Fetch the matching **long playbook** (raw URL in the next section) and follow it. Fetch platform docs before inventing HTTP shapes (`docs get <docType>`).

### Connect

| ID | Developer says | Do this |
|----|----------------|---------|
| **S01** | "帮我连上 Vectoree" / `set up Vectoree` | `login` → `link` → `whoami` → `current` |
| **S02** | "这个目录绑到已有项目" | `link` (pick existing; do not create unless asked) |
| **S03** | "CI / 无浏览器怎么连" | Set `VECTOREE_API_KEY` (+ optional `VECTOREE_API_URL`). Verify with `ai status`. Do not default to `--use-device-code`. |
| **S04** | "我连上了吗 / 现在用的哪个项目" | `current` / `whoami` / `ai status` (read-only) |

### Model gateway

| ID | Developer says | Do this |
|----|----------------|---------|
| **S10** | "有哪些模型" | `ai models list` / `search` / `get`. Pick by modality and price. |
| **S11** | "用 DeepSeek / Claude / 某个模型" | `ai models search` → `ai chat` → `ai snippet` → paste into app code |
| **S12** | "先免费打一下" | `ai chat "ping"` (default `vectoree/free`) |
| **S13** | "帮我选便宜能用的" | `ai chat --model vectoree/auto`. Do not hardcode a vendor. |
| **S14** | "网关通不通 / 花了多少" | `ai status`. Usage lives in Dashboard → Organization → Billing until `ai usage` exists. |
| **S15** | "把这段改成走 Vectoree 网关" | `ai snippet --lang ts\|python`, then rewrite existing OpenAI SDK calls (`baseURL` + project key). |
| **S16** | "把我的 Codex 供应商切成 Vectoree" | See **C07**. One-click script, or surgically edit `~/.codex/config.toml`. |

### Database

| ID | Developer says | Do this |
|----|----------------|---------|
| **S20** | "现在有哪些表" | `db list` → `db schema <table>` before any write |
| **S21** | "建一张 todos / notes / orders" | `db create` with `--columns`, then `db insert` / `db sql` for sample rows |
| **S22** | "给 users 加一个 avatar 字段" | `db schema` → `db sql` (`ALTER TABLE …`). Confirm with the user first. |
| **S23** | "查一下最近 20 条" | `db query <table> --limit 20` |
| **S24** | "跑这段 SQL" | Show the SQL. Then `db sql` (prompts unless `--yes`) |

Drop / full-table delete: stop and confirm.

### Storage

| ID | Developer says | Do this |
|----|----------------|---------|
| **S30** | "有哪些桶" | `storage buckets list` |
| **S31** | "建一个公开的 uploads / avatars" | `storage buckets create <name> --public`. Say public vs private out loud. |
| **S32** | "把这张图传上去" | `storage upload <bucket> <path>` → `storage ls` |
| **S33** | "前端头像上传接到 Vectoree" | Create bucket, then follow `docs get storage-sdk`. Signed download URLs are P1; do not pretend the CLI has `storage url`. |

### Docs (usually step 0)

| ID | Developer says | Do this |
|----|----------------|---------|
| **S40** | "Vectoree 怎么接数据库 / Auth / AI" | `docs list` → `docs get <docType>`. Never invent APIs from memory. |

`docType` values: `instructions`, `auth-sdk`, `db-sdk`, `storage-sdk`, `ai-integration-sdk`. Also present but not launch-path: `functions-sdk`, `real-time`, `deployment`, `payments`. `docs search` is not available.

---

## Composite playbooks (what people actually ask)

Work toward connecting **their app**. Do not tour CLI modules.

| ID | Developer says | Chain | Long playbook |
|----|----------------|-------|---------------|
| **C01** | "初始化这个前端项目，连上 Vectoree" | S01 → S04 → write `.env` / `.gitignore` | `scenarios/connect.md` |
| **C02** | "做个能存数据的待办" | C01 → S21 → frontend CRUD via REST (`docs get db-sdk`) | `scenarios/connect.md` + `database.md` |
| **C03** | "做个 AI 聊天页" | C01 → S11/S12 → server route to `/api/v1/chat/completions` | `scenarios/connect.md` + `model-gateway.md` |
| **C04** | "待办 + 聊天" | C02 + C03 | C02 + C03 files |
| **C05** | "能上传图片的内容页" | C01 → S21 (`posts`) → S31/S33 | `scenarios/connect.md` + `database.md` + `storage.md` |
| **C06** | "把现有 OpenAI 调用迁到 Vectoree" | S04 → S15 | `scenarios/model-gateway.md` |
| **C07** | "把我的 Codex 供应商切成 Vectoree" | S04 (need a key) → S16 | `scenarios/model-gateway.md` |

### Example prompts (paste into the agent)

**C01**

```text
Initialize this frontend project on Vectoree.
Install with: npx skills add VectoreeAI/vectoree-skills
Then login, link, write .env, gitignore .vectoree/, then ai status.
```

**C03**

```text
Build a small AI chat page on Vectoree. Use vectoree/free first.
Probe with npx @vectoree/cli ai chat "ping", then paste ai snippet into a server route.
Do not put the API key in the browser bundle.
```

**C07**

```text
Point my Codex CLI at Vectoree as the model provider.
Follow C07 in the Vectoree skill (scenarios/model-gateway.md).
I already have (or you can create) a project API key.
```

---

## Long playbooks

These four files ship **next to this SKILL.md** (`scenarios/`). If they exist on disk, read them. Otherwise fetch the raw URL.

| File | IDs | Raw URL |
|------|-----|---------|
| `scenarios/connect.md` | S01–S04, C01 | https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/scenarios/connect.md |
| `scenarios/model-gateway.md` | S10–S16, C03, C06, C07 | https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/scenarios/model-gateway.md |
| `scenarios/database.md` | S20–S24, C02 | https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/scenarios/database.md |
| `scenarios/storage.md` | S30–S33, C05 | https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/scenarios/storage.md |

C04 is C02 + C03 (fetch database.md and model-gateway.md). C05 also needs connect.md (and database.md if the page stores posts).

---

## Command reference (launch slice only)

### Auth / link

```bash
npx @vectoree/cli login
npx @vectoree/cli login --use-device-code
npx @vectoree/cli logout
npx @vectoree/cli whoami
npx @vectoree/cli link
npx @vectoree/cli unlink
npx @vectoree/cli unlink --revoke
npx @vectoree/cli current
npx @vectoree/cli keys list
```

### Docs

```bash
npx @vectoree/cli docs list
npx @vectoree/cli docs get instructions
```

### Database

```bash
npx @vectoree/cli db list
npx @vectoree/cli db schema <table>
npx @vectoree/cli db create <table>
npx @vectoree/cli db create <table> --columns '[...]'
npx @vectoree/cli db query <table> --limit 20
npx @vectoree/cli db insert <table> --data '{"col":"value"}'
npx @vectoree/cli db sql "SELECT 1"
```

### Storage

```bash
npx @vectoree/cli storage buckets list
npx @vectoree/cli storage buckets create <name> [--public]
npx @vectoree/cli storage ls <bucket> [prefix]
npx @vectoree/cli storage upload <bucket> ./file.png
npx @vectoree/cli storage upload <bucket> ./file.png --key avatars/me.png
```

### AI / Model Gateway

```bash
npx @vectoree/cli ai models list
npx @vectoree/cli ai models search <query>
npx @vectoree/cli ai models get <model>
npx @vectoree/cli ai status
npx @vectoree/cli ai chat "<prompt>" [--model <id>]   # default: vectoree/free
npx @vectoree/cli ai snippet [--model <id>] [--lang ts|python]
```

Global flags: `--json`, `--yes`, `--api-url <url>`.

Safe-first order: `current` / `ai status` → `db list` / `storage buckets list` → writes. Confirm destructive SQL.

---

## Out of scope (do not promise, do not invent commands)

If asked, one honest sentence + point at the Dashboard when it still helps. Do not fabricate CLI.

| Topic | What to say |
|-------|-------------|
| Email / Google login, SMTP, list/create users | Tenant Auth is not in the launch CLI. Configure in Dashboard → Auth. |
| Custom login page (`--without-ui`) | Not shipped. |
| Stripe subscriptions for *their* users | Org wallet top-up ≠ tenant payments. Dashboard billing is the org wallet. |
| File-based migrations, RLS, indexes | Use `db sql` for a one-off; versioned migrations are later. |
| Storage download / signed URL | CLI has list/create/ls/upload only. |
| App templates / "金融模板" | No template catalog. |
| Deploy / live URL / custom domain | Not a self-serve PaaS this launch. |
| `@vectoree/sdk` as the app runtime | Later. Agent path is CLI + gateway HTTP + REST. |
| MCP | Not this launch. |
| Functions / Realtime / Compute | Hidden. |

**This is not Skills Hub.** Do not treat these as vectoree-skills scenes:

- "给这个 AI 应用装上搜索 / RAG"
- "从能力中心挑一个工具接到我的 Agent 产品里"
- any `vectoree skills install …`

Hub is a different product (runtime tools inside the developer's AI app). If asked: that line does not exist yet.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|--------|----------------|-----|
| Not logged in | No session | `npx @vectoree/cli login` |
| No API key | Not linked | `npx @vectoree/cli link` or set `VECTOREE_API_KEY` |
| Connection refused | Wrong `VECTOREE_API_URL` | Origin only: `https://vectoree.ai` (or `https://vectoree.net` for staging) |
| 401 / 403 | Invalid key or missing scope | Relink. CLI keys need `gateway:*`, `database:*`, `storage:*` |
| Wallet / billing errors on `ai chat` | Org wallet empty | Organization → Billing |
| `db create` rejects columns | Used reserved `id` / `created_at` / `updated_at` as the only fields | Add at least one custom column |
| Codex `BILLING_PRICE_NOT_CONFIGURED` | Bad or unpriced model slug | `ai models search`; try the `~…-latest` form |
| ChatGPT desktop history looks empty after C07 | API-provider mode | Expected. Restore with the setup script → `r` |

---

**One CLI. Cloud control from your AI agent.**

Repository: [VectoreeAI/vectoree-skills](https://github.com/VectoreeAI/vectoree-skills) · CLI: [@vectoree/cli](https://www.npmjs.com/package/@vectoree/cli)
