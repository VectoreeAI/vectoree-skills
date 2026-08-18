---
name: vectoree
version: 0.7.0
description: Use when connecting an app to Vectoree Cloud, adding login for users of that app (not Vectoree Cloud login), listing or calling models through the Vectoree gateway (chat, TTS, STT, image, video, embeddings), adding or switching web search to Vectoree Tool Hub MCP, managing project database or storage via @vectoree/cli, migrating OpenAI SDK calls, or pointing Codex at Vectoree. Install with npx skills add VectoreeAI/vectoree-skills.
homepage: https://github.com/VectoreeAI/vectoree-skills
cli_package: "@vectoree/cli"
api_base_hint: Default API origin is https://vectoree.ai (override with VECTOREE_API_URL if needed)
last_updated: 2026-08-18
---

# Vectoree for AI Coding Agents

**One CLI (`vectoree`).** Skill = scene orchestration. CLI = atomic commands. Do not invent platform APIs; call `@vectoree/cli`, then change application code.

**Install:** `npx skills add VectoreeAI/vectoree-skills`

**Canonical files:** [`skills/vectoree/SKILL.md`](https://github.com/VectoreeAI/vectoree-skills/blob/main/skills/vectoree/SKILL.md) (what `npx skills add` copies). Raw fallback: `https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/SKILL.md`

This skill is for **developers and their coding agents** wiring Vectoree Cloud into an app, into Codex, or into the agent's MCP search. It is not ClawHub / Skills Hub, and not a catalog the end user clicks in the dashboard.

Three surfaces. Do not mix them:

| Surface | What | Who |
|---------|------|-----|
| **CLI** | `npx @vectoree/cli …` | Agent ops: login, link, db, storage, probe models and tools |
| **OpenAI-compatible HTTP** | `POST /api/v1/chat/completions` (text). TTS/STT/image/video/embeddings use other `/api/v1/*` paths — see S11b. | App runtime inference |
| **Hosted MCP** | `POST /mcp` (`search`, `extract`) | Agent web search / page extract |

```text
Developer → coding agent → this skill → @vectoree/cli + MCP / app code → Vectoree Cloud
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
| **S01** | "帮我连上 Vectoree" / `set up Vectoree` / "登录一下 Vectoree" / CLI 登录 | `login` → `link` → `whoami` → `current` |
| **S02** | "这个目录绑到已有项目" | `link` (pick existing; do not create unless asked) |
| **S03** | "CI / 无浏览器怎么连" | Set `VECTOREE_API_KEY` (+ optional `VECTOREE_API_URL`). Verify with `ai status`. Do not default to `--use-device-code`. |
| **S04** | "我连上了吗 / 现在用的哪个项目" | `current` / `whoami` / `ai status` (read-only) |

### Model gateway

| ID | Developer says | Do this |
|----|----------------|---------|
| **S10** | "有哪些模型" / TTS / STT / 视频 / 生图 / embedding | `ai models list` / `search` / `get` with `--input-modality` / `--output-modality`. Then probe the **matching** command (do not `ai chat` a TTS slug). |
| **S11** | "用 DeepSeek / Claude / 某个文本模型" | `ai models search` → `ai chat` → `ai snippet` → paste into app code |
| **S11b** | "用 TTS / STT / 生图 / 视频 / embedding" | Filter catalog → `ai speech` / `transcribe` / `image` / `video` / `embed` → `ai snippet --model <id>`. Runtime paths: `/audio/speech`, `/audio/transcriptions`, `/images`, `/videos`, `/embeddings`. |
| **S12** | "先免费打一下" | `ai chat "ping"` (default `vectoree/free`) |
| **S13** | "帮我选便宜能用的" | `ai chat --model vectoree/auto`. Do not hardcode a vendor. |
| **S14** | "网关通不通 / 花了多少" | `ai status`. Usage lives in Dashboard → Organization → Billing until `ai usage` exists. |
| **S15** | "把这段改成走 Vectoree 网关" | `ai snippet --lang ts\|python`, then rewrite existing OpenAI SDK calls (`baseURL` + project key). |
| **S16** | "把我的 Codex 供应商切成 Vectoree" | See **C07**. One-click script, or surgically edit `~/.codex/config.toml`. |

### Tool Hub (search)

| ID | Developer says | Do this |
|----|----------------|---------|
| **S17** | "帮我加一下搜索的能力" / "搜索的能力切换到 vectoree" / replace Tavily or Brave | See **C08**. `tools snippet --write` (add `--replace-search` when switching). |
| **S18** | "搜索通不通 / 先打一下" | `tools search "<query>"`. Optional `tools extract <url>`. These bill the wallet. |
| **S19** | "MCP 地址 / 有哪些工具" | `tools status`. Tools are `search` and `extract` only. |

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
| **S41** | "给我的应用加登录" / email+password / Google 登录到我的 App | **C09**. If they said "登录 Vectoree", that is **S01**. |

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
| **C08** | "帮我加一下搜索" / "搜索切到 vectoree" | S04 (need `tools:*`) → S17 | `scenarios/tool-hub.md` |
| **C09** | "给我的应用加登录" | S04 → `auth status` / `snippet` / `open` | `scenarios/auth.md` |

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

**C08**

```text
Add web search via Vectoree Tool Hub (or switch my existing Tavily/Brave search MCP to Vectoree).
Follow C08 in the Vectoree skill (scenarios/tool-hub.md).
Use npx @vectoree/cli tools snippet --write --replace-search.
```

**C09**

```text
Add login to my app (email + 8-digit code). Follow C09 in the Vectoree skill (scenarios/auth.md).
This is not Vectoree Cloud login. Do not CREATE TABLE users.
```

---

## Long playbooks

These five files ship **next to this SKILL.md** (`scenarios/`). If they exist on disk, read them. Otherwise fetch the raw URL.

| File | IDs | Raw URL |
|------|-----|---------|
| `scenarios/connect.md` | S01–S04, C01 | https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/scenarios/connect.md |
| `scenarios/model-gateway.md` | S10–S16, C03, C06, C07 | https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/scenarios/model-gateway.md |
| `scenarios/tool-hub.md` | S17–S19, C08 | https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/scenarios/tool-hub.md |
| `scenarios/auth.md` | S41, C09 | https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/scenarios/auth.md |
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
npx @vectoree/cli auth status
npx @vectoree/cli auth snippet --ui
npx @vectoree/cli auth snippet --no-ui --lang ts
npx @vectoree/cli auth open
npx @vectoree/cli auth open --docs
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
npx @vectoree/cli ai models list [--input-modality <m>] [--output-modality <m>]
npx @vectoree/cli ai models search <query> [--input-modality <m>] [--output-modality <m>]
npx @vectoree/cli ai models get <model>
npx @vectoree/cli ai status
npx @vectoree/cli ai chat "<prompt>" [--model <id>]   # text chat only; default vectoree/free
npx @vectoree/cli ai speech "<text>" --model <tts-id> [--voice <id>] [--out speech.mp3]
npx @vectoree/cli ai transcribe --model <stt-id> [--file clip.wav]
npx @vectoree/cli ai image "<prompt>" --model <image-id>
npx @vectoree/cli ai video "<prompt>" --model <video-id>
npx @vectoree/cli ai embed "<text>" --model <embedding-id>
npx @vectoree/cli ai snippet [--model <id>] [--lang ts|python]   # snippet matches the model's modality

# modality examples (comma-separated values allowed)
npx @vectoree/cli ai models list --output-modality speech          # TTS
npx @vectoree/cli ai models list --input-modality audio --output-modality transcription  # STT
npx @vectoree/cli ai models list --output-modality video
npx @vectoree/cli ai models search veo --output-modality video
```

Need `@vectoree/cli` ≥ 0.1.10 for `auth status` / `snippet` / `open` (prefer `npx`). ≥ 0.1.9 for `ai speech` / `transcribe` / `image` / `video` / `embed` and `tools`. `ai chat` on a TTS slug is refused on purpose.

### Tool Hub

```bash
npx @vectoree/cli tools status
npx @vectoree/cli tools snippet
npx @vectoree/cli tools snippet --write --replace-search
npx @vectoree/cli tools snippet --write --client claude --replace-search
npx @vectoree/cli tools search "<query>"
npx @vectoree/cli tools extract <url>
```

`link` keys include `tools:*`. Older keys: relink.

Global flags: `--json`, `--yes`, `--api-url <url>`.

Safe-first order: `current` / `ai status` / `tools status` → `db list` / `storage buckets list` → writes. Confirm destructive SQL.

---

## Out of scope (do not promise, do not invent commands)

If asked, one honest sentence + point at the Dashboard when it still helps. Do not fabricate CLI.

| Topic | What to say |
|-------|-------------|
| Email / Google login for *your app's* users | **C09**. `auth status` / `snippet` / `open`. Identity is `auth.users`. Do not `CREATE TABLE users`. |
| "登录一下 Vectoree" / CLI login | **S01** (`vectoree login`). Not C09. |
| Copy Dashboard login / `POST /api/auth/signup` from the app | Forbidden. Those are Cloud console. |
| Hosted Auth UI / passwordless OTP login | Not shipped. Use `auth snippet --ui` or `--no-ui`. |
| Stripe subscriptions for *their* users | Org wallet top-up ≠ tenant payments. Dashboard billing is the org wallet. |
| File-based migrations, RLS, indexes | Use `db sql` for a one-off; versioned migrations are later. |
| Storage download / signed URL | CLI has list/create/ls/upload only. |
| App templates / "金融模板" | No template catalog. |
| Deploy / live URL / custom domain | Not a self-serve PaaS this launch. |
| `@vectoree/sdk` as the app runtime | Later. Agent path is CLI + gateway HTTP + REST + MCP. |
| RAG / vector store as a product | Search MCP is not RAG. Do not invent `skills install` or a tool marketplace beyond `search` / `extract`. |
| Functions / Realtime / Compute | Hidden. |
| `vectoree skills install …` | Does not exist. Tool Hub is hosted MCP, not ClawHub. |

**This is not ClawHub.** "帮我加搜索 / 把搜索切到 vectoree" is **C08** (`tools snippet --write`). Do not install random skill packs or invent extra tool names.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|--------|----------------|-----|
| Not logged in | No session | `npx @vectoree/cli login` |
| No API key | Not linked | `npx @vectoree/cli link` or set `VECTOREE_API_KEY` |
| Connection refused | Wrong `VECTOREE_API_URL` | Origin only: `https://vectoree.ai` (or `https://vectoree.net` for staging) |
| 401 / 403 | Invalid key or missing scope | Relink. CLI keys need `gateway:*`, `tools:*`, `database:*`, `storage:*` |
| `tools` / `auth` / `ai speech` command missing | Old CLI | `npx @vectoree/cli@0.1.10 --help` (need ≥ 0.1.10 for `auth`). Do not use a stale global install. |
| TTS `GATEWAY_NO_AVAILABLE_CHANNEL` on `ai chat` | Used chat for a speech model | `ai speech "hello" --model <id>`. Runtime is `POST /api/v1/audio/speech`. |
| Wallet / billing errors on `ai chat` or `tools search` | Org wallet empty | Organization → Billing |
| `db create` rejects columns | Used reserved `id` / `created_at` / `updated_at` as the only fields | Add at least one custom column |
| Codex `BILLING_PRICE_NOT_CONFIGURED` | Bad or unpriced model slug | `ai models search`; try the `~…-latest` form |
| ChatGPT desktop history looks empty after C07 | API-provider mode | Expected. Restore with the setup script → `r` |

---

**One CLI. Cloud control from your AI agent.**

Repository: [VectoreeAI/vectoree-skills](https://github.com/VectoreeAI/vectoree-skills) · CLI: [@vectoree/cli](https://www.npmjs.com/package/@vectoree/cli)
