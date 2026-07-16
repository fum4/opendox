# Research Synthesis — the design bets opendox makes

> Historical research document (2026-07-16). Conclusions drawn from the three cluster
> reports in this directory, plus a hands-on audit of a mature agent-maintained docs
> substrate (a private production repo, ~150 docs / 41 ADRs; audit not published here).
> These conclusions seeded the spec; the spec supersedes this document wherever they differ.

## The position

**Nothing today is simultaneously repo-committed, team-shared, tool-neutral, and
verified.** Repo-committed formats (AGENTS.md, cursor rules, memory banks) have no
verification or provenance; vendor auto-memory is verified by nobody and siloed
per-machine; doc-sync products either died (CodeSee), pivoted (Swimm), or check freshness
probabilistically at LLM prices. That four-way intersection is opendox.

One sentence, Diátaxis-style teachable:

> **Document only what an ideal agent cannot recover from the repo — and prove the rest
> still resolves.**

## The ten bets, each with its evidence

1. **Only non-recoverable "why" gets written; restatement is banned at spec level.**
   Evidence: 2,500-repo study — context files >150 lines cost +20–23% inference with no
   gain; CodeSee died selling recoverable knowledge; DeepWiki's audited quality is "one
   factual error per section" of confident paraphrase; the private-repo audit found the
   only drifted/contradictory docs were precisely the restated-structure ones.

2. **The deterministic linter is the product, not an accessory.** Every referenced
   path/symbol/command/link in a doc must resolve against the repo; failures gate CI.
   Evidence: in ~20 years of doc tooling, no one ships this (OpenSpec validates structure
   only; Spec Kit/Mintlify/plugins judge freshness by LLM at $10–40/day; Swimm built it,
   proved it works, and buried it in a services pivot). The unoccupied lane.

3. **A spec without a shipped consumer is a cargo cult — ship the skill + CLI in v1.**
   Evidence: llms.txt — massive producer adoption, ~0.1% of AI-crawler traffic, zero
   measured value. The reader is part of the spec.

4. **Provenance is first-class: `owner-stated` vs `agent-inferred`, plus evidence links.**
   An agent may never mark rationale `owner-stated` unless it appeared in the owner's own
   words. Evidence: DeepWiki's fatal flaw (inference indistinguishable from fact); MADR
   has `decision-makers` but no stated/inferred split; the field is circling this and
   nobody shipped it.

5. **Verified-at-commit stamps make freshness mechanical, not social.** When an agent
   confirms a doc still matches the code, that gets recorded (`verified: <date> @<commit>`);
   staleness becomes computable distance, and re-verification work accumulates instead of
   evaporating. Evidence: every "keep it updated" convention rots (memory banks, KEP stage
   metadata, arc42 quarterly rituals); only mechanical gates hold (Vale, typecheck history).

6. **Lifecycle = PEP's historical/canonical split + OpenSpec's deterministic fold.**
   Anything task-shaped (plan, spec, proposal) becomes an immutable historical document at
   ship time, with a mandatory forward pointer to the living doc that absorbed its truth;
   a linter rule enforces the banner. Evidence: the lifecycle table across ADR tools /
   Spec Kit / Kiro / RFCs — Python PEPs are the only process that honestly solved
   post-ship rot; OpenSpec's fold-and-archive is the only deterministic mechanism (its
   silent-drop merge bug is the cautionary detail: fail loud on collision).

7. **Format outlives tool: the durable core is a semver'd spec; CLI and skill are
   replaceable satellites.** Zero hosted components, zero framework-heavy stacks, plain
   markdown + YAML frontmatter, MADR-compatible where it overlaps. Evidence: adr-tools,
   log4brains, adr-manager all died; MADR and Diátaxis survived precisely because they are
   specs/ideas with zero runtime.

8. **Lint semantics, not directory shape.** Mandating folders gets cargo-culted
   (Diátaxis's four-folders failure, Pigweed's fragmentation complaint); the linter checks
   what an entry *is* (scoped? provenanced? resolvable? non-restating?), not where it
   lives. Scope globs double as lint targets (the glob must match existing paths).

9. **Two-tier checking: deterministic gate free on every push; agentic review only on
   escalation.** Evidence: Swimm's proven split (reference-resolution reliable,
   semantic-drift needs escalation); LLM-per-merge freshness costs $10–40/day and is
   unauditable.

10. **Compose with the winners, never compete:** reachable through AGENTS.md (60k+ repos,
    Linux Foundation — the distribution war is over), emit-adapters for per-tool rule
    dialects (be natively what Ruler bolts on), Vale-complementary ("Vale checks form,
    opendox checks truth"), volatile session state explicitly out of scope (the
    memory-bank bloat lesson).

## Anti-goals (each one killed a predecessor)

- No knowledge graph / graph database (premature at repo scale; grep + flat files win).
- No 14-type ontology (template slots offered to an LLM get filled with plausible filler
  — arc42, BMAD, Kiro's 16-acceptance-criteria-for-a-one-liner).
- No proprietary format or editor (Swimm), no hosted component (adr-manager), no
  framework-heavy stack (log4brains).
- No semantic-drift *guessing* in the deterministic tier (Swimm's boundary).
- No per-language deep analysis treadmill (CodeSee); reference resolution stays shallow:
  paths, symbols-by-search, commands, links.
- No generated descriptive docs (DeepWiki/Kiro steering trio — the auto-generation trap).

## Open questions the spec must settle

1. Recoverable facts in docs: ban-and-link vs generated blocks (leaning ban-and-link v1).
2. Exact frontmatter schema: minimal field set, MADR-compat mapping.
3. Symbol reference syntax that survives renames tolerably (path-anchored search, not
   line numbers, not semantic IDs — the ADR-0010 "SessionVerifier" rot case).
4. Baseline mode semantics for adopting on an existing corpus (the Vale onboarding lesson).
5. How the skill's write-protocol decides "is this worth an entry at all" (the Kiro
   proportionality failure).
