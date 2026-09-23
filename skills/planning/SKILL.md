---
name: planning
description: "Turn an idea into a codebase-grounded spec and ADR, review and repair the plan, then create a Ready GitHub issue. Explain unfamiliar platform constraints in terms the developer knows. Never writes production code."
---

# planning

Turns a raw idea into two committed files plus one Ready GitHub issue that only ever *points*
at those files. This skill never touches production code — it produces a reviewed decision
and spec, or an unresolved plan with no issue.

Order is fixed and enforced: **the reviewed files are available on `origin/main` before an issue
is labeled Ready.** `build` branches from that ref; a local commit alone does not make its spec
available to the next session.

## Files

- **ADR** — `docs/decisions/<NNNN>-<slug>.md`: the decision and why, alternatives considered,
  consequences. `<NNNN>` = one past the highest existing number in `docs/decisions/` (`0001` if
  none exist).
- **Spec** — `docs/specs/<slug>.md`: goal, acceptance criteria (a checklist, each item concrete
  and testable), scope-out (what this deliberately does not cover).

## Steps

1. **Learn the developer's context once.** Use what the user already said. Otherwise, ask how
   familiar they are with this project's language/platform and which languages or frameworks
   they prefer for architectural analogies. If they want this remembered, save those facts and
   explanation preferences in the local personal file `$CODEX_HOME/developer-context.md` (or
   `~/.codex/developer-context.md` when `CODEX_HOME` is unset); read it on later planning runs.
   An absent profile or unanswered question does not stop planning. The profile changes how
   findings are explained, never the technical standard used to review the plan.

2. **Ground and draft.** Read the user's goal, relevant code and existing docs. Docs may be
   missing or written late; use the code and verified platform constraints as the evidence
   available now. Treat the user's rough implementation idea as a proposal unless they state
   it as a requirement. Draft both files from that evidence. Write them to disk but do **not**
   commit yet.

3. **Independent review.** Dispatch a fresh sub-agent (`Agent` tool, no `fork` — it must
   NOT inherit this conversation). Give it the user's feature goal and constraints, with
   tentative implementation ideas marked as tentative, plus the two file paths. Do not supply
   the drafter's preferred verdict. Tell it to read the files and relevant code itself:

   > Check whether the decision fits the user's goal and the codebase, the spec can be built
   > without inventing behavior, and each acceptance criterion is checkable. Verify any claimed
   > code or platform constraint before using it to object. For each finding, state the affected
   > goal or plan clause, the observed code/constraint and why it matters, and a concrete repair.
   > Separate build-blocking defects from optional improvements and questions. A new idea or
   > unrequested feature is not a blocker merely because the plan omits it. Return READY,
   > REVISE (repairable by the planner), or USER DECISION (a real product trade-off or a blocker
   > that cannot be resolved from evidence); explain the smallest change needed to proceed.

4. **Resolve the review.** Check each finding against the code and the user's stated scope.
   For REVISE, repair the ADR/spec yourself and request one focused re-review of the changed
   points; do not make the user interpret raw objections or restart a broad review. After two
   review rounds, stop the cycle and present any remaining blocker with evidence and a
   recommended path. For USER DECISION, explain the options and recommend one before asking.
   For every material finding shown to the user, explain unfamiliar terms in ordinary language;
   when useful, add a brief analogy from their familiar language/framework and say where the
   analogy stops. Name what the cited file or symbol does, rather than giving its name alone.
   Preserve the reviewer's underlying concern, but do not forward its raw wording as the only
   explanation. If the user explicitly overrules a validated blocker, append a
   `## Verifier objections (overruled by user)` section to the spec with the objection and the
   user's decision. Do not turn optional suggestions into scope without the user's direction.

5. **On READY, or after the resolved review** — commit the reviewed files:
   ```
   git add docs/decisions/<n>.md docs/specs/<slug>.md
   git commit -m "Add ADR <NNNN> and spec for <slug>"
   git status --porcelain docs/decisions docs/specs
   ```
   Publish them through the repository's normal push or PR/merge flow. If publication awaits
   review, keep the plan pending and do not create or label an issue Ready yet. Once published,
   fetch `origin/main` and confirm both paths can be read from it before creating the issue:
   ```
   git fetch origin main
   git show "origin/main:docs/decisions/<n>.md"
   git show "origin/main:docs/specs/<slug>.md"
   gh issue create --title "<short specific title>" --label ready --body "<see format below>"
   ```

6. **Never build.** If asked to also implement it, decline — hand off to `build` instead. This
   skill's only output is two committed files and one issue (or an unresolved plan with no issue).

## Issue body format

The issue is a pointer, not a duplicate of the files:

```
## Spec docs touched
- docs/decisions/<NNNN>-<slug>.md
- docs/specs/<slug>.md

## Summary
<one or two sentences — what this is, for the reader who won't open the files>
```

## ADR file format (`docs/decisions/<NNNN>-<slug>.md`)

```
# <NNNN>: <decision title>

## Decision
<what was decided>

## Why
<the reasoning / problem it solves>

## Alternatives considered
- <alternative> — <why not>

## Consequences
<what this commits us to>
```

## Spec file format (`docs/specs/<slug>.md`)

```
# <slug>

## Goal
<why this exists, one or two sentences>

## Acceptance criteria
- [ ] <testable criterion>
- [ ] <testable criterion>

## Scope-out
- <deliberately not covered>
```
