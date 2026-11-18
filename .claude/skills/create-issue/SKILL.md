---
name: create-issue
description: Turns an idea into a GitHub issue in the format of the template issues (value proposition, acceptance criteria, tasks). Shows the draft first and creates the issue only after approval. Use it for your own feature ideas.
argument-hint: <your idea in your own words>
disable-model-invocation: true
---

Turn this idea into a GitHub issue: $ARGUMENTS

Do not write code and do not create branches.

1. Read one story from `.github/templates/` (for example `04-gallery-page.md`) and use the same format: a short intro sentence, **Value Proposition** (As a / I want to / so that), **Acceptance Criteria** and **Tasks** as checklists, and a **Hint** only if it really helps.
2. If the idea is unclear or too big for one pull request, ask me first. One issue is one small story.
3. Write acceptance criteria that a user can see and a test can check. Only write what the idea asks for.
4. Show me the title and the full draft. The title is a short name like `Dark Mode`, without a number: GitHub numbers the issue. Do not put the title as a heading into the body.
5. Wait for my explicit yes. If I ask for changes, show the new draft and wait again.
6. After my yes, write the body to a file in `$TMPDIR` and run `gh issue create --title "<title>" --body-file <file>`.
7. Give me the URL and the number of the new issue.
