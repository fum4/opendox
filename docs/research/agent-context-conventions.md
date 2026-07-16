# Agent-Context Conventions — Research Report

> Historical research document (2026-07-16). Mechanisms for giving coding agents
> persistent repo knowledge, assessed for what opendox can learn/own.
> Sources: official docs, GitHub, empirical studies, critical posts — cited inline.

---

## 1. CLAUDE.md / AGENTS.md (the agents.md open standard)

**What/mechanism.** A plain-markdown file at repo root (nestable in monorepos; nearest file wins) that agents auto-load into context at session start — "a README for agents." No required fields, no schema; agents parse whatever prose is there. Stewarded since Dec 2025 by the Agentic AI Foundation (Linux Foundation), originating from OpenAI/Amp/Google/Cursor/Factory ([agents.md](https://agents.md/)). Claude Code reads `CLAUDE.md` (not AGENTS.md natively — the sanctioned bridge is `@AGENTS.md` import or a symlink; [Claude Code memory docs](https://code.claude.com/docs/en/memory), [gist on the sync problem](https://gist.github.com/yurukusa/d36197848911f025add142abefcde685)).

**Does well.**
- Won the standards war: 60k+ public repos, 25+ tools including Codex, Copilot coding agent, Cursor, Jules, Zed, Aider ([agents.md](https://agents.md/)); neutral governance since Dec 2025.
- Zero-infrastructure: one markdown file, version-controlled, reviewable in PRs like any code ([Windows Forum analysis](https://windowsforum.com/threads/agents-md-in-2026-turning-agent-prompts-into-reviewable-repo-policy.430224/)).
- Empirically, *some* content works: command-first executable instructions and explicit "done" criteria measurably improve completion; nesting handles monorepos ([Crosley, AGENTS.md Patterns](https://blakecrosley.com/blog/agents-md-patterns)).

**Fails.**
- **No schema, no verification, no provenance.** The spec is literally "any markdown." Nothing marks what's owner-stated vs LLM-generated, nothing checks that a named path/command still exists ([agents.md](https://agents.md/) — the spec contains zero staleness/verification language).
- **Drift is the documented #1 failure.** HN: architecture sections "wrong within a month"; one dev's file kept steering the agent to TRPC months after the project migrated to Convex — the agent trusted the file *over the code* ([HN thread](https://news.ycombinator.com/item?id=48629831)). Stale context files are actively worse than none, because they're injected authoritatively.
- **Bloat penalty is quantified.** Study across 2,500+ repos: files >150 lines give diminishing returns and +20–23% inference cost with no performance gain; prose paragraphs and vague directives ("be careful") produce no measurable behavior change ([Crosley](https://blakecrosley.com/blog/agents-md-patterns), [field guide](https://www.iuriio.com/blog/posts/2026/05/agents-md-field-guide-2026)). Most files in the wild are "too long, too vague, or LLM-generated."
- **"Just a README" critique.** HN consensus that most content in these files is recoverable — the agent could discover it from the codebase in seconds; research recommendation is to include *only* what can't be inferred ([HN](https://news.ycombinator.com/item?id=45791391), [study writeup](https://medium.com/activated-thinker/every-serious-developer-had-one-agents-md-file-now-theres-a-study-1f2513be062e)). This is precisely opendox's thesis, already validated by the community's pain.
- **Advisory, not enforced.** Anthropic's own docs: content is "context, not enforced configuration"; agents skip instructions, especially conflicting ones ([Claude Code docs](https://code.claude.com/docs/en/memory)).
- CLAUDE.md/AGENTS.md dual-file drift when a repo maintains both.

**Adoption signal.** Strongest in cluster: 60k+ repos, Linux Foundation governance, universal tool support. The *format* won; the *content quality problem* is unsolved — Claude Code even shipped a `/doctor` trim check that cuts derivable content (directory layouts, dep lists) and keeps "pitfalls, rationale, conventions" — Anthropic converging on opendox's exact content policy, but as a one-shot heuristic, not a spec.

**Lessons for opendox.**
1. Don't compete with AGENTS.md — *compose with it*. It's the won distribution channel; opendox docs should be reachable from it (import/link), while opendox supplies what the standard explicitly lacks: schema, provenance, verified-against-commit, lint.
2. The empirical line-count data (150-line cliff, 20%+ cost penalty for bloat) is your quantitative argument for the "no restatement" ban — cite it. And the TRPC/Convex anecdote is your argument for verification stamps: unverified authority actively misleads.

---

## 2. Cline Memory Bank (and memory-bank patterns generally)

**What/mechanism.** A prompt-pattern (not software): instruct the agent to maintain a `memory-bank/` of six hierarchical markdown files — `projectbrief.md`, `productContext.md`, `activeContext.md`, `systemPatterns.md`, `techContext.md`, `progress.md` — read at session start ("my memory resets completely; I rely ENTIRELY on my Memory Bank"), updated on trigger phrases like "update memory bank" ([Cline docs](https://docs.cline.bot/prompting/cline-memory-bank), [Cline blog](https://cline.bot/blog/memory-bank-how-to-make-cline-an-ai-agent-that-never-forgets)). Widely forked: [roo-code-memory-bank](https://github.com/GreatScottyMac/roo-code-memory-bank), [vanzan01/cursor-memory-bank](https://github.com/vanzan01/cursor-memory-bank), MCP-server variants.

**Does well.**
- First mainstream articulation that agents should *author and consume* their own persistent docs — direct ancestor of opendox's premise.
- The file taxonomy (stable brief → volatile activeContext) is a real insight: separating knowledge by *rate of change* so churn concentrates in one file.
- Tool-agnostic in principle (plain markdown), hence the fork ecosystem across Cline/Roo/Cursor.

**Fails.**
- **Drift by design.** Updates depend on the user remembering trigger phrases or the agent volunteering; docs admit no staleness detection exists — "trusts human judgment rather than automated verification" ([Cline docs](https://docs.cline.bot/prompting/cline-memory-bank)). "Forget to update them and Cline drifts" ([MemoryLake](https://www.memorylake.ai/en/blogs/cline-forgets-project-context), [Hopwood](https://danhopwood.com/posts/the-memory-bank-framework)).
- **No provenance at all** — agent-inferred summaries and owner statements are indistinguishable, and the agent *summarizing its own work* compounds errors (summarization drift; the generating agent can't catch its own mistakes — [dev.to validation post](https://dev.to/whoffagents/validation-is-the-bottleneck-why-your-claude-agent-keeps-drifting-14jn)).
- **Bloat/restatement built in**: `techContext.md` and `systemPatterns.md` are, in practice, restatements of what the code shows — exactly the recoverable content opendox bans. Files grow monotonically; the pattern includes no pruning discipline.
- **Fragmentation**: every fork has an incompatible file set; a standards request sat as an open issue ([roo-code-memory-bank #40](https://github.com/GreatScottyMac/roo-code-memory-bank/issues/40)) — nobody standardized it.

**Adoption signal.** Real but hobbyist-heavy: popular in Cline/Roo/Cursor communities 2024–2025, many forks and MCP wrappers; notably *no* major vendor adopted the six-file taxonomy — vendors went to auto-memory instead (see §5). Momentum has visibly shifted away from it.

**Lessons for opendox.**
1. Agent-authored persistence works as a workflow but rots without machine-checked freshness — the memory bank proves demand *and* proves that "please keep it updated" is not a mechanism. Opendox's verified-at-commit stamp + linter is the missing enforcement layer.
2. Steal the rate-of-change layering (stable why-knowledge vs volatile working state), but keep volatile session state *out of scope* — mixing "current task" with durable decisions is what made memory banks balloon.

---

## 3. Cursor rules (.cursor/rules, .cursorrules) and per-tool rule files

**What/mechanism.** `.cursor/rules/*.mdc` — markdown with YAML frontmatter (`alwaysApply`, `description`, `globs`) giving four activation modes: always / agent-decided / glob-triggered / manual @-mention; three precedence tiers (Team > Project > User); legacy single-file `.cursorrules` deprecated ([Cursor docs](https://cursor.com/docs/context/rules)). Same genus: `.github/copilot-instructions.md` + `.github/instructions/*.instructions.md` (Copilot), `.windsurfrules`, `.clinerules`, Claude Code's `.claude/rules/` with `paths:` frontmatter.

**Does well.**
- **Conditional loading is the real innovation**: glob-scoped rules load only when relevant files are touched — the only mainstream answer to context bloat, now copied by Claude Code (`.claude/rules/` `paths:`) and Copilot (`applyTo:`).
- Modularity (one topic per file, <500-line guidance) beats the monolith; frontmatter is a de facto minimal schema — proof that agents tolerate structured metadata in context files.

**Fails.**
- **Tool lock-in, terminally.** `.mdc` frontmatter is Cursor-proprietary; every vendor cloned the idea with an incompatible dialect, yielding 5–8 parallel rule files per polyglot repo. Whole tools exist just to fan out one source to all formats ([Ruler](https://github.com/intellectronica/ruler) — 20+ agent targets; Packmind similar). The ecosystem's dedicated "transpile my context files" tooling is an indictment.
- **Adherence decay**: community's chronic complaint — rules ignored after a few messages as recency wins in context optimization; users resort to "remember the rules" reinforcement ([Cursor forum](https://forum.cursor.com/t/best-cursor-rules-configuration/55979), [Elementor eng](https://medium.com/elementor-engineers/cursor-rules-best-practices-for-developers-16a438a4935c)).
- **Content is copy-pasted, not repo-derived**: [awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules) (very popular) is a library of generic per-framework boilerplate — "you're a Next.js expert…" — pure restatement of framework docs, zero repo-specific why-knowledge. The dominant usage pattern is exactly the anti-pattern.
- No verification, no provenance, no staleness stamps; rules also don't apply to all product surfaces (Cursor: agent chat only, not Tab/inline).

**Adoption signal.** High within Cursor's large user base; awesome-cursorrules is one of the most-starred repos in the space. But the *format* is losing to AGENTS.md for portable content (Cursor itself now reads AGENTS.md and recommends it "for straightforward cases").

**Lessons for opendox.**
1. Path/glob scoping is table stakes — opendox docs should declare what code region they govern (which also gives the linter its verification target: the glob must match existing paths).
2. Never bind the spec to one tool's dialect. Ship tool-neutral source of truth + (at most) generated adapters — be what Ruler bolts on, natively.

---

## 4. llms.txt

**What/mechanism.** Jeremy Howard / Answer.AI proposal (Sept 2024): `/llms.txt` at a website root — H1 + blockquote summary + H2-sectioned curated link lists to markdown versions of pages — to help LLMs at inference time; companion `.md` mirrors of HTML pages ([llmstxt.org](https://llmstxt.org/)).

**Does well.**
- Curated *index pointing to detail* rather than inlining everything — a good progressive-disclosure shape (Claude Code's MEMORY.md index + topic files, and Anthropic's own docs `llms.txt`, use the same shape).
- Trivially cheap to produce; docs platforms (Mintlify etc.) auto-generate it, so producer-side coverage snowballed.

**Fails.**
- **Consumers never showed up.** No major AI platform confirmed reading it; crawler-log audits: 408 requests out of 500M AI-bot visits in one 90-day study, 0.1% in another ([indexlab audit](https://www.indexlab.ai/blog/llms-txt-does-it-actually-work-october-2025-updated), [longato audit](https://www.longato.ch/llms-recommendation-2025-august/)).
- SE Ranking, ~300k domains: **no statistically significant correlation** with AI citation frequency ([SE Ranking](https://seranking.com/blog/llms-txt/)). Google on record: doesn't support it, won't; Mueller compared it to the keywords meta tag ([Medium summary](https://medium.com/@jsanthoshwriter/google-says-you-dont-need-llms-txt-so-why-is-everyone-talking-about-it-1756d5c98ffd)); Ahrefs: "no evidence any LLM uses it."
- No freshness/verification mechanism either — a stale llms.txt indexes pages that moved; nobody checks.

**Adoption signal.** Producer adoption high (docs-tooling default), consumer adoption ~nil. It survives only in the niche where an *agent is explicitly told* to fetch it (docs-navigation by coding agents) — i.e., where a deterministic consumer exists.

**Lessons for opendox.**
1. **A convention without a committed consumer is a cargo cult.** llms.txt optimized for hypothetical crawlers that never came. Opendox must ship its own consumer (the skill/CLI that loads, verifies, and lints) in v1 — the reader is part of the spec.
2. Producer-side auto-generation drove llms.txt's *file counts* while proving nothing about *value* — beware measuring opendox adoption by files-in-repos rather than verified, lint-passing files.

---

## 5. Newer 2025–2026 conventions

**a) Vendor auto-memory (Claude Code auto memory, Windsurf/Devin Cascade Memories).** Agent-authored, machine-local memory: Claude Code writes `MEMORY.md` (index, first 200 lines/25KB loaded) + on-demand topic files per repo ([docs](https://code.claude.com/docs/en/memory)); Cascade auto-generates memories from conversation ([Windsurf docs](https://docs.windsurf.com/windsurf/cascade/memories)). *Well:* zero user effort; index+topic-file progressive disclosure; plain markdown, auditable via `/memory`. *Fails:* machine-local — not in the repo, not shared with the team, dies with the laptop, invisible in code review; no provenance or verification; hidden auto-memories have actively corrupted projects ([ITStrategists postmortem](https://itstrategists.com/fixing-windsurf-auto-memory-how-hidden-memories-broke-my-projects-and-how-i-recovered/)); Devin carries nothing across sessions without it ([Hindsight](https://hindsight.vectorize.io/blog/2026/07/02/devin-desktop-persistent-memory)). *Adoption:* default-on in Claude Code — massive silent adoption. *Lesson:* vendors validated agent-authored memory but siloed it per-machine/per-vendor. Opendox's opening: the **repo-committed, team-shared, reviewable** version of the same knowledge — auto-memory content that passes provenance+lint should be *promotable* into opendox docs via PR.

**b) Kiro steering files (.kiro/steering/) + spec-driven development.** AWS Kiro: generated `product.md` / `tech.md` / `structure.md` plus custom steering docs with inclusion modes (always / file-match / manual) ([Kiro docs](https://kiro.dev/docs/steering/)); adjacent: GitHub Spec Kit et al. treat specs/requirements as first-class agent-readable artifacts. *Well:* named-doc taxonomy + inclusion modes; agent bootstraps the files from the codebase. *Fails:* the generated trio is largely code restatement (structure.md = directory layout — the most drift-prone content per the HN evidence); vendor directory; no verification. *Adoption:* moderate, AWS-backed. *Lesson:* auto-generating "foundation docs" from code produces exactly the recoverable content that goes stale — opendox's ban on restatement is the correct inversion of Kiro's default.

**c) Ruler / Packmind (rule fan-out).** Single `.ruler/` source compiled to 20+ per-tool formats ([Ruler](https://github.com/intellectronica/ruler)). Pure symptom-treatment of format fragmentation; confirms demand for one canonical source. *Lesson:* opendox as canonical source + a `sync`/emit step to AGENTS.md/rules dialects rides the standard rather than fighting it.

**d) MCP memory servers / agentmemory-class tools.** Database-backed cross-tool memory via MCP ([agentmemory](https://github.com/rohitg00/agentmemory), various memory-bank MCP servers). Opaque blobs, not reviewable/diffable, not in the repo — the opposite of docs-as-code. Signals the demand but not the answer for *repo* knowledge.

---

## Cross-cutting synthesis — what the cluster collectively fails at

Every mechanism above is a **write-only, trust-me channel**. Across all five:

1. **No verification, anywhere.** Not one convention — including the Linux-Foundation-blessed standard — can check that a claim in the file is still true of the code. The dominant documented failure (HN drift stories, memory-bank rot, Kiro's structure.md) is agents obeying stale files *over* live code. Opendox's deterministic linter (every referenced symbol/path/command must resolve) + verified-against-commit stamp has **zero incumbent competition**. This is the ownable gap.
2. **No provenance.** Owner intent, agent inference, and copy-pasted boilerplate are indistinguishable in every format. Nobody can answer "who says so, and how sure are we?" — which matters enormously now that most files are LLM-generated (and measurably worse for it).
3. **No content policy with teeth.** The ecosystem *knows* recoverable content is the poison (150-line data, "just a README" critique, Claude's `/doctor` trim) but expresses it only as blog-post advice or one-shot heuristics. A spec-level ban on restatement, mechanically distinguishable from why-knowledge, is unclaimed.
4. **Freshness is nobody's job.** No convention has a lifecycle: no stamp, no expiry, no CI gate. Docs-drift is treated as human discipline — which the memory-bank experience proves does not scale to agent-velocity codebases where the *agent* is the primary author.
5. **Fragmentation vs siloing dilemma.** Repo-committed formats fragment per tool (rules dialects); vendor memories unify per tool but silo per machine. Nothing today is simultaneously repo-committed, team-shared, tool-neutral, *and* verified. That four-way intersection is opendox's position.

Two cautions from the evidence: (a) llms.txt shows a spec without a shipped consumer is dead on arrival — the CLI/skill that reads and enforces opendox is not optional tooling, it *is* the product; (b) AGENTS.md shows the distribution battle is over — opendox should present as a disciplined layer reachable *through* AGENTS.md (an import/pointer), not a rival root file.
