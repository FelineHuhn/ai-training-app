---
name: open-pr
description: Runs all checks, commits the work in small steps, pushes the branch and opens a pull request for an issue. Use it when an issue is implemented and you checked it yourself.
argument-hint: <issue number>
disable-model-invocation: true
---

Open a pull request for an issue.

`<n>` is the issue number from the argument `$ARGUMENTS`, without `#`. If the argument is missing or not a number, stop and ask me.

1. **Branch:** run `git branch --show-current`. If it is `main` or does not start with `feature/<n>-`, stop and tell me.
2. **Read:** the issue with `gh issue view <n>` and the plan file `docs/plans/<n>-*.md`. The table in the plan says which test covers which acceptance criterion.
3. **Checks:** if `CLAUDE.md` has no **Commands** section, stop and tell me. Run all checks from that section. If one fails, stop and report the error. Do not open a pull request, and do not change tests, thresholds or rules to make a check green.
4. **Commit:** look at `git status` and `git diff`. Group the changes by purpose (for example one acceptance criterion with its test) and commit each group with a clear message in the imperative ("Add gallery list"). If a file belongs to several criteria, commit it once with a message that names them. Add files by name, never with `git add -A` or `git add .`. Never commit `.env` files or secrets.
5. **Before the push:** run `git status`. If anything that belongs to the issue is still uncommitted or untracked, stop and tell me.
6. **Push:** `git push -u origin HEAD`.
7. **Pull request:** write the body to a file in `$TMPDIR` and create the pull request with `gh pr create --base main --title "<issue title>" --body-file <file>`. Use this body:

```markdown
Closes #<n>

## Summary

<what changed and why, 2 to 4 bullet points>

## How I tested

<only commands you ran in this conversation, with their result>

## Acceptance criteria

- [x] <criterion covered by a passing test or check>
- [ ] <criterion not checked yet, and why>
```

## Claim nothing you did not check

- Tick a criterion only if its test from the plan, or a check that you ran in this conversation, passed. Leave all others unticked.
- Under "How I tested", list a manual check in the browser only if I told you that I did it. Ask me if you do not know.

At the end, give me the URL of the pull request. Remind me to use one review tool for it: the reviewer sub-agent or `/code-review`, not both.
