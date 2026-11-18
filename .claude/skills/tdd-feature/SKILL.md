---
name: tdd-feature
description: Implements an approved plan test-first, one acceptance criterion at a time, and runs all checks at the end. Use it after /plan-feature, with the plan file. Does nothing that is not in the plan.
argument-hint: <plan file, e.g. docs/plans/4-gallery-page.md>
disable-model-invocation: true
---

Implement the approved plan in `$ARGUMENTS`, test-first.

## Before you start

1. Read the plan file and `CLAUDE.md`.
2. If `CLAUDE.md` has no **Commands** section, stop and tell me. Take every command from that section. Never run the tests in watch mode: use the command for a single test run.
3. If the current branch is `main`, stop and tell me.

## For each acceptance criterion, in the order of the plan

1. **Red:** write the test from the plan. Run only this test file, once.
2. **Check the reason:** the test must fail because the feature is missing (an assertion fails, an element is not found). If it fails because of a typo, a wrong import or a syntax error, fix the test and run it again. If it passes right away, stop and tell me: the test does not check the new behavior. Tell me the failure message in one line.
3. **Green:** write the minimal code that makes this test pass. Nothing more.
4. Run the test again until it passes.
5. **Clean up:** improve names and structure of the new code. Run the test again: it must stay green.

## Rules

- Never change an existing test unless the approved plan says so. If an existing test fails, change the code, not the test.
- Do nothing that is not in the plan: no extra features, no refactoring of other code, no new dependencies.
- If you need something the plan does not cover, stop and ask me.
- Never lower a threshold, switch off a rule or skip a test to make a check green.
- Do not commit. `/open-pr` does that.

## At the end

Run all checks from the **Commands** section of `CLAUDE.md`. Then report:

- for each acceptance criterion: the test, the reason it failed first, and that it passes now
- the result of each check (passed or failed, with the error)
- every file you changed

End with the next command for me: `/open-pr <n>`.
