# Vectoree Skills

Agent skill for [Vectoree Cloud](https://vectoree.ai). Developers paste one raw URL into Cursor, Claude Code, Codex, or any coding assistant. The agent then runs [`@vectoree/cli`](https://www.npmjs.com/package/@vectoree/cli) to connect a project, call the Model Gateway, and manage database / storage.

This is **not** Skills Hub (runtime tools inside an AI app). Hub is a different product.

## Use it

Paste this into your coding agent:

```text
Set up Vectoree as the backend for this project.
Follow the skill at:
https://raw.githubusercontent.com/VectoreeAI/vectoree-skills/main/skill.md

1. npx @vectoree/cli login
2. npx @vectoree/cli link
3. npx @vectoree/cli ai status
```

Canonical file: [`skill.md`](./skill.md) (raw URL above).

Long playbooks (also fetchable as raw URLs):

- [`scenarios/connect.md`](./scenarios/connect.md)
- [`scenarios/model-gateway.md`](./scenarios/model-gateway.md)
- [`scenarios/database.md`](./scenarios/database.md)
- [`scenarios/storage.md`](./scenarios/storage.md)

## Point Codex at Vectoree

If you want Codex CLI / ChatGPT desktop to use the Vectoree Model Gateway (playbook **C07** in `scenarios/model-gateway.md`):

```bash
bash <(curl -fsSL https://vectoree.ai/scripts/codex-vectoree-setup.sh)
```

Windows: `irm https://vectoree.ai/scripts/codex-vectoree-setup.ps1 | iex`

## Source of truth

Edit `docs/agent-native/vectoree-skill.md` and `docs/agent-native/scenarios/` in the platform monorepo, then copy them here. Do not make the public copy the only copy.

## License

Apache License 2.0. See [LICENSE](./LICENSE).
