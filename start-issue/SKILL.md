---
name: start-issue
description: Kick off implementation of a Linear issue. Fetches the issue and its parent PRD, creates a git branch using Linear's suggested branch name, and presents a consolidated context summary ready for development. Use when starting work on a Linear issue, picking up a ticket, or when user says "start issue TEAM-123", "work on TEAM-123", or "pick up TEAM-123".
---

# start-issue

## Workflow

### 1. Get the issue ID

Ask the user for the Linear issue identifier if not already provided (e.g. `ENG-42`).

### 2. Fetch issue + parent PRD (parallel)

In a single message, use the Linear MCP tools to:

1. Fetch the issue: `get_issue(id, includeRelations: true)`
2. From the `relatedTo` field, identify the parent PRD issue (the one that looks like a PRD — has a Problem Statement / User Stories structure, typically the issue that spawned this slice via `/prd-to-issues`). Fetch it with `get_issue`.

If no parent PRD exists, note it and continue.

### 3. Create the git branch

Use the `branchName` field from the Linear issue.

```bash
git fetch origin
git checkout -b <branchName>
```

If the branch already exists locally or on remote, check it out:

```bash
git checkout <branchName>
```

If there are uncommitted changes blocking checkout, STOP and ask the user how to proceed — do not stash or discard without confirmation.

### 4. Present context summary

Print a structured summary before handing off to the developer:

```
=== TEAM-123: <title> ===

Acceptance criteria:
<criteria from issue body>

User stories addressed:
<from issue body>

=== Parent PRD: TEAM-100 ===

Problem: <problem statement excerpt>

Relevant implementation decisions:
<only the decisions relevant to this slice>

=== Ready ===
Branch: <branchName>
Next: run /atdd to begin implementation.
```

Ask: "Does this context look right? Anything to add or correct before starting?"
