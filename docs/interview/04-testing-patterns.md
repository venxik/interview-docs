# Testing Patterns

This guide covers testing strategies and patterns used in the project, including Jest mocking and unit testing approaches.

---

## Topics Covered

- Testing strategy with mocked database layer
- Jest testing patterns
- Repository mocking techniques
- Unit test best practices

---

### **Category 6: Testing**

**19. What is the testing strategy, and why is it beneficial?**

- **Answer:** "The strategy, as documented in the `README`, is to use **unit tests with a mocked database layer**. All repository methods are mocked using Jest.
  - **Benefits:** This makes the tests extremely fast (the docs say ~2.5 seconds for 155 tests), and they don't require any external dependencies like Docker or a running database. This means any developer can run `npm test` anywhere and get immediate feedback, which is great for CI/CD and developer productivity."

---
### **Category 14: Jest Testing Patterns in This Project**

**47. What Jest mocking techniques are used in this project and when to use each?**

- **Answer:** This project uses several Jest mocking patterns for different scenarios. Let me explain each:

**1. Module-Level Mocking with `jest.mock()`**

Used to mock entire modules before imports:

```typescript
// At the top of test file (BEFORE imports)
jest.mock("@services");
jest.mock("@shared/utils/convertCsvToJson");
jest.mock("@database/repositories");

// Then import the mocked modules
import { dataUploadService } from "@services";
import { convertCsvToJson } from "@shared/utils/convertCsvToJson";

// Cast to mocked types for better TypeScript support
const mockedService = dataUploadService as jest.Mocked<
  typeof dataUploadService
>;
const mockedConvertCsv = convertCsvToJson as jest.MockedFunction<
  typeof convertCsvToJson
>;
```

**When to use:**

- ✅ Mock external dependencies (services, repositories)
- ✅ Prevent real database calls
- ✅ Mock file system operations
- ✅ Mock third-party libraries (axios, fs, etc.)

**Why it must be at the top:**

- Jest hoists `jest.mock()` calls to the top automatically
- Must be before imports to intercept module loading
- If placed after imports, the real module is already loaded

**2. `jest.fn()` - Creating Mock Functions**

Creates a mock function you can track and configure:

```typescript
// From UploadDataController.test.ts
mockResponse = {
  status: jest.fn().mockReturnThis(),
  json: jest.fn().mockReturnThis(),
  sendStatus: jest.fn().mockReturnThis(),
};
mockNext = jest.fn();

mockTransactionInstance = {
  commit: jest.fn(),
  rollback: jest.fn(),
} as unknown as TTransaction;
```

**When to use:**

- ✅ Create mock objects (req, res, next)
- ✅ Track function calls (how many times, with what arguments)
- ✅ Create simple callbacks

**Common patterns:**

```typescript
// Basic mock
const mockFn = jest.fn();

// Mock with return value
const mockFn = jest.fn().mockReturnValue(42);

// Mock with different returns on multiple calls
mockFn.mockReturnValueOnce("first").mockReturnValueOnce("second");

// Mock async function
const mockFn = jest.fn().mockResolvedValue({ data: "test" });
```

**3. `jest.fn().mockReturnThis()` - Method Chaining**

Allows chaining method calls (common in Express):

```typescript
// Why we need this:
mockResponse = {
  status: jest.fn().mockReturnThis(), // Returns mockResponse itself
  json: jest.fn().mockReturnThis(), // Returns mockResponse itself
};

// Now this works (method chaining):
res.status(400).json({ error: "Bad Request" });
// ↓ Calls status(400) → returns res
// ↓ Calls json() on returned res → works!
```

**Without `mockReturnThis()`:**

```typescript
const mockResponse = {
  status: jest.fn(), // Returns undefined by default
  json: jest.fn(),
};

// This BREAKS:
res.status(400).json({ error: "test" });
// ↓ status() returns undefined
// ↓ Tries to call undefined.json() → TypeError!
```

**When to use:**

- ✅ Mock Express response object
- ✅ Mock builder patterns
- ✅ Mock fluent APIs (chained methods)

**4. `jest.spyOn()` - Spying on Existing Methods**

Creates a mock but preserves the original implementation:

