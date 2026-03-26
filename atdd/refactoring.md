# Refactor Candidates

Refactor only after the outer acceptance test is GREEN. Never refactor while RED.

## Outside-In Specific

**Extract interface from discovered contracts**

When your mock had an implicit contract, make it explicit:

```typescript
// Before: implicit (only in mock setup)
const mockRepo = { save: jest.fn(), findById: jest.fn() };

// After: explicit interface
interface OrderRepository {
  save(order: NewOrder): Promise<{ id: string }>;
  findById(id: string): Promise<Order | null>;
}
```

**Consolidate duplicate roles**

If two layers mock collaborators with overlapping responsibilities, they may be the same role. Merge into one interface.

**Replace in-memory fakes with real adapters**

Once the interface is stable and contract tests pass, swap the in-memory implementation for the real adapter. The contract tests verify the swap is safe.

**Introduce a layer if responsibilities blurred**

If your service is doing too much (validation + orchestration + transformation), extract a domain object. Drive its interface with a unit test first.

## Standard Refactoring

- **Duplication** → Extract function/class
- **Long methods** → Break into private helpers (keep tests on public interface)
- **Deep nesting** → Flatten with early returns or extracted functions
- **Primitive obsession** → Introduce value objects (e.g., `Money`, `OrderId`)
- **Feature envy** → Move logic to where the data lives

## Architecture Alignment

After refactoring, you may notice your layer boundaries naturally form a port-and-adapter pattern:

- Your service interfaces (discovered via mocking) are the **ports**
- Your concrete implementations (PostgresRepo, StripeClient) are the **adapters**

If this alignment is natural in your codebase, lean into it. If not, don't force it — the right boundary is the one your tests revealed.
