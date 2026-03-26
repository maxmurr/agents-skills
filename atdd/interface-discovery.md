# Interface Discovery

The central insight of outside-in TDD: **mocking a collaborator that doesn't exist yet defines its interface**.

When you write a unit test for a layer and mock the next layer inward, the mock's setup and expectations become the specification for that collaborator. You're not testing that a function was called — you're designing an interface.

## The Mechanism

```typescript
// You're implementing OrderController. You don't have OrderService yet.
// Writing this test DISCOVERS the OrderService interface:

test("POST /orders creates an order and returns its id", async () => {
  const mockOrderService = {
    createOrder: jest.fn().mockResolvedValue({ orderId: "ord_123", total: 5998 }),
  };

  const controller = new OrderController(mockOrderService);
  const response = await controller.handlePost({ productId: "abc", quantity: 2 });

  expect(mockOrderService.createOrder).toHaveBeenCalledWith({
    productId: "abc",
    quantity: 2,
  });
  expect(response.status).toBe(201);
  expect(response.body.orderId).toBe("ord_123");
});
```

After writing this test, you know exactly what `OrderService` needs:
- A `createOrder(params)` method
- That accepts `{ productId, quantity }`
- That returns `{ orderId, total }`

This is "Mock Roles, Not Objects" — you're mocking the *role* `OrderService` plays, not a concrete class. The interface emerges from what the controller actually needs.

## Working Inward

Each inner-loop cycle implements one layer and mocks the next:

```
OrderController tests    →  mocks OrderService         (discovers OrderService interface)
OrderService tests       →  mocks OrderRepository      (discovers OrderRepository interface)
OrderRepository tests    →  uses real DB / in-memory   (leaf node — no more mocking)
```

At each step, the mock's API becomes the spec for the next layer to implement.

## When to Stop Mocking

Stop mocking when you reach a **leaf node** — an object with no meaningful collaborators:

- Pure domain logic (calculations, validations, transformations)
- Value objects and data structures
- The outermost infrastructure adapter (database driver, HTTP client)

At leaf nodes, use real objects. Don't mock a plain `calculateTotal(items)` function.

## Common Mistake: Over-Mocking

```typescript
// BAD: Mocking a pure domain object adds no value
const mockCart = {
  getTotal: jest.fn().mockReturnValue(100),
  getItems: jest.fn().mockReturnValue([]),
};

// GOOD: Use the real Cart — it has no external dependencies
const cart = new Cart([{ price: 100, quantity: 1 }]);
expect(cart.getTotal()).toBe(100);
```

Mock at layer boundaries, not within a layer. If two objects collaborate purely within the same layer (same conceptual level), prefer using both real objects together.
