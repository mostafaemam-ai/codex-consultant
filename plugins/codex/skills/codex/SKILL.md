---
name: codex
description: Call the OpenAI Codex CLI (gpt-5.x) to get a second opinion, delegate a task, or hold a multi-turn conversation with it, then relay the result. Use whenever the user says "ask codex", "call codex", "run this by codex", "get codex's opinion", or wants a back-and-forth with codex.
---

# codex

Drive the [`codex` CLI](https://github.com/openai/codex) (OpenAI Codex) non-interactively and relay its answer back to the user. By default Codex acts as a **read-only consultant**: it reviews, critiques, and advises, but does not touch the repo.

## Prerequisites (check once, quickly)

- Confirm the binary is available: `command -v codex` (then `codex --version`).
- If `command -v codex` fails, tell the user Codex isn't installed or isn't on `PATH`, point them at the install steps (https://github.com/openai/codex), and stop.
- Codex must be authenticated (`codex login`, or an `OPENAI_API_KEY` in the environment). If a call fails with an auth error, relay that and stop.

Do **not** hard-code an absolute path to the binary — resolve it via `PATH` so the skill works on any machine.

## Default mode: CONSULTANT (read-only)

Treat Codex as a **consultant / advisor by default**: it reviews, critiques, and recommends — it must **not** edit files, run mutating commands, or create branches unless the user explicitly asks it to.

Enforce this two ways on every consultant call:
1. Run it read-only via config: `-c sandbox_mode="read-only"` (Codex can read the repo but not write).
2. Say so in the prompt: prepend `"You are a consultant. Do NOT edit any files or run mutating commands — only analyze and advise. "`

Only drop read-only mode when the user clearly says "let codex fix/change/implement it."

## How to call it

Always run non-interactively via `codex exec`. Never launch bare `codex` — that opens an interactive TUI that hangs the tool call.

**Consultant question (default — read-only):**
```bash
codex exec --skip-git-repo-check -c sandbox_mode="read-only" \
  "You are a consultant. Do NOT edit any files — only analyze and advise. <prompt>" < /dev/null
```

**Delegate an actual change (only when the user asks Codex to DO the work):**
```bash
codex exec --skip-git-repo-check "<prompt>" < /dev/null
```

- `< /dev/null` prevents it blocking on "Reading additional input from stdin...".
- `--skip-git-repo-check` avoids a repo-check prompt; safe everywhere.
- Give the full output plenty of time — set the Bash `timeout` to 180000+ ms for non-trivial prompts.

## Choosing the model (configurable)

Do **not** hard-code or assume a specific model — Codex uses whatever the user has configured, so the skill tracks their setup automatically. There are two levers:

1. **Per-user default (persistent):** the model comes from the user's `~/.codex/config.toml`, e.g.
   ```toml
   model = "gpt-5.5"
   model_reasoning_effort = "high"
   ```
   If unset, Codex falls back to its own built-in default. This is *not* necessarily "the latest" model — it's whatever the user pinned. To use a newer model everywhere, the user edits this file.
2. **Per-request override (one call):** pass `-m <model>` when the user asks for a specific model for a single question — e.g. `codex exec -m gpt-5.5 ...`. You can also override reasoning effort per call with `-c model_reasoning_effort="high"`.

Only add `-m`/`-c model=...` when the user names a model; otherwise let their config default apply. If the user asks "which model is codex using?", check `~/.codex/config.toml` (or note that Codex is on its built-in default) rather than guessing.

## Consultant workflow

1. Gather the context Codex needs (the diff, the file, the question) and pipe or paste it in — Codex sees the repo but give it a clear focus.
2. Ask a sharp, specific question ("What are the correctness risks in this resolver?", "Is this the right architecture for X?", "Review this diff for bugs").
3. Relay Codex's advice, attributed to it. Add your own take if you agree or disagree.
4. For a back-and-forth consultation, use `resume --last` so Codex keeps the thread.

**Continue the conversation (multi-turn):**
```bash
codex exec resume --last --skip-git-repo-check "<follow-up>" < /dev/null
```
- `--last` resumes the most recent session, preserving its memory of prior turns.
- To resume a specific session: `codex exec resume <session-id> ...` (the session id is printed in each run's header).

**Pipe file/context in:**
```bash
codex exec --skip-git-repo-check "Review this diff" < some.diff
```
(stdin is appended to the prompt as a `<stdin>` block.)

## Reading the output

Output includes a header block (workdir, model, session id, tokens) and then the reply, which is also echoed at the very end. Pipe through `tail` to grab the answer, e.g. `... 2>&1 | tail -40`. Note the `session id` if the user may want a follow-up.

## Relaying to the user

- Summarize or quote Codex's answer clearly, attributing it to Codex.
- If Codex made file changes (only possible when you dropped read-only mode), report exactly what it changed — don't assume.
- For a genuine "conversation," keep using `resume --last` for each subsequent turn so context carries over.

## Notes / gotchas

- In consultant (read-only) mode Codex cannot modify the workspace. If you deliberately drop read-only mode, Codex runs with `approval: never` and a `workspace-write` sandbox, so it can edit files in the workdir — only do this when the user asked Codex to make changes.
- Default reasoning effort is `high`.
- Token usage is printed per run — mention it if the user cares about cost.
