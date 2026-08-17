---
name: vectoree
version: 0.2.0
description: Vectoree Cloud. Login, link a project, manage database/storage, call the Model Gateway, and point Codex at Vectoree via @vectoree/cli. Works with Cursor, Claude Code, Codex, and any agent that can run shell commands.
homepage: https://github.com/VectoreeAI/vectoree-skills
cli_package: "@vectoree/cli"
api_base_hint: Default API origin is https://vectoree.ai (override with VECTOREE_API_URL if needed)
last_updated: 2026-08-17
---

# Vectoree for AI Coding Agents

**One CLI (`vectoree`).** Skill = scene orchestration. CLI = atomic commands. Do not invent platform APIs; call `@vectoree/cli`, then change application code.

**Canonical skill URL (raw):** `https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skill.md`

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

### 1) Tell your AI agent

Paste into Cursor, Claude Code, Codex, or any coding assistant:

```text
Set up Vectoree as the backend for this project.
Follow the skill at:
https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skill.md

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

Match what the developer said. Run the listed commands. Fetch docs before inventing HTTP shapes (`docs get <docType>`).

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

| ID | Developer says | Chain | Done when |
|----|----------------|-------|-----------|
| **C01** | "初始化这个前端项目，连上 Vectoree" | S01 → S04 → write `.env` / `.gitignore` | `current` shows a project; key is not in git |
| **C02** | "做个能存数据的待办" | C01 → S21 → frontend CRUD via REST (`docs get db-sdk`) | `db query todos` shows sample rows; page can list/add |
| **C03** | "做个 AI 聊天页" | C01 → S11/S12 → server route to `/api/v1/chat/completions` | `ai chat` works; page can send a message |
| **C04** | "待办 + 聊天" | C02 + C03 | Table exists, gateway works, both UI pieces work |
| **C05** | "能上传图片的内容页" | C01 → S21 (`posts`) → S31/S33 | Bucket exists, `storage ls` shows a file, page can pick an image |
| **C06** | "把现有 OpenAI 调用迁到 Vectoree" | S04 → S15 | `baseURL` is the project gateway; model ids come from `models search` |
| **C07** | "把我的 Codex 供应商切成 Vectoree" | S04 (need a key) → S16 | Codex `model_provider = "vectoree"`; banner shows the chosen slug |

### Example prompts (paste into the agent)

**C01**

```text
Initialize this frontend project on Vectoree. Follow
https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skill.md
Run login, link, write .env, gitignore .vectoree/, then ai status.
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
Use the skill playbook C07. I already have (or you can create) a project API key.
```

---

## Playbook details

### C01: Initialize and connect

```bash
npx @vectoree/cli login
npx @vectoree/cli link
npx @vectoree/cli whoami
npx @vectoree/cli current
```

`link` interactively picks or creates a project and writes a scoped key into `.vectoree/`.

Then:

1. Copy the key into `.env` as `VECTOREE_API_KEY` (and `VECTOREE_API_URL=https://vectoree.ai`).
2. Ensure `.gitignore` contains `.vectoree/` and `.env`.
3. Do not print the full key back to the user.

### C03: AI chat page

```bash
npx @vectoree/cli ai models search deepseek
npx @vectoree/cli ai chat "ping"                          # default: vectoree/free
npx @vectoree/cli ai chat "ping" --model vectoree/auto    # cost-aware router
npx @vectoree/cli ai snippet --model vectoree/free --lang ts
```

Product models:

| Model | Kind | Behavior |
|-------|------|----------|
| `vectoree/free` | alias | Free-tier default. Use this for a first ping. |
| `vectoree/auto` | router | Picks a cheap/available catalog model per request. |

App code uses the OpenAI SDK against the **Vectoree** origin, not a third-party dashboard key:

```ts
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: `${process.env.VECTOREE_API_URL}/api/v1`,
  apiKey: process.env.VECTOREE_API_KEY,
});
```

Keep the key on the server (API route or server action). The browser calls your server; your server calls Vectoree.

### S21: Create a table

`id`, `created_at`, and `updated_at` are reserved and auto-added. `--columns` **replaces** the CLI default, so include a primary key plus at least one custom column:

