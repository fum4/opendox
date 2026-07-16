# Decision & Spec Tooling — Research Report

> Historical research document (2026-07-16). Stats verified via GitHub/API and primary
> sources, July 2026. ADR ecosystem; Spec Kit/OpenSpec; Kiro/arc42/Diátaxis; RFC-process
> tooling — assessed for what opendox can learn/own.

---

## 1. ADR ecosystem

### adr-tools (npryce/adr-tools)
- **What**: The original bash ADR CLI (2016) — `adr new`, supersede links, TOC/graph generation over numbered markdown in `doc/adr/`.
- **Does well**: Zero-dependency, zero lock-in (output is plain markdown, so tool death doesn't strand data); made "start doing ADRs" a one-command decision; popularized the `NNNN-title.md` convention.
- **Fails**: **Dead** — last real commit 2020-03-30, last release 2018; issue #94 "Still maintained?" open since March 2020, unanswered; a 2025 stewardship-transfer offer (#151) also ignored. Create-only lifecycle: nothing verifies supersede links, detects staleness, or connects ADRs to code. Bash made it unattractive to extend → graveyard of ports (adr-tools-python, dotnet-adr, adr-j) that also mostly died.
- **Adoption**: 5.6k stars, 631 forks — highest in category; the *convention* it spread outlived the binary by six years and counting.
- **Lessons for opendox**: (1) The format survives, the tool dies — make the spec load-bearing and useful with zero tooling installed; CLI optional. (2) Single-maintainer BDFL = silent death; org-own it, and make the linter the artifact worth maintaining (adr-tools had no verifier, so its rot was invisible).

### log4brains (thomvaill/log4brains)
- **What**: TypeScript CLI + Next.js static-site generator (2020): interactive `adr new`, hot-reload preview, publishable searchable ADR knowledge base.
- **Does well**: Best authoring UX of its generation; proved appetite for *browsable* ADRs. Still ~12k npm downloads/month despite abandonment.
- **Fails**: Stalled April 2021 (maintainer overload), one solo revival burst Dec 2024 (v1.1.0), silent 19 months since. Open issue (Oct 2025): "yarn audit says 84 critical vulns" — unanswered. Root cause: heavy stack (bundled Next.js app, Lerna monorepo) = enormous maintenance surface for a docs tool, carried by one person. Value was the rendered site, so a broken build = worthless tool. No enforcement — pretty rendering of rotting content.
- **Adoption**: 1.5k stars; inertia installs of a known-vulnerable tool show users don't migrate off dead docs tooling.
- **Lessons**: (1) Keep runtime surface near zero — a deterministic linter with no framework deps ages like bash, not like a bundled webapp; any renderer must be separable and sacrificial. (2) Publish an explicit maintenance/status contract; log4brains' README never admitted its state.

### MADR template (adr/madr)
- **What**: Not a tool — a semver'd markdown *template spec* (context/options with "Good, because…"/decision outcome/consequences + YAML front matter: `status`, `date`, `decision-makers`, `consulted`, `informed`).
- **Does well**: **The ecosystem's sole survivor** — v4.0.0 (Sept 2024), commits as recent as July 2026, org-governed, changelog-disciplined. Zero runtime = zero decay; consumable by any editor, generator, or LLM; absorbed dead tools' users. Structured argument fields are already machine-parseable "why."
- **Fails**: A template enforces nothing: no linter for MADR validity, no provenance model, `status: superseded` is an honor-system string. Even a spec pays migration tax — 3.0's "Any Decision Records" rename was reverted in 4.0, field renames broke downstream tooling.
- **Adoption**: 2.3k stars, 463 forks, only 29 open issues; JavaLand 2026 talks; the de facto ADR format.
- **Lessons**: (1) Spec-first is the proven survival strategy — semver the opendox format with a changelog, tools as replaceable satellites; avoid rename churn. (2) MADR's front matter is the extension point — provenance fields + resolvable references are the genuine gap; extend rather than invent a competing template.

### adr-manager (adr/adr-manager)
- **What**: Web-based form editor for MADR files via GitHub OAuth; University of Stuttgart student project (ICSE SCORE contest 2021).
- **Does well**: Lowered entry barrier; survived organizationally longer than log4brains by living in the `adr` org.
- **Fails**: Dormant since Dec 2023 (only activity since: a README edit). Its OAuth-proxy dependency (Pizzly) died underneath it and forced a rewrite. Editing-only — no lifecycle, no validation of decision-vs-code.
- **Adoption**: 155 stars; no evidence of real team usage.
- **Lessons**: (1) Anything with an auth flow/hosted component inherits an ops treadmill — opendox core must run offline, in-repo. (2) Research-born tools demo well and die young.

### ADR + AI discourse (2024–26)
ADRs-as-agent-context is now mainstream (Chris Swan: "natural language, perfect for LLMs"; ADR-generation skills are marketplace commodities). **Enforcement is the emerging frontier**: Mneme HQ intercepts agent Edit/Write calls and blocks diffs violating relevant ADRs — but that's *LLM-judged policy* enforcement, complementary to (not overlapping) opendox's deterministic reference linting. The 2026 critical literature (Java Code Geeks) lands exactly on opendox's thesis: ADR rot is *architectural, not cultural* — point-in-time documents asked to perform a living-artifact function; "superseded means someone remembers to update the status field — in practice, nobody does."

---

## 2. GitHub Spec Kit (github/spec-kit, Sept 2025)

- **What/mechanism**: CLI (`specify init`) installs slash commands: `/speckit.constitution` → `/speckit.specify` (spec.md) → `/speckit.plan` → `/speckit.tasks` → `/speckit.implement`; each feature gets a numbered `specs/` dir, usually own branch. Later: `/speckit.converge` (LLM diff of codebase vs spec) and an "Evolving Specs" brownfield guide.
- **Does well**: Forces requirements thinking before code; massive distribution (GitHub brand, all major agents); `constitution.md` is the one artifact users consistently keep alive.
- **Fails**: (a) **Review burden** — Birgitta Böckeler (martinfowler.com): "numerous repetitive markdown files that were tedious to review… I'd rather review code than all these markdown files." (b) **Illusion of work** — top repo discussion #1784: agent drowns in kilobytes of its own spec text, over-engineers, then ignores the spec anyway; one user improved results by *cutting docs by two-thirds*. (c) **No lifecycle answer** — the evolving-specs guide offers *three competing patterns* (flow-forward / living-spec / flow-back) and makes you choose; default behavior is per-feature dirs that fossilize. (d) **Zero deterministic enforcement** — converge/analyze are LLM-judged, nothing mechanically couples spec to code. (e) Greenfield bias.
- **Adoption**: ~122k stars, 10.8k forks, 192 releases, active as of 2026-07-15 — momentum held, but 2026 HN sentiment ("Ask HN: still using SDD?") is "yes, but slimmed": smaller specs, hybrid with plain plan mode, agent-maintained.
- **Lessons**: (1) The #1 complaint is reviewing generated prose — opendox's "only non-recoverable why" is the antidote; enforce brevity *structurally*, because agents pad. (2) "Pick one of three lifecycles" = no lifecycle; opendox needs exactly one deterministic one. (3) Nothing in the market mechanically verifies anything — the deterministic linter is the differentiator, not the template.

---

## 3. OpenSpec (Fission-AI/OpenSpec) and peers

- **What/mechanism**: Lighter, brownfield-first SDD (Aug 2025). Two tiers: `openspec/specs/` = living truth of what *is* built; `openspec/changes/<name>/` = proposals with **delta specs** (ADDED/MODIFIED/REMOVED). Workflow: propose → apply → `openspec archive`, which **deterministically folds deltas into the living specs** and moves the change to `changes/archive/YYYY-MM-DD-<name>/` with rationale preserved.
- **Does well**: The only tool with a clean post-ship lifecycle (delta → merge → dated archive with provenance); delta review beats full re-spec for brownfield; `openspec validate --strict` is a real deterministic CLI gate people actually run.
- **Fails**: Validator is **structure-only** (sections present, delta well-formed) — docs explicitly disclaim spec↔code sync ("depends on implementation discipline"). Worse, the deterministic merge itself has correctness bugs: issue #1246 — archiving two changes that MODIFY the same requirement **silently drops scenarios**. 476 open issues vs ~630 commits = maintenance strain.
- **Adoption**: ~61k stars, active (v1.6.0, July 2026); the "what people actually moved to" answer in 2026 threads.
- **Lessons**: (1) The delta→fold→archive shape is right and users accept a deterministic CLI step — but nobody has climbed the next rung: verifying references against the repo. That rung is opendox. (2) Deterministic merges must fail loud on collision, never drop silently.

**Peers, briefly**: **Tessl** ($125M raised; spec-as-source, code as compiled output) — Framework still not GA ~9 months after announcement; the maximalist bet stalls in practice; its Spec Registry's *measured eval scores per artifact* is a good provenance precedent. **BMAD-METHOD** (~50k stars) — 12-persona agile theater producing PRD/architecture/story artifact sprawl, no post-implementation lifecycle; the cautionary tale opendox's non-recoverability rule exists to prevent. **Agent-OS** (~5k stars, momentum faded by mid-2026) — unbranded markdown-scaffolding frameworks without a verification moat commoditize fast.

---

## 4. AWS Kiro spec system

- **What/mechanism**: AWS agentic IDE (preview July 2025, GA Nov 2025). Per feature: `.kiro/specs/<feature>/` with `requirements.md` (EARS notation: "WHEN X, THE SYSTEM SHALL Y"), `design.md`, `tasks.md` (dependency-graph task execution), plus repo-level steering files (`product.md`, `tech.md`, `structure.md`). Human phase gates between files.
- **Does well**: Simplest 3-file mental model of the SDD tools (Böckeler: "most lightweight of the three"); EARS makes acceptance criteria testable *in shape*; steering files widely praised.
- **Fails**: (a) **Proportionality** — Fowler-site tester: small bug fix → "4 user stories with 16 acceptance criteria… a sledgehammer to crack a nut"; one trial produced "passing tests but zero useful code." (b) **Spec-first, not spec-anchored** — specs effectively discarded post-merge; AWS's own engineer conceded specs are "mostly static documents" needing manual refresh; competitors bolted on reconcile commands precisely because of this gap. (c) Duplicated intent when importing existing rules. (d) Launch ops mess: 100k+ waitlist, then Aug 2025 pricing backlash ("wallet-wrecking" — spec requests ~5× vibe requests + a metering bug).
- **Adoption/status 2026**: GA; AWS claims 250k+ preview devs; Amazon Q Developer being folded into it. Reputation dented by Feb 2026 FT report tying internal Kiro agents to AWS production outages (Amazon disputes). Reviews nuanced: hooks/steering good, spec workflow ceremony + IDE lock-in bad.
- **Lessons**: (1) **Docs anchored to task lifecycle rot at merge; anchor to code lifetime** — provenance + verified-against-commit stamps + linter-enforced freshness is exactly the missing organ. (2) Enforce an "is this worth an entry at all?" threshold, or agents generate 16 acceptance criteria for a one-liner.

---

## 5. arc42 / Diátaxis

### arc42
- **What**: Starke/Hruschka's 12-section architecture-doc template (~2005): constraints, context, building blocks, runtime, deployment, crosscutting concepts, decisions, quality, risks/debt, glossary. Pure template, "skip what you don't need."
- **Does well**: Got the *taxonomy* of why-knowledge right 20 years ago — §2 Constraints, §9 Decisions, §11 Risks/Debt are squarely non-recoverable "why." Tool-agnostic, docs-as-code friendly (docToolchain).
- **Fails**: Template-filling anti-pattern — sections filled with boilerplate to "complete" it; heavyweight for agile teams. 2026 practitioner series (hompus.nl): "structure alone does not keep documentation alive… it dies because there is no workflow that keeps it connected to change" — and all its proposed fixes are manual ritual (quarterly read-throughs). Adoption plateau: DACH enterprise niche; ~1.2k stars after 20 years. Small AI ecosystem emerging (docToolchain LLM-Prompts, arc42-toolkit Claude skills) — which mostly demonstrates agents will happily generate plausible filler for every template slot.
- **Lessons**: (1) A correct taxonomy without an enforcement loop is a corpse — the linter is the organ arc42 never had. (2) Comprehensive templates breed boilerplate; opendox should have **no fill-me sections** — entries exist only when a non-recoverable fact exists.

### Diátaxis
- **What**: Procida's four-mode compass (tutorials / how-to / reference / explanation on acquisition↔application × action↔cognition axes). A thinking framework, deliberately not a template.
- **Does well / why it spread**: Zero adoption cost (no tooling, no format, works incrementally on one page), names a universally felt failure (the mixed-mode page), one-sentence teachable. Adopters: Django, Canonical (foundation for all company docs), Cloudflare, Gatsby, hundreds more.
- **Fails**: Cargo-culted into four literal folders — Pigweed (Google): "resulted in too much fragmentation… annoying to jump back-and-forth"; Procida's own fix ("modes, not pages") confirms it under-specifies structure so adopters over-specify it. **No lifecycle, provenance, or freshness story at all** — a compass, not a maintenance system. Human-reader assumptions (quadrants defined by a human's momentary need; agents mostly want reference + explanation).
- **The explanation quadrant is opendox's territory, un-operationalized**: Diátaxis says explanation covers "design decisions, historical reasons, technical constraints," warns it "tends to absorb other things" and must stay "closely bounded" — but gives no schema, no unit of record, no trigger for when it must be written or updated. It's notoriously the least-written quadrant because it never blocks anyone's task. opendox is essentially *explanation, operationalized for agents*.
- **Lessons**: (1) Spread comes from a one-sentence idea with zero adoption cost — keep "only non-recoverable why" that teachable, and lint *semantics* (recoverable? provenanced? resolvable?) rather than mandating directory shape, or opendox gets four-folders-cargo-culted too. (2) Mechanize the "explanation absorbs everything" warning as lint rules: ban restating code, ban how-to creep, require each entry closely bounded to a scope — which conveniently supplies the provenance anchor.

