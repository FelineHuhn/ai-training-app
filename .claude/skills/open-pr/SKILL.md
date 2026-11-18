---
name: open-pr
description: Runs all checks, commits the work in small steps, pushes the branch and opens a pull request for an issue. Use it when an issue is implemented and you checked it yourself.
argument-hint: <issue number>
disable-model-invocation: true
---

Open a pull request for issue #$ARGUMENTS.

1. **Branch:** run `git branch --show-current`. If it is `main`, stop and tell me.
2. **Checks:** run all checks from the **Commands** section of `CLAUDE.md`. If one fails, stop and report the error. Do not open a pull request, and do not change tests, thresholds or rules to make a check green.
3. **Commit:** look at `git status` and `git diff`. Commit in small steps that each have one purpose (for example one acceptance criterion with its test), with a clear message in the imperative ("Add gallery list"). Add files by name, never with `git add -A` or `git add .`. Never commit `.env` files or secrets.
4. **Push:** `git push -u origin HEAD`.
5. **Pull request:** read the issue with `gh issue view $ARGUMENTS`. Write the body to a file in `$TMPDIR` and create the pull request with `gh pr create --base main --title "<issue title>" --body-file <file>`. Use this body:

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

- Tick a criterion only if a test or a check that you ran in this conversation covers it. Leave all others unticked.
- Under "How I tested", list a manual check in the browser only if I told you that I did it. Ask me if you do not know.

At the end, give me the URL of the pull request.
