---
name: atdd
description: Outside-in TDD with double-loop (acceptance + unit). Use when user wants acceptance-test-driven development, outside-in TDD, London-school TDD, GOOS-style development, or wants to drive design from user-facing behavior inward.
---

# Acceptance Test-Driven Development (Outside-In TDD)

## Philosophy

**Core principle**: We only build what the outside needs. Start at the user-facing boundary — an API endpoint, CLI command, or UI interaction — and let the outer behavior drive every design decision inward.

Two feedback loops run simultaneously: an **outer acceptance test** that defines "done" for the feature, and **inner unit tests** that drive each layer's implementation. Mocks aren't just test isolation — they're the mechanism for discovering what the next layer inward needs to do before it exists.

See [double-loop.md](double-loop.md) for the loop mechanics and [acceptance-tests.md](acceptance-tests.md) for what makes a good acceptance test.

## Anti-Pattern: Inside-Out Guessing

Don't start by building domain logic or infrastructure hoping it'll compose into the right feature. Building bottom-up means you're guessing what the outer layer will need.

The acceptance test is the anchor. Every design decision flows from it.

## Workflow

### 1. Planning

Before any code:

- [ ] Confirm what the user observes when the feature works (the acceptance criterion)
- [ ] Identify the system entry point (API route, command handler, event listener)
- [ ] Sketch the acceptance test scenario in Given/When/Then form
- [ ] Identify the layers you'll traverse (e.g., handler → service → repository)
- [ ] Get user approval on the scenario and layer map

Ask: "What does success look like from outside the system?"

### 2. Walking Skeleton (if new architecture)

If this is a greenfield project or a new architectural slice:

- [ ] Write ONE acceptance test through the full stack
- [ ] Implement the thinnest path that connects all layers — even if trivially
- [ ] Goal: prove the architecture works before building real features

See [double-loop.md](double-loop.md) for details.

### 3. Double Loop

**Outer RED**: Write a failing acceptance test for the feature.

**Inner loop** — repeat for each layer, working outside → in:

```
RED:   Write unit test for current layer object
       Mock the next layer's collaborator (this discovers its interface)
GREEN: Implement just enough to pass
       Move inward: the mocked collaborator becomes the next object to implement
```

See [interface-discovery.md](interface-discovery.md) for how mocking discovers interfaces.
See [collaboration-tests.md](collaboration-tests.md) for how to write unit tests at each layer.

**Outer GREEN**: When all layers are implemented and connected, the acceptance test passes.

### 4. Refactor

After outer GREEN:

- [ ] Extract clean interfaces from the contracts your mocks revealed
- [ ] Replace in-memory fakes with real implementations where needed
- [ ] Write contract tests to verify real implementations match mock contracts
- [ ] Apply standard refactoring (duplication, long methods, etc.)

See [refactoring.md](refactoring.md) for candidates.

## Checklist Per Cycle

```
[ ] Acceptance test describes what the user observes — not internal behavior
[ ] Each unit test mocks only the immediate next-layer collaborator
[ ] Mock expectations are the specification for the collaborator's interface
[ ] No layer exists without a test that drove it
[ ] Acceptance test turns GREEN only when all layers are real (no mocks left in the path)
```
