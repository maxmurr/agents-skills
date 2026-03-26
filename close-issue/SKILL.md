---
name: close-issue
description: Close a completed Linear issue after merge. Marks the issue as Done, posts a completion comment, and suggests running /index-knowledge. Use when a PR has been merged and the user wants to close the issue, mark it done, or wrap up a ticket.
---

# close-issue

## Workflow

### 1. Get the Linear issue ID

Ask the user for the Linear issue identifier if not already in context (e.g. `ENG-42`).

Optionally ask for the merged PR URL (used in the completion comment).

### 2. Verify merge (if PR URL provided)

If a PR URL was given, confirm it is merged:

```bash
gh pr view <pr-url> --json state --jq '.state'
```

If the PR is not yet merged, STOP and tell the user. Do not close the Linear issue for an unmerged PR.

### 3. Mark as Done in Linear

Use `save_issue(id, state: "Done")`.

### 4. Post completion comment

Use `save_comment(issueId, body)` on the Linear issue:

```
Shipped. <pr-url if available>
```

### 5. Report + suggest next steps

Print:

```
<issue-id> marked Done.

If you modified significant code, run /index-knowledge to update AGENTS.md.
```
