---
name: loop
description: "Execute a Ready GitHub issue after a narrow pre-build check, build via an Engineer sub-agent, independently verify code and device checks, then PR/merge. Route design gaps back to planning without spending build retries."
---

# loop

Execution only. Never makes a design decision — if an issue is ambiguous, block it and move on;
that gap gets fixed by the `planning` skill, not by improvising here.

Read the optional personal `$CODEX_HOME/developer-context.md` (or
`~/.codex/developer-context.md` when unset). Use it only to explain unfamiliar platform
concepts and QA findings in terms the developer knows. It never changes the spec or pass
criteria; do not ask for proficiency again in loop.

## Steps

1. **Find work**: `gh issue list --label ready --state open --json number,title,body,labels`.
   Skip anything already labeled `in-progress` or `blocked`, or with an unresolved dependency.
   Pick the lowest issue number among the remaining issues.
   No ready issue → report idle and stop (don't spin).

2. **Preflight before changing GitHub state**: fetch `origin/main`, then read each declared
   spec path from that fetched commit (`git show "origin/main:<path>"`). The local working tree
   alone does not prove the next task branch can see a spec. Check the relevant code and every
   acceptance criterion: can its result be observed by an automated check or a named human/device
   check? Re-check any premise the implementation depends on. A choice of implementation detail
   is not a design gap when the stated behavior has one clear reading. A missing spec, false
   premise, or criterion whose pass/fail still requires a product decision goes back to planning:
   comment with the specific evidence, add `blocked`, remove `ready`, and skip without building.
   If the fetch fails, stop before any issue or branch mutation.

3. **Pick it** only after preflight passes:
   ```
   git checkout -b task/<n>-<slug> origin/main
   gh issue edit <n> --add-label in-progress --remove-label ready
   ```
   If the branch already exists, inspect and resume its work rather than creating another one.
   The spec file's acceptance criteria are the contract; the issue body points to them.

4. **Build** — dispatch a fresh Engineer sub-agent (`Agent` tool) with the spec file's contents
   (not this conversation, not a paraphrase). Instruct it: implement this, test-first — write
   the acceptance test(s) first, confirm they fail, then implement until they pass. Commit the
   test and implementation as work lands. Report the changed files, commits, commands and exit
   codes, including the red-then-green transition. The loop session pushes the task branch after
   the Engineer report (`git push -u origin task/<n>-<slug>`) so a stopped session can recover
   its landed commits.

5. **Verify independently** — dispatch a second fresh sub-agent as QA, given the spec file's
   contents plus the branch diff (`git diff origin/main...HEAD`). It must NOT be told the Engineer's
   self-report as fact — it re-derives everything itself:
   - Re-run the automated checks and read their exit codes; don't take "tests pass" on faith.
   - Walk every acceptance criterion against the code and tests, one by one. Check that docs
     describe the behavior actually shipped in this PR.
   - For a check needing an iPhone or human judgment, record who performed it and the result.
     Defer it only to a named review step or tracked follow-up the spec or user allows. An unmet,
     undeferred check is not PASS.
   - Report PASS or FAIL with the observations. Classify each FAIL as implementation defect,
     spec gap, or unavailable test environment. A red caused by missing tooling or device access
     is not proof that the implementation failed.

6. **On FAIL** — re-dispatch the Engineer only for an implementation defect, with the QA evidence
   attached. Max 2 retries (3 build attempts total). A spec gap returns to planning immediately;
   an unavailable test environment stops this session and reports the missing prerequisite
   without counting it as a build failure. For a spec gap or a defect still failing after retries:
   ```
   gh issue comment <n> --body "<failed criterion, observed evidence, and needed plan or code change>"
   gh issue edit <n> --add-label blocked --remove-label in-progress
   ```
   Explain the blocker in ordinary language; name what cited files or symbols do and, when
   useful, give a brief analogy from the developer's familiar framework. Do not forward the raw
   QA verdict as the only explanation. Then move on to the next Ready issue.

7. **On PASS**: push the committed task branch (`git push -u origin task/<n>-<slug>`), then open
   a PR whose body carries a short summary, the automated results, and each human/device check's
   result or deferral pointer:
   ```
   gh pr create --title "<title> (#<n>)" --body "Closes #<n>

   <summary, QA evidence, and human/device check disposition>"
   ```
   Merge it (`gh pr merge --squash`) if the user said this session runs unattended; otherwise
   leave it open for the user to merge — ask once at session start which applies, don't ask
   per-issue. An unattended merge waits while a required human/device check is neither done nor
   explicitly deferred to a tracked follow-up allowed by the spec or user.

8. **Repeat** from step 1 for the next Ready issue until none remain or the user stops the
   session.