---

## 6. RFC-process tooling / design-docs-as-code

### Oxide RFD system (oxidecomputer/rfd-site, rfd-api)
- **What**: AsciiDoc RFDs in a git repo with front-matter `state` metadata (prediscussion → discussion → published/abandoned), discussed via PRs, plus serious custom tooling: a web frontend with full-text search (self-hosted Meilisearch), per-user access control, inline display of GitHub PR comments next to document lines, hover-previews showing title/authors/status/last-update of cross-referenced RFDs, short URLs, a Matrix bot.
- **Does well**: The best-in-class *reading and discussion* experience; metadata is machine-consumed (CSV export, automation); RFDs are genuinely the company's backbone ("Oxide and Friends" episode). Cross-RFD link previews with status are a mini reference-integrity system.
- **Fails**: Explicitly a *discussion* record, not living documentation — "published" is terminal; no mechanism keeps a published RFD true as the system evolves, and no code-reference verification. Also the tooling cost is a full-time-ish investment only a doc-culture company makes; nobody else has replicated it (a lone university fork exists).
- **Adoption**: One company (plus forks); influential as a pattern, not as a product.
- **Lessons**: (1) Machine-readable state metadata + tooling that *surfaces* status at every cross-reference (hover shows state + last-update) is cheap and high-value — opendox's verified-against-commit stamp should be surfaced wherever a doc is referenced, not buried in front matter. (2) Even the best RFC system punts on post-publication truth — "published" ≠ "still true."

