# The Double Loop

## Structure

```
┌─────────────────────────────────────────────────────────┐
│  OUTER LOOP (Acceptance Test)                           │
│                                                         │
│  RED ──────────────────────────────────────── GREEN     │
│  Write failing   ┌───────────────────────┐   Feature   │
│  acceptance test │  INNER LOOP (Unit)    │   complete  │
│                  │                       │             │
│                  │  RED → GREEN → REFACTOR│             │
│                  │  RED → GREEN → REFACTOR│             │
│                  │  RED → GREEN → REFACTOR│             │
│                  │  (once per layer)     │             │
│                  └───────────────────────┘             │
└─────────────────────────────────────────────────────────┘
```

**Outer loop** (hours): The acceptance test stays RED while you cycle through the inner loop for each layer. It turns GREEN only when all layers are implemented and connected.

**Inner loop** (minutes): One RED→GREEN→REFACTOR cycle per layer object. Each cycle uses mocks to defer the next layer inward, then the next cycle implements that layer.

The outer test is your compass — it tells you when you're done. The inner tests are your engine — they drive each step forward.

## Walking Skeleton

A walking skeleton is the thinnest end-to-end implementation of the system. Its purpose is not to deliver functionality — it's to prove the architecture works.

**When to build one**:
- Starting a new project
- Adding a new architectural slice (e.g., first background job, first event handler)
- When you're unsure how layers connect end-to-end

**What it contains**:
- A real (but trivial) acceptance test
- All layers wired together: entry point → application logic → data store or external system
- Build + deploy pipeline working (CI runs the acceptance test)
- No real business logic — just "hello world" through the full stack

**How it differs from a real feature**: The skeleton focuses on infrastructure and connectivity. A real feature uses the skeleton as the base and adds the behavior that the acceptance test actually verifies.

Build the skeleton once. All subsequent features add slices through existing layers.

## Loop Mechanics in Practice

```typescript
// Outer RED: write this first, let it fail
test("user can place an order", async () => {
  const response = await api.post("/orders", { productId: "123", quantity: 2 });
  expect(response.status).toBe(201);
  expect(response.body.orderId).toBeDefined();
});

// This drives the inner loop:
// 1. Implement OrderController (mock OrderService)
// 2. Implement OrderService (mock OrderRepository)
// 3. Implement OrderRepository (real DB)
// → Outer test turns GREEN
```

**Key rule**: The acceptance test uses real infrastructure (real HTTP, real database) or high-fidelity fakes at the system boundary. Inner unit tests use mocks. Never mix them — acceptance tests must not mock internal layers.
