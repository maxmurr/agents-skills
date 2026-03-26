# Acceptance Tests

## Good Acceptance Tests

**Test through the system's external interface. Describe what the user observes.**

```typescript
// GOOD: Tests through real HTTP, describes user-facing behavior
test("user can place an order", async () => {
  // Given: a user with items in their cart
  await seedProduct({ id: "abc", price: 2999, stock: 10 });

  // When: they submit an order
  const response = await api.post("/orders", {
    productId: "abc",
    quantity: 2,
  });

  // Then: they receive an order confirmation
  expect(response.status).toBe(201);
  expect(response.body.orderId).toMatch(/^ord_/);
  expect(response.body.total).toBe(5998);
});
```

Characteristics:
- Written in domain language, not implementation language
- Exercises the real entry point (HTTP, CLI, event)
- Asserts on observable output (response, side effects visible from outside)
- Survives any internal refactor

## Bad Acceptance Tests

**Too granular — testing a service method instead of a user scenario:**

```typescript
// BAD: This is a unit test, not an acceptance test
test("OrderService.create returns an order with correct total", async () => {
  const service = new OrderService(mockRepo);
  const order = await service.create({ productId: "abc", quantity: 2 });
  expect(order.total).toBe(5998);
});
```

**Leaks internal structure — broken by any rename:**

```typescript
// BAD: Tests implementation path, not behavior
test("placing order calls OrderRepository.save", async () => {
  const mockRepo = jest.fn();
  await api.post("/orders", { productId: "abc", quantity: 2 });
  expect(mockRepo.save).toHaveBeenCalled();
});
```

## Scope Guidelines

**Keep acceptance tests focused on happy paths and critical business flows.** Edge cases and error conditions belong in unit tests at the layer where the logic lives.

| Test type | Coverage focus |
|---|---|
| Acceptance | Happy path, key error states visible to user |
| Unit (per layer) | All conditions, branches, error cases for that layer |

**One acceptance test per user scenario** — not one per code path. A scenario like "user places an order" covers the whole flow; don't write separate acceptance tests for each internal branch.

## Given/When/Then Structure

Every acceptance test should be readable as a specification:

- **Given** — the starting state (seed data, pre-existing records)
- **When** — the user action (HTTP request, CLI command, event published)
- **Then** — the observable outcome (response, state change visible from outside)

The system under test in an acceptance test is the whole system, not a class.