### Rust RFCs / Kubernetes KEPs / Python PEPs (the big open-source proposal processes)
- **What**: Markdown proposals in a dedicated repo, PR-based discussion, acceptance merge. KEPs add `kep.yaml` machine-readable metadata (status, owning-sig, stage, milestone) explicitly to "reduce tribal knowledge" and drive release-tracking tooling.
- **Does well**: Durable public record of *why*; KEP metadata proves per-doc YAML status blocks work at scale (release team tooling filters on them every cycle).
- **Fails — this is the cluster's clearest lifecycle evidence**:
  - Rust (ncameron, "We need to talk about RFCs"): there is "**no explicit process for updating RFCs and tracking the status**" — readers *cannot tell whether an accepted RFC was implemented as written or changed significantly*; 54+ RFCs from 2020+ still open; discussion volume overwhelms value; incentives work against ever closing stale proposals.
  - Kubernetes: KEP process itself acknowledges review-bandwidth bottleneck; stale KEPs accumulate despite the metadata, because the metadata records *process stage*, not *truth against code*.
  - **Python solved it honestly**: PEP 1 declares that once resolved, a PEP is a **historical document, not a living specification** — implemented PEPs get a banner: "This PEP is a historical document. The up-to-date, canonical spec is maintained at <living-docs location>" (e.g. PEPs 426/588/627/643 → PyPA specs). The proposal record and the living truth are *different documents with an explicit pointer between them*.