```typescript
// From ClassRepository.test.ts
beforeEach(() => {
  repository = new ClassRepository();

  // Spy on existing methods
  jest.spyOn(repository, "findOne").mockImplementation(jest.fn());
  jest.spyOn(repository, "findAll").mockImplementation(jest.fn());
  jest.spyOn(repository, "upsert").mockImplementation(jest.fn());
});

afterEach(() => {
  jest.restoreAllMocks(); // Restore original implementations
});

// In tests:
it("should find a class by code", async () => {
  (repository.findOne as jest.Mock).mockResolvedValue(mockClass);

  const result = await repository.findByCode("P1-1");

  expect(repository.findOne).toHaveBeenCalledWith({
    where: { code: "P1-1" },
  });
});
```

**When to use:**

- ✅ Test methods that call other methods on the same class
- ✅ Verify internal method calls
- ✅ Mock specific methods while keeping others real
- ✅ Temporarily replace method behavior

**Key difference from jest.fn():**

| Feature             | `jest.fn()`                | `jest.spyOn()`                        |
| ------------------- | -------------------------- | ------------------------------------- |
| **Purpose**         | Create brand new mock      | Mock existing method                  |
| **Original code**   | Doesn't exist              | Preserved (can be restored)           |
| **Use case**        | Mock external dependencies | Mock methods on real objects          |
| **Restoration**     | N/A                        | `jest.restoreAllMocks()`              |
| **Example**         | `const mock = jest.fn()`   | `jest.spyOn(obj, 'method')`           |
| **In this project** | Mock req/res/next          | Mock repository methods in unit tests |

**5. `mockResolvedValue()` vs `mockReturnValue()` - Async vs Sync**

**`mockResolvedValue()`** - For async functions:

```typescript
// From DataUploadService.test.ts
mockedTeacherRepo.findByEmails.mockResolvedValue([
  { id: 1, email: 'teacher@test.com', name: 'Teacher' },
]);

// Equivalent to:
mockedTeacherRepo.findByEmails.mockImplementation(async () => {
  return Promise.resolve([...]);
});

// The function being tested can await it:
const teachers = await teacherRepository.findByEmails(['teacher@test.com']);
// teachers = [{ id: 1, ... }]
```

**`mockReturnValue()`** - For synchronous functions:

```typescript
// From UploadDataController.test.ts
mockedValidateCsvData.mockReturnValue([]); // No validation errors

// Equivalent to:
mockedValidateCsvData.mockImplementation(() => {
  return [];
});

// The function returns immediately:
const errors = validateCsvData(data);
// errors = []
```

**When to use which:**

| Scenario                            | Use                        | Example                            |
| ----------------------------------- | -------------------------- | ---------------------------------- |
| Function returns `Promise<T>`       | `mockResolvedValue(T)`     | Database queries, API calls        |
| Function returns `T` directly       | `mockReturnValue(T)`       | Validation functions, calculations |
| Function returns rejected `Promise` | `mockRejectedValue(error)` | Error scenarios                    |
| Multiple calls, different returns   | `mockResolvedValueOnce()`  | First call succeeds, second fails  |

**6. `mockResolvedValueOnce()` - Sequential Mock Returns**

Returns different values on successive calls:

```typescript
// From DataUploadService.test.ts
// First call (check existing entities)
mockedTeacherRepo.findByEmails.mockResolvedValueOnce([]);

// Second call (after upsert, fetch created entities)
mockedTeacherRepo.findByEmails.mockResolvedValueOnce([
  { id: 1, email: 'teacher1@test.com', name: 'Teacher One' },
]);

// In the code being tested:
const existing = await teacherRepository.findByEmails([...]);  // Returns []
await teacherRepository.bulkUpsert([...]);
const teachers = await teacherRepository.findByEmails([...]);  // Returns [{ id: 1, ... }]
```

**Why we need this:**

- Same function called multiple times in one test
- Each call needs different return value
- Simulates state changes (before/after database operations)

**Pattern comparison:**

```typescript
// Without Once - same value every time
mock.mockResolvedValue("same");
await mock(); // 'same'
await mock(); // 'same'
await mock(); // 'same'

// With Once - different values
mock
  .mockResolvedValueOnce("first")
  .mockResolvedValueOnce("second")
  .mockResolvedValue("default");
await mock(); // 'first'
await mock(); // 'second'
await mock(); // 'default'
await mock(); // 'default'
```

**7. Assertion Patterns**

**Verify function was called:**

```typescript
expect(mockFunction).toHaveBeenCalled();
expect(mockFunction).toHaveBeenCalledTimes(2);
expect(mockFunction).not.toHaveBeenCalled();
```

**Verify function was called with specific arguments:**