```bash
npx @vectoree/cli db list
npx @vectoree/cli db create todos --columns '[
  {"columnName":"id","type":"uuid","isPrimaryKey":true,"isNullable":false,"isUnique":true,"defaultValue":"gen_random_uuid()"},
  {"columnName":"title","type":"string","isNullable":false,"isUnique":false},
  {"columnName":"done","type":"boolean","isNullable":false,"isUnique":false}
]'
npx @vectoree/cli db insert todos --data '{"title":"First task","done":false}'
npx @vectoree/cli db query todos --limit 20
```

Column types: `string` | `integer` | `float` | `boolean` | `uuid` | `date` | `datetime` | `json`.

App CRUD: `docs get db-sdk`. Launch path is REST `/api/database/records/{table}` with `Authorization: Bearer $VECTOREE_API_KEY`, from a **server** route. Do not ship `@vectoree/sdk` as if it were a launch deliverable.

### S22: Add a column

```bash
npx @vectoree/cli db schema users
npx @vectoree/cli db sql "ALTER TABLE users ADD COLUMN avatar text"
```

Show the SQL first. Use `--yes` only after the user agrees.

### S31 / C05: Bucket + upload

```bash
npx @vectoree/cli storage buckets list
npx @vectoree/cli storage buckets create uploads --public
npx @vectoree/cli storage upload uploads ./photo.png
npx @vectoree/cli storage ls uploads
```

Public buckets are world-readable. Private is the default without `--public`. Frontend upload: `docs get storage-sdk`. CLI cannot yet mint signed URLs (`storage url` is P1).

### C06: Migrate existing OpenAI calls

1. `npx @vectoree/cli current` and `ai status`.
2. Search the repo for `openai`, `baseURL`, `OPENAI_API_KEY`, `api.openai.com`.
3. `npx @vectoree/cli ai models search <vendor or name>` and pick a real catalog id (or `vectoree/auto`).
4. `npx @vectoree/cli ai snippet --lang ts` (or `python`) and apply:
   - `baseURL` → `{VECTOREE_API_URL}/api/v1`
   - `apiKey` → `VECTOREE_API_KEY`
   - `model` → the id from search, not a guessed OpenAI-only name
5. Probe with `ai chat "ping" --model <id>`.

### C07: Point Codex at Vectoree

This switches **Codex CLI** (and ChatGPT desktop, which shares `~/.codex`) to the Vectoree Model Gateway. It is not the same as C01 (app backend). You still need a project API key (`sk-ve-v1-…` or employee `ek-ve-v1-…`). Do not use the instance master key (`ik_`).

**Human, interactive (preferred):**

```bash
# macOS / Linux
bash <(curl -fsSL https://vectoree.ai/scripts/codex-vectoree-setup.sh)
```

```powershell
# Windows
irm https://vectoree.ai/scripts/codex-vectoree-setup.ps1 | iex
```

Menu: `1`–`5` curated slugs, `c` custom slug, `r` restore. Type `y` on first install. Optional env: `VECTOREE_API_KEY`, `VECTOREE_BASE_URL` (default `https://vectoree.ai/api/v1/`), `CODEX_HOME`.

**Agent, non-interactive:** do not replace the whole file (MCP servers and other keys must stay). Backup `~/.codex/config.toml` first, then set:

```toml
model = "~deepseek/deepseek-v4-flash-latest"
model_provider = "vectoree"
model_reasoning_effort = "high"

[model_providers.vectoree]
name = "Vectoree"
base_url = "https://vectoree.ai/api/v1/"
wire_api = "responses"
experimental_bearer_token = "sk-ve-v1-..."
```

`wire_api = "responses"` matches `POST /api/v1/responses`. Put the token from `.vectoree/config.json` or the user; never echo it.

Verify: Codex CLI startup banner shows `model: <slug>`. ChatGPT desktop model picker shows **Custom** (expected in API-provider mode; account history UI can look empty; local `~/.codex/sessions` are not deleted).

Restore: re-run the script, choose `r`, fully quit and reopen Codex / ChatGPT.

If Codex returns `BILLING_PRICE_NOT_CONFIGURED`, the slug is wrong or unpriced. `ai models search` / Dashboard Admin → Models. `*-latest` aliases often need the `~` prefix.

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
