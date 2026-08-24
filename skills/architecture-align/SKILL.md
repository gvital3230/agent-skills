---
name: architecture-align
description: Align work with a centralised architecture repo when making or recording an architectural decision, in any project. Use when choosing a technology, changing a module/service boundary, data flow, or API contract, adding a cross-cutting pattern, writing/updating an ADR, or when a change might contradict a documented architectural decision. Resolves the task to rule IDs via the central INDEX.yaml, reads only those rules and blocks, checks alignment, records contained deviations, and routes shared changes through a central proposal.
---

# architecture-align

Keep a project's decisions consistent with its centralised architecture authority, and
keep the alignment record current during the session. Domain-agnostic — applies to
application, service, infrastructure, data, or library repos alike.

## When this applies

Architectural decisions only — technology choices, module/service boundaries, data
flows, API/interface contracts, cross-cutting patterns, anything you'd write an ADR
about. Skip for routine edits (renaming a variable, bumping a version, fixing a typo)
unless they touch a documented decision.

## How resolution works

This skill is generic. The **project supplies the specifics** — read the repo's
`CLAUDE.md` (and any nested one) for: which central repo this codebase aligns to, and
where project-local ADRs live. The skill provides the procedure; the repo provides the
facts.

- **Central repo:** its local clone path is in the env var **`$CENTRAL_ARCHITECTURE_PATH`**
  — the standard convention across repos, set per machine in a gitignored
  `.claude/settings.local.json` (a nested settings file can override it for a subtree
  belonging to a different company/product). Never assume a hardcoded path. If the var
  is unset or the path is missing, **ask** the user for the clone location (or offer to
  clone the repo named in the project's `CLAUDE.md`). If the clone looks stale, note
  that and offer to `git -C "$CENTRAL_ARCHITECTURE_PATH" pull`.
- **Entry point:** `$CENTRAL_ARCHITECTURE_PATH/INDEX.yaml` — the machine-readable map of
  the central repo's three layers: `decisions/` (*why* — ADRs, read only when preparing
  a proposal or needing rationale), `rules/` (*what* — one-paragraph rules with stable
  IDs under `## <ID>:` headings), `blocks/` (*how* — copyable artifacts, each declaring
  a `kind:`: **`literal`** blocks consumer repos copy byte-for-byte, **`skeleton`**
  blocks whose interface is normative and whose recipe is the service's; ADR-0022 §8).
  `INDEX.yaml` maps every rule ID to its file and founding ADR, every block to its path,
  `kind:` and the rule IDs it implements, plus `stacks:` (golden stacks for new
  services) and `services:` (per-service configuration rows).
  **If `INDEX.yaml` does not exist, the central repo predates this layout — use the
  Fallback section at the bottom instead of the three modes.**
- **Project-local:** ADRs conventionally at `<project>/docs/adr/NNNN-*.md`; recorded
  variances in `<project>/docs/deviations.yaml`; copied-block records in the file the
  project keeps its `PROVENANCE` lines in. Defer to the project's `CLAUDE.md` if it
  declares differently.

**Citations** (ADR-0022 §6): a bare `ADR-NNNN` always means the central repo's
`decisions/NNNN-*.md`. A service-local decision is always qualified `<service>:ADR-NNNN`
(e.g. `catalog:ADR-0001`). Rule IDs (`NAM-`/`TST-`/`DEP-`/`API-`/`EVT-`/`OBS-`) take no
qualifier — only the central repo defines them.

## Mode 1 — Comply (the default: doing work under the rules)

1. **Load `INDEX.yaml`. Match the task to rule IDs** by domain prefix and title; pull
   this service's row from `services:` for its configuration (deploy profile, registry
   keys, tooling).
2. **Read only the matched rules** — the `## <ID>:` sections of the rule file(s) the
   index names, plus any rule they link to. Do not read whole documents, unmatched
   domains, or the founding ADRs — a rule's `Why:` link is for proposals (mode 3), not
   for compliance.
3. **Where a matched rule names a `Block:`, read that block's `kind:` in `INDEX.yaml`
   first — it decides what copying means** (ADR-0022 §8):
   - **`kind: literal`** — copy byte-for-byte. Never reimplement or paraphrase it.
   - **`kind: skeleton`** — the *interface* is normative, the recipe is not. Keep the
     names, argument shape, output contract, required keys and house style exactly;
     write the commands in the service's own toolchain. Angle-bracket tokens (`<svc>`,
     `<toolchain>`, `<env>`) are fill-ins — substitute them. A JVM service copying
     `blocks/shared/Makefile` keeps DEP-2's target set and each target's meaning, and
     writes `check: ; ./gradlew check` — that is conformance, not divergence.

   Either way, record the copy as a `PROVENANCE` line:
   `<block name> <source repo-relative path> <source commit sha> <date copied>`
   (ADR-0022 §8). For a **new service**: choose a golden stack from `stacks:` and stamp
   the matching scaffold (`blocks/service-<stack>/`, where it exists) plus every
   stack-neutral block the matched rules name.
