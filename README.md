# Engineering Loop Skills

Two Codex skills for a small planning-to-delivery workflow in a GitHub repository.

- [`planning`](skills/planning/SKILL.md) turns a feature idea into a reviewed architecture decision, a testable spec, and a Ready issue. It checks review findings against the code, repairs routine gaps, and explains unfamiliar platform constraints before asking for a product decision.
- [`build`](skills/build/SKILL.md) checks a Ready issue before building, delegates implementation and independent QA, and opens a pull request with verification evidence. Design gaps return to planning; code defects get bounded retries.

## Use in a project

Copy the two folders under `skills/` to your Codex skills directory (`$CODEX_HOME/skills`, or `~/.codex/skills` when `CODEX_HOME` is unset), or to the target project's `.codex/skills` directory. Invoke `planning` to prepare an issue, then `build` to execute Ready issues.

The target project needs Git, an authenticated GitHub CLI (`gh`), an `origin/main` branch, GitHub Issues, and `ready`, `in-progress`, and `blocked` labels. The skills write decision records under `docs/decisions/` and specs under `docs/specs/`. Planning publishes those files to `origin/main` before labeling an issue Ready, because the build skill branches from that ref.

## Developer context

Planning can ask once about the developer's familiarity with the project's language and the frameworks they prefer for analogies. If the developer wants that remembered, the skills use a personal `$CODEX_HOME/developer-context.md` file (or `~/.codex/developer-context.md` when unset). Build reads it without asking again. This file affects explanations, not technical acceptance criteria, and is not part of this repository.

## Layout

```text
skills/
  planning/SKILL.md
  build/SKILL.md
```

The skills are instructions; they do not include a CLI or modify a project until invoked there.
