# Agentic Engineering Skills

A collection of Claude Code skills for shipping software end-to-end — from problem statement to merged PR — with an AI agent at each step.

---

## The Workflow

```
/write-a-prd
      │
      ▼
[ team review + iterate ]   ← update the PRD issue before moving on
      │
      ▼
/prd-to-issues
      │
      ├── issue 1 ──┐
      ├── issue 2 ──┤
      └── issue N ──┘
                    │  (repeat per issue)
                    ▼
            /start-issue
                    │
                    ▼
               /atdd  ◄──────────────────────────────────────┐
                    │                                         │
                    ▼                                     (more issues)
            /simplify
                    │
                    ▼
            /submit-pr
                    │
                    ▼
         [ human review + merge ]
                    │
                    ▼
           /close-issue
                    │
              (after sprint)
                    ▼
        /index-knowledge
```

---

## Phase 1 — Define

### `/write-a-prd`

Turns a vague feature idea into a structured PRD stored as a Linear issue.

1. Describe the problem you want to solve
2. The skill interviews you to resolve every design branch
3. Sketches the major modules to build/modify
4. Writes the PRD and creates it as a Linear issue

**Output:** A Linear issue with Problem Statement, Solution, User Stories, Implementation Decisions, Testing Decisions, and Out of Scope.

**Before moving on:** share the Linear issue with your team. Iterate on the PRD until it's accepted. All feedback should be incorporated into the Linear issue before running `/prd-to-issues` — the breakdown reflects whatever the PRD says at that moment.

---

## Phase 2 — Break Down

### `/prd-to-issues`

Slices the PRD into independently-grabbable implementation tickets.

1. Provide the PRD's Linear issue identifier (e.g. `ENG-100`)
2. The skill proposes **vertical slice** issues — each is a thin but complete end-to-end path (schema → API → UI → tests)
3. Each issue is labeled **AFK** (can be implemented and merged without human input) or **HITL** (requires a decision or design review)
4. Dependencies are expressed via Linear's `blockedBy` relations
5. You approve the breakdown; issues are created in Linear

**Key principle:** Prefer many thin slices over few thick ones. A completed slice is demoable on its own.

---

## Phase 3 — Implement (per issue)

### `/start-issue ENG-42`

Loads everything you need to start a specific issue.

1. Fetches the issue from Linear
2. Fetches the parent PRD (via `relatedTo` links)
3. Creates a git branch using Linear's suggested branch name
4. Prints a consolidated context summary: acceptance criteria, user stories, relevant implementation decisions

Run this at the start of every issue. It replaces the manual context-loading step.

---

### `/atdd`

Outside-in, acceptance-test-driven development. The primary implementation skill.

The double loop:

```
Outer RED:  Write a failing acceptance test (what the user observes)
  Inner RED → GREEN: TDD each layer, working outside → in
    (mocks discover the next layer's interface before it exists)
Outer GREEN: All layers connected; acceptance test passes
Refactor:   Extract clean interfaces, replace fakes with real implementations
```

**Start here after `/start-issue`.** The acceptance criteria from the Linear issue is your outer RED scenario.

For simpler, non-layered work, use `/tdd` instead (plain red-green-refactor without the outer acceptance loop).

---

### `/simplify`

Reviews all changed code for over-engineering, duplication, and missed reuse opportunities, then fixes what it finds.

Run after `/atdd` turns green, before opening the PR.

---

## Phase 4 — Ship

### `/submit-pr`

Submits the completed implementation as a pull request.

1. Runs the project's test suite — blocks if any test fails
2. Pushes the branch
3. Creates a PR via `gh pr create` with the Linear issue linked and acceptance criteria in the body
4. Posts the PR URL as a comment on the Linear issue

---

### `/close-issue ENG-42`

Wraps up after the PR is merged.

1. Verifies the PR is merged (via `gh pr view`)
2. Marks the Linear issue as **Done**
3. Posts a completion comment
4. Suggests running `/index-knowledge` if significant code changed

---

## Phase 5 — Document

### `/index-knowledge`

Regenerates the `AGENTS.md` + `CLAUDE.md` knowledge base across the codebase.

Run after completing a milestone or a batch of issues. Creates/updates hierarchical documentation that future agent sessions load as context.

---

## Meta Skills

### `/write-a-skill`

Creates new skills using the correct structure. Use when you identify a repeating workflow that isn't covered by the skills above.

### `/write-a-prd` → `/prd-to-issues` → ... (recursive)

The workflow itself can be used to plan new skills.

---

## Skill Index

| Skill | Phase | What it does |
|---|---|---|
| `write-a-prd` | Define | Interview → PRD → Linear issue |
| `prd-to-issues` | Break down | PRD → vertical slice issues in Linear |
| `start-issue` | Implement | Load issue context, create git branch |
| `atdd` | Implement | Outside-in TDD with acceptance + unit loops |
| `tdd` | Implement | Plain red-green-refactor (simpler tasks) |
| `simplify` | Implement | Code quality pass on changed code |
| `submit-pr` | Ship | Test gate → PR → Linear comment |
| `close-issue` | Ship | Mark Done → completion comment |
| `index-knowledge` | Document | Regenerate AGENTS.md + CLAUDE.md |
| `write-a-skill` | Meta | Create new skills |

---

## Decision Guide

**When to use `/atdd` vs `/tdd`**

- New feature touching multiple layers (handler → service → repo) → `/atdd`
- Bug fix or isolated change to one module → `/tdd`
- Not sure → `/atdd` (the outer acceptance test keeps you honest)

**When to use HITL vs AFK issues**

- HITL: architectural decision needed, design review required, external dependency, or anything you wouldn't merge without a human seeing it first
- AFK: clearly specified, self-contained, covered by automated tests

**When to run `/index-knowledge`**

- After completing a full feature (multiple issues)
- When a new agent session will need accurate codebase context
- Not after every single issue — the overhead isn't worth it for small changes