4. **Cite the rule IDs** you complied with in your output.

If the work at hand cannot satisfy a matched rule, that is a divergence — switch to
mode 2's fork before proceeding.

## Mode 2 — Check / converge (auditing a repo against the rules)

1. **PROVENANCE diff first — and the block's `kind:` decides what the diff proves.**
   For each `PROVENANCE` line, compare the recorded sha with the central block at its
   recorded path. An **out-of-date sha is a mechanical finding for either kind**. Then:
   - **`kind: literal`** — diff the copied file's content against the current block too;
     a content diff is a mechanical finding on its own, no interpretation needed.
   - **`kind: skeleton`** — **do not report a content diff as a finding.** A skeleton is
     expected to differ: its recipe is the service's. Check it against the rule IDs in
     the block's `implements:` list instead (step 2) — are the target names, argument
     shape, output contract, required keys and house style still what the rules say?
     "This Makefile runs `./gradlew` where the block runs `npm`" is not a finding.
2. **Then the rule-level diff.** Resolve which rule IDs apply to this repo via
   `INDEX.yaml` (its `services:` row tells you which profiles and domains bind it), read
   those rule sections, and compare against repo reality. An existing
   `docs/deviations.yaml` entry makes its rule a **known variance** — report it as such,
   not as a fresh finding.
3. **Every finding cites a rule ID.**
4. **For each unrecorded divergence, run the blast-radius fork** (ADR-0022 §5) — the
   containment test is mechanical, not a severity judgement: *does any artifact another
   repo consumes change* — a registry key, a published schema, an envelope field, a
   queue or topic name, a generated client's shape?
   - **Contained (no):** write the entry in this repo's `docs/deviations.yaml`, keyed by
     the rule ID, with `summary` (what the repo does instead), `since` (date), `reason`,
     and `revisit` — a **named trigger** or `permanent — <why>` — and **proceed**. No
     central PR, no waiting.
   - **Shared (yes):** **STOP** — agreement in the central repo comes first. Go to
     mode 3.

## Mode 3 — Propose (changing the central repo)

- **A rule change is an ordinary PR** to the central repo: edit the rule's paragraph in
  `rules/`, keep `INDEX.yaml` in step. Rule IDs are append-only — a retired rule keeps
  its number, marked retired; numbers are never reused.
- **Reversing or superseding a decision takes an ADR first**: a new `decisions/` entry,
  then the rule edit citing it. Check the rule's `Why:` link to find which decision
  you're actually arguing with, and read that ADR before proposing.
- Use qualified citations throughout (bare `ADR-NNNN` = central; `<service>:ADR-NNNN` =
  that service's local ADR).
- **Recorded deviations need no proposal.** The central repo reconciles the fleet's
  `deviations.yaml` files in a periodic batch sweep — each entry is adopted into the
  rule, accepted as a standing variance, or rejected with convergence work filed. When a
  mode-2 entry will simply be picked up by the sweep, say so instead of opening a PR.

## Fallback — central repo without `INDEX.yaml`

An older central repo (or a clone predating the restructure) keeps its authority as ADRs
in `docs/adr/NNNN-*.md`, with no rule layer. Then:

1. List `$CENTRAL_ARCHITECTURE_PATH/docs/adr/` and match the task to relevant ADRs by
   number and title; **read only the matched ADRs**, not the directory wholesale. Skim
   the target/model file (e.g. a C4/Structurizr `workspace.dsl`) only if the change is
   structural. Then read the project-local ADRs that touch the same ground.
2. Check alignment: *conforms* → proceed, citing the ADR(s); *not covered* → proceed,
   noting a new local decision (candidate ADR); *conflicts* → **STOP** and state the
   conflict plainly — which central ADR, what it says, what you'd do instead, and why —
   then ask the user. This layout has no deviation protocol, so never diverge silently.
3. Record the outcome as a project-local ADR (`<project>/docs/adr/NNNN-title.md`, next
   number, same format as existing ones); if the decision changes or supersedes a
   central ADR, propose the matching central edit separately — don't assume it's done.

## Output

When you act under this skill, tell the user: which mode ran, which rule IDs (or, in
fallback, ADRs) you resolved and read, the verdict per finding — *conform* / *known
variance* / *contained, deviation recorded* / *shared, proposal owed* / *new local
decision* — and any central-repo change or `deviations.yaml` write still owed.
