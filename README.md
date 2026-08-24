# agent-skills

Reusable [agent skills](https://github.com/vercel-labs/skills) for Claude Code (and other
SKILL.md-compatible agents). Each skill is a directory under `skills/` containing a
`SKILL.md` with YAML frontmatter (`name`, `description`).

## Install

Using [`npx skills`](https://github.com/vercel-labs/skills):

```bash
# one skill into the current project (./.claude/skills)
npx skills add gvital3230/agent-skills --skill architecture-align -a claude-code

# globally, for every project (~/.claude/skills)
npx skills add gvital3230/agent-skills --skill architecture-align -a claude-code -g
npx skills add gvital3230/agent-skills --skill architecture-mentor -a claude-code -g

# list everything available in this repo
npx skills add gvital3230/agent-skills --list
```

Or just copy a skill directory into your `~/.claude/skills/` (user scope) or
`.claude/skills/` (project scope).

## Skills

`architecture-align` and `architecture-mentor` are a pair and share one setup. Install both
if you work against a central architecture repo: align answers *does this comply, and what
do I copy*; mentor answers the questions no rule decides. **This repo is their source of
truth** — the central architecture repo is what they read, not where they live.

### `architecture-align`

Align work with a centralised architecture repo when making or recording an architectural
decision — in any project. Resolves the task to rule IDs through the central `INDEX.yaml`,
reads only those rules and the blocks they name, checks alignment, and routes divergence by
blast radius.

**Three modes:**

- **Comply** — match the task to rule IDs, read only those rule paragraphs, copy the blocks
  they name literally (recording a `PROVENANCE` line), cite the IDs.
- **Check / converge** — `PROVENANCE`-vs-block diff, then a rule-level diff; each finding
  cites a rule ID. Divergence forks on blast radius: contained → record it in the repo's
  `docs/deviations.yaml` and proceed; shared (an artifact another repo consumes changes) →
  stop, agree centrally first.
- **Propose** — a rule change is an ordinary PR to the central repo; reversing a decision
  takes an ADR first.

A **fallback** covers central repos that predate this layout (ADRs in `docs/adr/`, no
`INDEX.yaml`): match ADRs by number and title, read only those, never diverge silently.

**Per-repo setup it expects:**

- The repo's `CLAUDE.md` declares which central architecture repo this codebase aligns to,
  and where project-local ADRs live (conventionally `docs/adr/NNNN-*.md`).
- A `CENTRAL_ARCHITECTURE_PATH` env var pointing to the local clone of that central
  repo — set per machine in a gitignored `.claude/settings.local.json`:

  ```json
  { "env": { "CENTRAL_ARCHITECTURE_PATH": "/abs/path/to/your/architecture-repo" } }
  ```

  The var name is the same in every repo, so the skill stays generic; only the path is
  machine-specific (and never committed).

### `architecture-mentor`

Senior-architect peer review on an architectural question — the judgement counterpart to
`architecture-align`. Align grades work against the rules, treating them as given; mentor is
the one that may question the rules themselves. Two behaviours are always on: an
**overengineering defence** (the burden of proof is on the thing being added) and an
**altitude alarm** (it interrupts, once, when the discussion sinks from *which property must
hold* into fields, flags and names).

**Four modes:**

- **Scope triage** — does this change even belong in the central repo? A six-step test
  ending in a named destination: central rule, block, configuration row, service-local ADR,
  `deviations.yaml` entry, an open question with a revisit trigger, or nothing at all.
- **Completeness** — the angles a design must survive before it is a decision, in five
  clusters (why now · shape · when it goes wrong · what it costs · how it ends), each
  answered or explicitly waived.
- **Simplification audit** — count the moving parts, hunt restatement, unenforceable rules
  and speculative specification, report a ranked list of deletions and merges. "Nothing to
  cut" is a valid result.
- **Second opinion** — does a rule still deserve to stand? Two doors in: the architect points
  at a rule, or evidence indicts one nobody was suspicious of (deviations clustering on one
  rule ID, a premise that moved, a case the founding argument never worked through,
  compliance that became theatre). Argued on named grounds and priced both ways — including
  the verdict *stands, and the fleet is wrong*. It does not manufacture disagreement, and it
  does not let accumulated pressure decide on its own.

**It is grounded, not hermetic.** Advice that has only read the documents is speculative
specification — reasoning about a plausible platform instead of the one that runs. So it
climbs an evidence ladder, stopping the moment a claim is settled: the declared model
(`INDEX.yaml`, `TARGET.md`, the topology, the matched ADRs) → the registers of known
divergence (open questions, the fleet's `deviations.yaml`) → cheap facts from the repos
(existence and shape: a manifest, a lockfile, a CI stage) → the code itself. The declared
model is treated as a **claim, not a fact** — where it disagrees with what the repos
actually do, that disagreement is usually the most valuable finding of the session.

Bounded so it does not become a code audit: it looks only to settle a named claim, reads for
existence and shape rather than correctness, budgets a handful of targeted looks, labels
every statement **declared · observed · assumed**, and reports what it did *not* check.
Bugs are `/code-review`'s job.

It advises in conversation by default, and on request drafts an ADR, a rule paragraph, an
open-questions entry or a `deviations.yaml` entry in the central repo's house format.
Same per-repo setup as `architecture-align` above.
