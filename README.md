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

# list everything available in this repo
npx skills add gvital3230/agent-skills --list
```

Or just copy a skill directory into your `~/.claude/skills/` (user scope) or
`.claude/skills/` (project scope).

## Skills

### `architecture-align`

Align work with a centralised architecture / ADR repo when making or recording an
architectural decision — in any project. Reads central + project ADRs, checks alignment,
flags deviations for discussion before diverging, and keeps docs current.

**Per-repo setup it expects:**

- The repo's `CLAUDE.md` declares which central architecture/ADR repo this codebase
  aligns to, and where project-local ADRs live (conventionally `docs/adr/NNNN-*.md`).
- A `CENTRAL_ARCHITECTURE_PATH` env var pointing to the local clone of that central
  repo — set per machine in a gitignored `.claude/settings.local.json`:

  ```json
  { "env": { "CENTRAL_ARCHITECTURE_PATH": "/abs/path/to/your/architecture-repo" } }
  ```

  The var name is the same in every repo, so the skill stays generic; only the path is
  machine-specific (and never committed).
