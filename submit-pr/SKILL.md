---
name: submit-pr
description: Submit implementation as a pull request. Runs tests, creates a PR with the Linear issue linked, and posts the PR URL as a comment on the Linear issue. Use when implementation is complete and user wants to open a PR, submit work, or create a pull request.
---

# submit-pr

## Workflow

### 1. Get the Linear issue ID

Ask the user for the Linear issue identifier if not already in context (e.g. `ENG-42`).

Fetch the issue: `get_issue(id)` to get the title, description, and acceptance criteria.

### 2. Run tests

Run the project's test command (check `package.json`, `Makefile`, or ask the user).

If tests fail: STOP. Report which tests failed and ask the user to fix them before proceeding. Do not create the PR with a failing build.

### 3. Push the branch

```bash
git push -u origin <current-branch>
```

### 4. Create the PR

Use `gh pr create` with:

- **Title**: the Linear issue title
- **Body**: use the template below
- **Base**: `main` (or ask if the project uses a different default branch)

```
## Linear issue

<linear-issue-url>

## What changed

<1-3 bullet summary of what was implemented — derived from the issue's "What to build" and acceptance criteria>

## Acceptance criteria

<paste the checklist from the Linear issue body>

## Test plan

<brief description of what the tests cover>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### 5. Post PR link to Linear

Use `save_comment(issueId, body)` on the Linear issue:

```
PR ready for review: <pr-url>
```

### 6. Report

Print:
```
PR created: <pr-url>
Linear comment posted on <issue-id>
```
