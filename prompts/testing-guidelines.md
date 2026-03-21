# Testing Guidelines

Reference for implementer agents. Describes what good tests look like and common anti-patterns to avoid.

## Core Principle

Tests exist to catch bugs before users do. A test that can't fail is worthless. A test that tests the mock instead of the code is worse than worthless — it creates false confidence.

## TDD Cycle (mandatory for all implementation)

```
1. Write ONE failing test
2. Run it — confirm it FAILS for the right reason
3. Write MINIMAL code to make it pass
4. Run it — confirm it PASSES
5. Refactor if needed (tests still pass)
6. Repeat from 1
```

**Never skip step 2.** A test that has never failed has never proven it can catch a bug.

## Anti-Patterns (avoid these)

### 1. Testing Mock Behavior Instead of Real Behavior

BAD: "verify that the mock was called with these arguments"
```
// This tests that YOUR CODE calls a function — not that the feature works
expect(mockDb.save).toHaveBeenCalledWith({ name: "test" });
```

GOOD: "verify that the system produces the right output"
```
// This tests actual behavior
const result = await createUser({ name: "test" });
expect(result.id).toBeDefined();
expect(await db.users.findById(result.id)).toMatchObject({ name: "test" });
```

### 2. Adding Test-Only Methods to Production Code

BAD: Adding `getInternalState()` or `_testHelper()` to a class so tests can inspect internals.

GOOD: Test through the public interface. If you can't test behavior through the public API, the design may need to change.

### 3. Tautological Tests

BAD: Tests that pass regardless of implementation:
```
// This will pass even if formatCurrency is completely broken
const mock = jest.fn().mockReturnValue("$100.00");
expect(mock("100")).toBe("$100.00");
```

GOOD: Tests that exercise real code:
```
expect(formatCurrency(100, "USD")).toBe("$100.00");
expect(formatCurrency(100, "EUR")).toBe("100.00 EUR");  // different format
expect(formatCurrency(0, "USD")).toBe("$0.00");          // edge case
```

### 4. Over-Mocking

BAD: Mocking everything including internal modules:
```
jest.mock("./utils");
jest.mock("./validator");
jest.mock("./formatter");
// Now you're testing that your code calls functions, not that it works
```

GOOD: Use real dependencies for internal modules, mock only at system boundaries:
```
// Real internal deps
import { validate } from "./validator";
import { format } from "./formatter";

// Mock only external services
jest.mock("./http-client");  // external API calls
```

### 5. Snapshot Abuse

BAD: Snapshotting entire HTML/JSON output — tests break on any change, devs blindly update snapshots.

GOOD: Assert specific properties that matter:
```
expect(result.status).toBe("success");
expect(result.data.items).toHaveLength(3);
expect(result.data.items[0].name).toBe("Expected Name");
```

### 6. Testing Implementation Details

BAD: Testing internal data structures, private method calls, internal state.

GOOD: Testing observable behavior — inputs produce expected outputs, side effects are visible.

## When to Use Mocks vs Real Dependencies

| Dependency Type | Approach |
|---|---|
| Internal utility functions | Real (never mock) |
| Database (with test DB available) | Real (use test DB, transaction rollback) |
| Database (no test DB) | Mock the repository layer, not the DB driver |
| External HTTP APIs | Mock (use recorded responses) |
| File system | Real (use temp directories, clean up after) |
| Time/dates | Inject clock, don't mock Date globally |
| Random values | Inject seed or generator |
| Environment variables | Set in test setup, restore in teardown |

## Test Structure

Every test should be readable as a specification:

```
describe("TransactionImport", () => {
  describe("when importing a valid bank statement", () => {
    it("creates transactions with correct amounts and dates", async () => {
      // Arrange — set up test data
      // Act — call the function under test
      // Assert — verify the outcome
    });

    it("skips duplicate transactions based on hash", async () => { ... });
    it("categorizes transactions using rules first, AI second", async () => { ... });
  });

  describe("when the statement is encrypted", () => {
    it("stores pending file and requests password", async () => { ... });
  });

  describe("when parsing fails", () => {
    it("falls back to regex parser", async () => { ... });
    it("returns error if both parsers fail", async () => { ... });
  });
});
```

## Verification Checklist (before reporting DONE)

- [ ] Every new function has at least one test
- [ ] Tests cover happy path AND error paths
- [ ] Tests use real dependencies where practical
- [ ] No test-only methods were added to production code
- [ ] Full test suite runs green (paste output as evidence)
- [ ] Each test was seen to FAIL before passing (TDD cycle)
