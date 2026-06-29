---
name: architecture-align
description: Align work with a centralised architecture / ADR repo when making or recording an architectural decision, in any project. Use when choosing a technology, changing a module/service boundary, data flow, or API contract, adding a cross-cutting pattern, writing/updating an ADR, or when a change might contradict a documented architectural decision. Reads central + project ADRs, checks alignment, flags deviations for discussion, and keeps docs current.
---

# architecture-align

Keep a project's decisions consistent with its centralised architecture authority, and
keep documentation aligned during the session. Domain-agnostic — applies to application,
service, infrastructure, data, or library repos alike.

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
  clone the repo named in the project's `CLAUDE.md`). Typical contents:
  - `docs/adr/NNNN-*.md` — authoritative ADRs
  - `docs/documentation/` (or similar) — longer-form architecture docs
  - a model/target file if the org keeps one (e.g. C4/Structurizr `workspace.dsl`,
    `ARCHITECTURE-TARGET.md`)
- **Project-local:** conventionally `<project>/docs/adr/NNNN-*.md`, with longer docs in
  `<project>/docs/`. Defer to whatever the project's `CLAUDE.md` declares if different.

## Procedure

1. **Read down the chain.** Read the relevant central ADRs first (under
   `$CENTRAL_ARCHITECTURE_PATH/docs/adr/`), then the project-local ADRs. Skim the
   target/model file if the change is structural. The central clone is on disk — read
   files directly. If it looks stale, note that and offer to
   `git -C "$CENTRAL_ARCHITECTURE_PATH" pull`.
2. **Check alignment.**
   - *Conforms* → proceed; cite the ADR(s) you're implementing.
   - *Not covered* → proceed, and note it's a new local decision (candidate ADR).
   - *Conflicts* → **STOP.** State the conflict plainly: which central ADR, what it
     says, what you'd do instead, and why. Ask the user how to proceed. Never silently
     diverge.
3. **Record the decision.** Once decided, write/update the project-local ADR
   (`<project>/docs/adr/NNNN-title.md`, next number, same format as existing ones).
4. **Flag central drift.** If the decision changes or supersedes a central ADR, say so
   and propose the matching edit in the central repo (separate repo, separate commit) —
   don't assume it's done.

## Output

When you act under this skill, tell the user: which ADRs you read, the alignment
verdict (conform / new / conflict), and any central-repo change still owed.
