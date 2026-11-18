---
name: plan-feature
description: Plans one GitHub issue in Plan mode, then creates the feature branch and commits the approved plan as a file. Use it at the start of every issue. Writes no application code.
argument-hint: <issue number>
disable-model-invocation: true
---

Plan issue #$ARGUMENTS. Do not write, change or delete application code, tests or configuration.

## Before you plan

1. If Plan mode is not active, stop and tell me to switch to Plan mode (Shift+Tab) and start the skill again.
2. Run `git branch --show-current` and `git status`. If the branch is not `main` or there are uncommitted changes, stop and tell me.
3. Read the issue: `gh issue view $ARGUMENTS`.
4. Read `CLAUDE.md` and explore only the code that matters for this issue.

## The plan

Present a plan with exactly these parts:

1. **Goal:** one or two sentences.
2. **Acceptance criteria and tests:** a table. One row per acceptance criterion: the criterion, the test that covers it (file and test name) and what the test checks. If the issue is a setup issue without a feature to test, write for each criterion how it is checked instead (a command or a manual step).
3. **Files:** every file that is created or changed, with one line on why. Mark every change to an **existing test** and say why it is needed.
4. **Out of scope:** what this issue does not ask for and what you will not do.
5. **Open questions:** anything unclear in the issue. Write "None" if there is nothing.

Only plan what the issue asks for. No extra features, no refactoring of unrelated code, no new dependencies unless the issue asks for them.

## After I approved the plan

Do exactly these three things, then stop:

1. Create the branch `feature/<n>-<slug>`. `<slug>` is the issue title in kebab-case, without the number prefix: issue `4: Gallery Page` gives `feature/4-gallery-page`.
2. Save the approved plan, including my changes, as `docs/plans/<n>-<slug>.md`.
3. Commit only this file: `git add docs/plans/<n>-<slug>.md`, message `Add plan for #<n>`.

Do not push and do not start the implementation. End with the next command for me: `/tdd-feature docs/plans/<n>-<slug>.md`.
