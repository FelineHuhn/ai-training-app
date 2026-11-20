# Handoff for the next agent session

Last updated: 2026-11-20. Read this first, then `docs/superpowers/specs/2026-11-18-family-organizer-design.md`.

Talk to the owner in **German**. Code, tests, commits, issues and docs are in **English**.

## What this project is now

- The repository started as Recap Project 6 (Art Gallery App with a coding agent). The owner changed the app to a **family organizer**: recipes, a shopping list and to-do lists for two parents on their phones.
- The approved design is `docs/superpowers/specs/2026-11-18-family-organizer-design.md`.
- Stack: Vite + React + TypeScript (template), plus an Express API in `server/` with Mongoose and MongoDB Atlas, deployed on Vercel. Data is protected by a long family code, not a login.
- The **course rules still apply**:
  - the agent writes all code;
  - one issue = one branch = one pull request;
  - only what the issue asks for;
  - no new dependencies without approval;
  - never change a test to make it pass.

## Done

- Security baseline: `.claude/settings.json`, `docs/agent-security.md` and `.gitignore` rules. Merged in PR #14.
- Skills in `.claude/skills/`, started by the owner only:
  - `/plan-feature <n>`
  - `/tdd-feature <plan file>`
  - `/open-pr <n>`
  - `/create-issue <idea>`

  The reviewer sub-agent is `.claude/agents/reviewer.md`, which reviews `git diff main...HEAD`. Merged in PR #14.
- Design document: PR #15 (branch `docs/family-organizer-design`). It was **open** when this file was written.
- Issue **#16 "README and App Shell"** (story 1 of the design) is created.
- The art gallery issues 4 to 13 were deleted by the owner.

## Next, in this order

1. Merge PR #15 if it is still open.
2. **Setup issues 1, 2, 3** (still open, done before any story): CLAUDE.md and tooling, CI, SonarQube Cloud and branch protection.
   - The skills `/tdd-feature` and `/open-pr` stop on purpose until `CLAUDE.md` has a **Commands** section, which issue 1 adds.
   - Setup issues are implemented without `/tdd-feature`.
3. Issue #16, then create stories 2 to 12 from the design with `/create-issue`, one at a time.

The loop for every issue:

1. Clean `main`, `/clear`, Plan mode.
2. `/plan-feature <n>`
3. `/tdd-feature docs/plans/<n>-<slug>.md`
4. The owner checks the app in the browser.
5. `/open-pr <n>`
6. One review: the reviewer sub-agent **or** `/code-review`, not both.
7. Green CI, then merge.

## Gotchas in this environment

- **`gh` fails inside the Bash sandbox** with `x509: OSStatus -26276`, and `gh auth status` then wrongly says the token is invalid. The login is fine. Run `gh` outside the sandbox. The owner approves each call on purpose and decided not to exclude `gh` from the sandbox.
- **The sandbox cannot write or delete files in `.claude/`.**
  - `git switch` between branches that add or remove `.claude/` files fails halfway and leaves them untracked.
  - Check that the files equal `origin/main`, then repair it outside the sandbox.
  - Use the Write and Edit tools for files in `.claude/`, not shell commands.
- **`git push -u` cannot write `.git/config` inside the sandbox.** The push itself works. Set the upstream afterwards outside the sandbox: `git branch --set-upstream-to=origin/<branch>`.
- Never run the tests in watch mode. Use the single-run command from the Commands section of `CLAUDE.md` once it exists.
- The agent cannot read `.env` (deny rule). The owner sets `MONGODB_URI` and `FAMILY_CODE` in `.env` and in Vercel; never ask for the values.