```typescript
// From UploadDataController.test.ts
expect(mockedConvertCsvToJson).toHaveBeenCalledWith(mockFile.path);

expect(mockedDataUploadService.processData).toHaveBeenCalledWith(
  mockCsvData,
  mockTransactionInstance,
);

// Check specific argument positions
expect(mockFn).toHaveBeenCalledWith(
  expect.any(String), // First arg can be any string
  expect.objectContaining({ id: 1 }), // Second arg must have id: 1
  expect.anything(), // Third arg can be anything
);
```

**Verify function was NOT called:**

```typescript
expect(mockedDataUploadService.processData).not.toHaveBeenCalled();
expect(mockedTransaction.rollback).not.toHaveBeenCalled();
```

**8. `beforeEach()` and `afterEach()` - Test Setup/Cleanup**

```typescript
describe("UploadDataController", () => {
  beforeEach(() => {
    jest.clearAllMocks(); // Reset all mock call counts

    mockRequest = {};
    mockResponse = {
      status: jest.fn().mockReturnThis(),
      json: jest.fn().mockReturnThis(),
    };

    mockedTransaction.beginTransaction.mockResolvedValue(
      mockTransactionInstance,
    );
  });

  afterEach(() => {
    jest.restoreAllMocks(); // Restore original implementations (for spyOn)
  });
});
```

**Why use these:**

- ✅ **`beforeEach()`**: Reset state before each test (isolation)
- ✅ **`jest.clearAllMocks()`**: Clear call history but keep mock implementations
- ✅ **`jest.restoreAllMocks()`**: Restore original functions (important for spies)
- ✅ **Test isolation**: Each test starts fresh, no side effects between tests

**9. Testing Error Scenarios**

```typescript
// From DataUploadService.test.ts
it("should re-throw an error if a repository fails", async () => {
  mockedTeacherRepo.bulkUpsert.mockRejectedValue(new Error("Database error"));

  await expect(service.processData(csvData, mockTransaction)).rejects.toThrow(
    "Database error",
  );
});

// Alternative syntax:
try {
  await service.processData(csvData, mockTransaction);
  fail("Should have thrown an error");
} catch (error) {
  expect(error.message).toBe("Database error");
}
```

**10. Type Casting Mocks for TypeScript**

```typescript
// Cast to jest.Mocked for typed mocks
const mockedService = dataUploadService as jest.Mocked<
  typeof dataUploadService
>;
// Now TypeScript knows all methods are mocked

// Cast to jest.MockedFunction for functions
const mockedFunction = convertCsvToJson as jest.MockedFunction<
  typeof convertCsvToJson
>;
// Now TypeScript knows it's a mocked function

// Cast mock response back to Express types
await handler(mockRequest as Request, mockResponse as Response, mockNext);
```

---

**Common Patterns in This Project:**

**Pattern 1: Controller Testing (UploadDataController.test.ts)**

```typescript
// 1. Mock all dependencies at module level
jest.mock("@services");
jest.mock("@database/repositories");

// 2. Create mock req/res/next
mockResponse = {
  status: jest.fn().mockReturnThis(),
  json: jest.fn().mockReturnThis(),
  sendStatus: jest.fn().mockReturnThis(),
};
mockNext = jest.fn();

// 3. Mock service responses
mockedService.method.mockResolvedValue(mockData);

// 4. Call controller handler
await handler(mockRequest as Request, mockResponse as Response, mockNext);

// 5. Assert response
expect(mockResponse.status).toHaveBeenCalledWith(200);
expect(mockResponse.json).toHaveBeenCalledWith(expectedData);
```

**Pattern 2: Service Testing (DataUploadService.test.ts)**

```typescript
// 1. Mock repository layer
jest.mock('@repositories');

// 2. Setup mock return values (simulate database)
mockedRepo.findByEmails.mockResolvedValueOnce([]); // Empty first
mockedRepo.findByEmails.mockResolvedValueOnce([...]); // Has data second time

// 3. Call service method
await service.processData(csvData, mockTransaction);

// 4. Verify repository calls
expect(mockedRepo.bulkUpsert).toHaveBeenCalledWith(expectedData, mockTransaction);
expect(mockedRepo.findOrCreate).toHaveBeenCalledTimes(1);
```

**Pattern 3: Repository Testing (ClassRepository.test.ts)**

