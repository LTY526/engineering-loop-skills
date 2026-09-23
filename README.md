# Engineering Loop Skills

Three reusable skills for exploring, planning, and building features in a GitHub repository. The skill files use the shared `SKILL.md` format and can run in Codex or Claude Code.

- [`explore`](skills/explore/SKILL.md) builds a small prototype on an isolated branch, refines it through user feedback, and records a handoff.
- [`planning`](skills/planning/SKILL.md) turns the agreed direction into a reviewed architecture decision, a testable spec, and a Ready issue.
- [`build`](skills/build/SKILL.md) checks a Ready issue, delegates implementation and independent QA, and prepares a PR with verification evidence.

Use `explore` when trying a feature will clarify its design. Otherwise start with `planning`. The usual path is **explore (optional) → planning → build**. Planning reads the exploration handoff; build can reuse prototype code when it fits the reviewed spec.

## Install

Copy the three folders under `skills/` into the skills directory for your agent:

| Agent | Personal skills | Project skills | Invoke |
| --- | --- | --- | --- |
| Codex | `$CODEX_HOME/skills/` (default `~/.codex/skills/`) | `.codex/skills/` | `$explore`, `$planning`, `$build` |
| Claude Code | `~/.claude/skills/` | `.claude/skills/` | `/explore`, `/planning`, `/build` |

Claude Code's [skill documentation](https://code.claude.com/docs/en/skills) describes those locations and slash commands. These files are portable instructions; each agent uses its own tools to carry them out. For the independent reviews in planning and build, Claude Code uses a fresh, non-fork `Agent` subagent; Codex uses `spawn_agent` with no inherited conversation. If either tool is unavailable, report the missing review step instead of claiming it happened.

## Project requirements

`explore` needs Git and a branch or worktree for the prototype. `planning` and `build` additionally need an authenticated GitHub CLI (`gh`), an `origin/main` branch, GitHub Issues, and `ready`, `in-progress`, and `blocked` labels. Planning writes decisions under `docs/decisions/` and specs under `docs/specs/`, publishes them to `origin/main`, and only then creates a Ready issue. Build branches from that ref.

## Developer context

Explore or planning can ask once about the developer's familiarity with the project's language and preferred framework analogies. If the developer wants this saved, use `$CODEX_HOME/developer-context.md` (default `~/.codex/developer-context.md`) in Codex or `~/.claude/developer-context.md` in Claude Code. Build reads the same file. It affects explanations, not acceptance criteria, and is not stored in this repository.

## Layout

```text
skills/
  explore/SKILL.md
  planning/SKILL.md
  build/SKILL.md
```

The skills do not modify a project until invoked there.