- **Lessons**: (1) **The PEP pattern is the correct lifecycle answer for the whole cluster**: decision/proposal docs are immutable history; "why" that must stay true lives in a separate living layer, with a mandatory forward pointer — opendox should encode this as a first-class rule (a shipped plan/spec either gets a "historical, superseded by <living doc>" stamp or the linter flags it). (2) Metadata that tracks *process stage* (KEP) still rots; only metadata verified *against the repo* (opendox's linter) tracks truth.

---

## Synthesis: what this cluster fails at that opendox can own

**1. Nobody has built the verifier. Anywhere.** Across ~20 years and every tool above, the state of the art in enforcement is: OpenSpec's structure-only validator, Spec Kit's LLM-judged "converge," Mneme's LLM-judged policy hooks, Oxide's hover-status previews. **Zero tools deterministically check doc content against the repo** — do referenced paths/symbols/commands exist, do supersede/cross-doc links resolve, was this verified at some commit. The sharpest community critique of the whole SDD wave (HN): "there's nothing that couples the english-language specs with the actual code and behaviour of the system." That deterministic linter is opendox's unoccupied ground — it's the fifth `adr new` command nobody needs vs the first verifier everybody lacks.

**2. The lifecycle problem — how the cluster handles "spec ships → drift bomb":**

| System | Post-ship answer | Verdict |
|---|---|---|
| ADR tools | `status: superseded` honor-system string | rots (nobody updates the field — documented) |
| Spec Kit | "pick one of three patterns" | non-answer; default = per-feature dirs fossilize |
| Kiro | specs "mostly static," manual refresh | worst-in-class; rot begins at merge |
| OpenSpec | deterministic fold-into-living-specs + dated archive | best shape, but living layer unverified + merge bugs |
| arc42/Diátaxis | manual ritual / nothing | rots / out of scope |
| Rust RFC/KEP | no update process / stage-only metadata | acknowledged failure |
| **Python PEP** | **historical-document banner + pointer to canonical living doc** | **the only honest, working answer** |

opendox should fuse the two working pieces: **OpenSpec's deterministic fold step** (change-scoped why-capture merges into a living layer, with dated provenance archive) + **PEP's historical/canonical split** (anything task-shaped is immutable history with a mandatory forward pointer) + **the linter nobody built** (living layer references must resolve; verified-at-commit stamps go stale mechanically, not socially). Rot detection becomes `git diff`-cheap instead of quarterly-review-expensive.

**3. Generated-prose volume is the new #1 failure mode.** Pre-2023 the binding constraint was human authoring friction (killed arc42-style comprehensiveness, kept ADRs elite). With agents as authors it inverted: the top practitioner complaints about Spec Kit/Kiro/BMAD are markdown flood, review fatigue, context bloat degrading the agent itself, and "illusion of work." opendox's non-recoverability ban is precisely the counter — but it must be *structurally enforced* (entry-count/size discipline, no fill-me sections, ban description/how-to creep into why-entries), because every template slot offered to an LLM gets filled with plausible filler.

**4. Format outlives tool — every time.** adr-tools, log4brains, adr-manager, Agent-OS all died or faded; MADR (semver'd template spec, org-owned, zero runtime) and Diátaxis (one-sentence idea, zero adoption cost) survived. opendox's durable core should be the semver'd spec + linter rules; the CLI/skill are replaceable satellites; no hosted components, no framework-heavy stack; extend MADR's front matter (provenance, verified-at) rather than competing with it; lint semantics, not directory shape, to avoid the four-folders cargo-cult.

**5. Provenance is asked for, never shipped.** MADR has `decision-makers` but no owner-stated vs inferred distinction; OpenSpec's dated archives and Tessl's registry eval-scores gesture at it; the 2026 agent-memory literature explicitly converges on rules-with-provenance-and-expiry and "record why it's gone so the agent doesn't bring it back." opendox's owner-stated/agent-inferred split + verified-against-commit stamps is the concrete implementation of what the field is circling.

Key sources: [spec-kit](https://github.com/github/spec-kit) · [spec-kit discussion #1784](https://github.com/github/spec-kit/discussions/1784) · [OpenSpec](https://github.com/Fission-AI/OpenSpec) · [OpenSpec #1246](https://github.com/Fission-AI/OpenSpec/issues/1246) · [Böckeler: SDD tools (martinfowler.com)](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) · [HN: Ask HN still using SDD?](https://news.ycombinator.com/item?id=46864948) · [HN: spec↔code coupling critique](https://news.ycombinator.com/item?id=47422065) · [adr-tools](https://github.com/npryce/adr-tools) · [log4brains](https://github.com/thomvaill/log4brains) · [MADR](https://github.com/adr/madr) · [adr.github.io](https://adr.github.io/) · [Java Code Geeks: ADR rot is architectural](https://www.javacodegeeks.com/2026/05/the-reason-most-architecture-decision-records-get-written-and-never-read-is-architectural-not-cultural.html) · [Mneme: agents + ADRs](https://mnemehq.com/insights/how-ai-coding-agents-use-adrs/) · [Kiro specs](https://kiro.dev/docs/specs/) · [Register: Kiro pricing](https://www.theregister.com/2025/08/18/aws_updated_kiro_pricing/) · [arc42](https://arc42.org/overview) · [hompus.nl: arc42 after the 12 chapters](https://blog.hompus.nl/2026/02/25/arc42-after-the-12-chapters/) · [Diátaxis: explanation](https://diataxis.fr/explanation/) · [HN: Diátaxis/Pigweed fragmentation](https://news.ycombinator.com/item?id=42340740) · [Oxide: A Tool for Discussion](https://oxide.computer/blog/a-tool-for-discussion) · [ncameron: We need to talk about RFCs](https://www.ncameron.org/blog/the-problem-with-rfcs/) · [KEP process](https://github.com/kubernetes/enhancements/blob/master/keps/README.md) · [PEP 1](https://peps.python.org/pep-0001/) · [PEP 627 historical banner example](https://peps.python.org/pep-0627/)
