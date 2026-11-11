# Agents merge opendox behind green checks

decided: tier=flag reversible=yes veto-by=2026-11-13

opendox had no factory.toml, so no prior merge tier. `[merge] by = "agents"` in cerebro/repo.toml records today's practice: agents open the PR and merge it. The approval paths (cerebro/**, workflows, infra/terraform/**, secrets/manifest, migrations) still need the owner's go whoever merges. The repo has no checks yet, so until code lands a merge is behind no gate at all.

This is the owner's call: veto it and the line becomes `by = "owner"`. Alternative dropped: `owner`, which would stop merges the repo already does today.

By: cerebro layout migration (agent), from the hub's brief
