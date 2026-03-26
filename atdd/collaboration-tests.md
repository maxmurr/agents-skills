# Collaboration Tests and Contract Tests

These two test types work as a complementary pair. Together they give confidence that layers integrate correctly without running the full acceptance test for every branch.

## Collaboration Tests

A collaboration test verifies that an object sends the right messages to its collaborator. It uses mocks.

```typescript
// OrderService collaboration test:
// Verifies that OrderService correctly delegates to OrderRepository

test("createOrder saves the order and returns its id", async () => {
  const mockRepo = {
    save: jest.fn().mockResolvedValue({ id: "ord_123" }),
  };

  const service = new OrderService(mockRepo);
  const result = await service.createOrder({ productId: "abc", quantity: 2, unitPrice: 2999 });

  expect(mockRepo.save).toHaveBeenCalledWith({
    productId: "abc",
    quantity: 2,
    total: 5998,
  });
  expect(result.orderId).toBe("ord_123");
});
```

This test answers: "Does `OrderService` call `OrderRepository.save` with the right data?"

## Contract Tests

A contract test verifies that a **real implementation** fulfills the same contract the mock promised. It runs against the real adapter.

```typescript
// OrderRepository contract test:
// Verifies that the real DB implementation matches what the mock promised

test("save persists the order and returns its id", async () => {
  const repo = new PostgresOrderRepository(testDb);

  const saved = await repo.save({
    productId: "abc",
    quantity: 2,
    total: 5998,
  });

  expect(saved.id).toMatch(/^ord_/);

  const found = await repo.findById(saved.id);
  expect(found.total).toBe(5998);
});
```

This test answers: "Does `PostgresOrderRepository` behave the same way the mock in our collaboration tests assumed?"

## Why Both?

| Test type | What it catches |
|---|---|
| Collaboration | Caller sends wrong message (wrong method, wrong args, wrong sequence) |
| Contract | Real implementation doesn't match what the mock promised |

If only collaboration tests exist: you know the caller is correct, but the real implementation might not match the mock.

If only contract tests exist: you know the real implementation works, but the caller might be using it wrong.

Both together: integration works correctly, guaranteed without running the slow acceptance test for every branch.

## Matching the Contract

The mock's shape in collaboration tests must match the real implementation tested in contract tests. Keep them in sync — if you change the `save` return type, update both.

A useful pattern: define a shared TypeScript interface, use it in both the mock and the real implementation.

```typescript
interface OrderRepository {
  save(order: NewOrder): Promise<{ id: string }>;
  findById(id: string): Promise<Order | null>;
}

// Collaboration test: mock satisfies OrderRepository
const mockRepo: OrderRepository = { save: jest.fn()... };

// Contract test: real implementation must satisfy OrderRepository
const repo: OrderRepository = new PostgresOrderRepository(testDb);
```