```typescript
// 1. Mock the underlying Sequelize model
jest.mock("@models");

// 2. Spy on base methods
jest.spyOn(repository, "findOne").mockImplementation(jest.fn());

// 3. Mock the spy's return value
(repository.findOne as jest.Mock).mockResolvedValue(mockData);

// 4. Call repository method
const result = await repository.findByCode("P1-1");

// 5. Verify base method was called correctly
expect(repository.findOne).toHaveBeenCalledWith({ where: { code: "P1-1" } });
```

---

**Interview Questions You Might Face:**

**Q: "Why use `jest.mock()` instead of `jest.spyOn()`?"**

**A:** "They serve different purposes:

- `jest.mock()` replaces an entire module before it's imported. Use it for external dependencies.
- `jest.spyOn()` wraps an existing method. Use it to track calls to real object methods.

In this project:

- We use `jest.mock('@repositories')` to prevent real database calls
- We use `jest.spyOn(repository, 'findOne')` to test methods that call other methods"

**Q: "What's the difference between `mockResolvedValue` and `mockReturnValue`?"**

**A:** "`mockResolvedValue` returns a Promise (for async functions), `mockReturnValue` returns a value directly (for sync functions).

Example:

```typescript
// Async (returns Promise)
mockedRepo.findAll.mockResolvedValue([...]);
// Same as: return Promise.resolve([...])

// Sync (returns value)
mockedValidator.validate.mockReturnValue(true);
// Same as: return true
```

**Q: "Why do you need `mockReturnThis()` for response objects?"**

**A:** "Express response methods support chaining:

```typescript
res.status(400).json({ error: "Bad Request" });
```

Without `mockReturnThis()`, `status()` returns `undefined`, so calling `.json()` crashes. With `mockReturnThis()`, each method returns the response object, enabling chaining."

**Q: "When would you use `mockResolvedValueOnce` versus `mockResolvedValue`?"**

**A:** "Use `Once` when the same function is called multiple times with different expected results:

```typescript
// Without Once - always returns same value
mock.mockResolvedValue("always this");

// With Once - returns different values in sequence
mock
  .mockResolvedValueOnce([]) // First call: empty array
  .mockResolvedValueOnce([data]); // Second call: has data

// Simulates: 'not found' → 'inserted' → 'now found'
```

This is common in our DataUploadService where we check for existing entities, then fetch them again after upserting."

**Q: "Why use `jest.clearAllMocks()` in `beforeEach()`?"**

**A:** "To ensure test isolation. Without it, mock call counts persist between tests:

```typescript
// Test 1
expect(mockFn).toHaveBeenCalledTimes(1); // ✅ Pass

// Test 2 (without clearAllMocks)
expect(mockFn).toHaveBeenCalledTimes(1); // ❌ Fail - count is 2!

// Test 2 (with clearAllMocks)
expect(mockFn).toHaveBeenCalledTimes(1); // ✅ Pass - count reset
```

---

**Summary: Jest Patterns in This Project**

1. ✅ **`jest.mock()`** - Mock entire modules (services, repositories)
2. ✅ **`jest.fn()`** - Create mock functions
3. ✅ **`mockReturnThis()`** - Enable method chaining
4. ✅ **`jest.spyOn()`** - Spy on real object methods
5. ✅ **`mockResolvedValue()`** - Mock async returns
6. ✅ **`mockReturnValue()`** - Mock sync returns
7. ✅ **`mockResolvedValueOnce()`** - Sequential different returns
8. ✅ **`beforeEach()`** - Reset mocks before each test
9. ✅ **Type casting** - `as jest.Mocked<>` for TypeScript
10. ✅ **Assertions** - `toHaveBeenCalled`, `toHaveBeenCalledWith`

**These patterns enable fast, isolated unit tests without touching the database - the core testing strategy of this project.**

---

### **Quick Tips for Your Interview**

1. **Show you understand the flow:** Always explain request → controller → service → repository → database
2. **Mention trade-offs:** Every choice has pros/cons (e.g., ORM vs raw SQL)
3. **Connect to your code:** Reference actual files ("`In ClassController.ts line 15...`")
4. **Admit what you don't know:** Better to say "I'm not sure but I would research..." than make something up
5. **Ask clarifying questions:** Shows you think critically

**Practice Exercise:** Pick any API endpoint in this project and explain its complete flow from HTTP request to database and back. Use this format:

1. HTTP Request arrives (method, path, body)
2. Express routes it to... (which file, which function)
3. Controller does... (try/catch, calls service)
4. Service does... (business logic, calls repository)
5. Repository does... (database query)
6. Data flows back up the chain
7. Response sent to client

Good luck with your interview!

---
