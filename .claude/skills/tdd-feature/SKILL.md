---
name: tdd-feature
description: Implements an approved plan test-first, one acceptance criterion at a time, and runs all checks at the end. Use it after /plan-feature, with the plan file. Does nothing that is not in the plan.
argument-hint: <plan file, e.g. docs/plans/4-gallery-page.md>
disable-model-invocation: true
---

Implement the approved plan in `$ARGUMENTS`, test-first.

## Before you start

1. If the argument is missing or the plan file does not exist, stop and ask me.
2. Read the plan file and `CLAUDE.md`. `<n>` is the number at the start of the plan file name.
3. If `CLAUDE.md` has no **Commands** section, stop and tell me. Take every command from that section. Never run the tests in watch mode: use the command for a single test run.
4. If the current branch is `main`, stop and tell me.

## For each acceptance criterion, in the order of the plan

1. **Red:** write the test from the plan. Run only this test file, once.
2. **Check the reason:** the test must fail because the feature is missing (an assertion fails, an element is not found).
   - If it fails only because the new module does not exist yet, create an empty stub (for example a component that returns `null`) and run it again.
   - If it fails because of a typo or a syntax error in the test, fix the test and run it again.
   - If it passes right away, stop and tell me: the test does not check the new behavior.
   - Tell me the failure message in one line.
3. **Green:** write the minimal code that makes this test pass. Nothing more. Do not change the new test.
4. Run the test again until it passes. If it still fails after 3 attempts, or you think the test is wrong, stop and tell me.
5. **Clean up:** improve names and structure of the new code. Do not change what the test checks. Run the test again: it must stay green.

## Rules

- Never change an existing test unless the approved plan says so. If an existing test fails, change the code, not the test.
- Do nothing that is not in the plan: no extra features, no refactoring of other code, no new dependencies.
- If you need something the plan does not cover, stop and ask me.
- Never lower a threshold, switch off a rule or skip a test to make a check green.
- Do not commit. `/open-pr` does that.

## At the end

Run all checks from the **Commands** section of `CLAUDE.md`. If a check fails, fix it only in the files of the plan, or stop and ask me. Then report:

- for each acceptance criterion: the test, the reason it failed first, and that it passes now
- the result of each check (passed or failed, with the error)
- every file you changed

Only if all checks pass, end with the next command for me: `/open-pr <n>`.
