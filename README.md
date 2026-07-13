# codex-consultant

A [Claude Code](https://claude.com/claude-code) plugin that lets Claude drive the [OpenAI Codex CLI](https://github.com/openai/codex) as a **read-only consultant** — get a second opinion, review a diff, sanity-check an architecture, or hold a multi-turn back-and-forth with Codex — and relay the result back to you.

By default Codex runs **read-only**: it reviews, critiques, and advises, but never edits files or runs mutating commands. You can explicitly let it make changes when you want to.

## What you get

Once installed, just talk to Claude naturally:

- *"Ask codex to review this diff for bugs."*
- *"Get codex's opinion on this architecture."*
- *"Run this by codex and tell me what it thinks."*
- *"Ask codex a follow-up: what about the error handling?"* (keeps the conversation going)

Claude will call the Codex CLI under the hood, then summarize and attribute Codex's advice.

## Prerequisites

You need the OpenAI Codex CLI installed and authenticated on the same machine as Claude Code:

1. **Install Codex** — follow the instructions at https://github.com/openai/codex.
2. **Authenticate** — run `codex login`, or set `OPENAI_API_KEY` in your environment.
3. **Verify** — `codex --version` should print a version.

Claude resolves the `codex` binary from your `PATH`, so no path configuration is needed.

## Install

In Claude Code, add this repo as a plugin marketplace and install the plugin:

```
/plugin marketplace add mostafaemam-ai/codex-consultant
/plugin install codex@codex-consultant
```

That's it. The `codex` skill is now available to Claude.

> **Tip:** To pin to a specific version or branch, use `/plugin marketplace add mostafaemam-ai/codex-consultant@<branch-or-tag>`.

### Updating

```
/plugin marketplace update codex-consultant
```

### Uninstalling

```
/plugin uninstall codex@codex-consultant
/plugin marketplace remove codex-consultant
```

## How it works

The plugin ships a single Skill ([`plugins/codex/skills/codex/SKILL.md`](plugins/codex/skills/codex/SKILL.md)) that teaches Claude how to:

- Run Codex non-interactively via `codex exec` (never the interactive TUI, which would hang).
- Default to a **read-only sandbox** (`-c sandbox_mode="read-only"`) so Codex can read the repo but not change it.
- Resume a session (`codex exec resume --last`) for multi-turn conversations.
- Read Codex's output and relay it back, attributed to Codex.

Claude only drops read-only mode when you explicitly ask Codex to make changes (e.g. *"let codex fix it"*).

## Repository layout

```
codex-consultant/
├── .claude-plugin/
│   └── marketplace.json        # marketplace manifest (this repo is the marketplace)
└── plugins/
    └── codex/
        ├── .claude-plugin/
        │   └── plugin.json     # plugin manifest
        └── skills/
            └── codex/
                └── SKILL.md    # the skill instructions
```

## License

[MIT](LICENSE) © Mostafa Emam
