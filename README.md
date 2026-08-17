# Vectoree Skills

Agent skill for [Vectoree Cloud](https://vectoree.ai). Install it into Cursor, Claude Code, Codex, and other agents, then run [`@vectoree/cli`](https://www.npmjs.com/package/@vectoree/cli).

This is **not** Skills Hub (runtime tools inside an AI app). Hub is a different product.

## Install

```bash
npx skills add VectoreeAI/vectoree-skills
```

That is the [Agent Skills](https://skills.sh) CLI. It finds `skills/vectoree/SKILL.md` and copies it into the agent skill directories.

Then:

```bash
npx @vectoree/cli login
npx @vectoree/cli link
npx @vectoree/cli ai status
```

Global install: `npx skills add VectoreeAI/vectoree-skills -g`.

## Fallback (no skills CLI)

Paste into the agent:

```text
Set up Vectoree as the backend for this project.
Follow the skill at:
https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skills/vectoree/SKILL.md

1. npx @vectoree/cli login
2. npx @vectoree/cli link
3. npx @vectoree/cli ai status
```

## Layout

```text
skills/vectoree/SKILL.md          ← what `npx skills add` installs
skills/vectoree/scenarios/
  connect.md
  model-gateway.md
  database.md
  storage.md
```

Keep `SKILL.md` only under `skills/vectoree/`. A root `skill.md` breaks install on macOS (case-insensitive: `skill.md` ≡ `SKILL.md`, so the CLI copies the whole repo).

## Point Codex at Vectoree

Playbook **C07** in `skills/vectoree/scenarios/model-gateway.md`:

```bash
bash <(curl -fsSL https://vectoree.ai/scripts/codex-vectoree-setup.sh)
```

Windows: `irm https://vectoree.ai/scripts/codex-vectoree-setup.ps1 | iex`

## Source of truth

Edit `docs/agent-native/vectoree-skill.md` and `docs/agent-native/scenarios/` in the platform monorepo, then copy them here. Do not make the public copy the only copy.

## License

Apache License 2.0. See [LICENSE](./LICENSE).
