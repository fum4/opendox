# opendox: repo rules

This file adds only what is specific to opendox. The cross-repo rules are the owner's global agent rules (devbox `agents/AGENTS.md`).

## What this is
A spec, CLI and skill for agent-written repo docs: capture the why, never restate the code (README). Today it holds only the research that shaped the design; public repo.

## Asks and decisions
About to ask the owner anything, just chose between options, or just heard a decision → the `cerebro-ledger` skill: it's a file in the repo (`ledger/asks/`, `docs/decisions/log/`), not a chat message, even if the skill didn't fire.

## Routing
| Occasion | Doc | Guardrail |
|---|---|---|
| Why opendox is shaped this way | [docs/research/synthesis.md](docs/research/synthesis.md) | Historical: once the spec exists, it supersedes the synthesis wherever they differ |
| The landscape it was compared against | [docs/research/README.md](docs/research/README.md) | Point-in-time research, not maintained: don't update it to match the present |
| Adding the first code | [.mise.toml](.mise.toml) | Replace the skipped pre-push / pre-merge markers with real tasks in the same change |
