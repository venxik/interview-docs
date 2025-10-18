# TypeScript Fundamentals

This guide covers TypeScript concepts and patterns used in the project.

---

## Topics Covered

- TypeScript interfaces and types
- Generics and type parameters
- Type inference and type safety
- TypeScript best practices in this project

---

### **Category 13: TypeScript in This Project**

**44. What TypeScript interfaces are used in this project and why?**

- **Answer:** This project uses several types of interfaces to enforce type safety and improve code quality. Let me break them down by category:

**1. Model Attributes Interfaces**

These define the structure of database entities:

```typescript
// Teacher.ts
interface TeacherAttributes {
  id: number;
  email: string;
  name: string;
  createdAt?: Date;
  updatedAt?: Date;
}

// Used in Sequelize Model class
class Teacher
  extends Model<TeacherAttributes, Optional<TeacherAttributes, "id">>
  implements TeacherAttributes
{
  public id!: number;
  public email!: string;
  public name!: string;
}
```

**Why:** Ensures Sequelize models are type-safe and prevents typos when accessing properties.

**2. Repository Contract Interface (IRepository)**

Defines the contract that all repositories must follow:

```typescript
// src/database/repositories/types/IRepository.ts
export interface IRepository<T> {
  findOne(options: {...}): Promise<T | null>;
  findAll(options?: {...}): Promise<T[]>;
  create(data: Partial<T>, transaction?: Transaction): Promise<T>;
  update(instance: T, transaction?: Transaction): Promise<T>;
  delete(instance: T, transaction?: Transaction): Promise<void>;
  upsert(data: Partial<T>, transaction?: Transaction): Promise<[T, boolean]>;
  destroy(options: {...}): Promise<number>;
}
```

**Why this pattern is powerful:**

- ✅ **Contract enforcement:** All repositories must implement these methods
- ✅ **Consistency:** Every repository has the same base API
- ✅ **Documentation:** Interface serves as living documentation
- ✅ **Refactoring safety:** If you change the interface, TypeScript will catch all places that need updating
- ✅ **Testability:** Easy to create mock repositories for testing

**Usage example:**

```typescript
// BaseRepository implements the contract
export abstract class BaseRepository<T extends Model>
  implements IRepository<T> {
  constructor(protected model: ModelCtor<T>) {}

  async findOne(options: {...}): Promise<T | null> {
    return this.model.findOne(options);
  }
  // ... other methods
}

// Specific repositories extend BaseRepository
export class TeacherRepository extends BaseRepository<Teacher> {
  // Automatically has findOne, findAll, create, etc.
  // Can add custom methods specific to Teacher
}
```

**3. API Response Interfaces**

Define the shape of data returned by services:

```typescript
// StudentListingService.ts
export interface StudentListingResponse {
  count: number;
  students: Array<{
    id: number;
    name: string;
    email: string;
  }>;
}

// External API response
export interface ExternalStudentResponse {
  count: number;
  students: ExternalStudent[];
}

export interface ExternalStudent {
  id: number;
  name: string;
  email: string;
}
```

**Why:**

- ✅ **Type safety in API responses:** Controllers know exactly what data shape to expect
- ✅ **Self-documenting:** Anyone reading the code can see the response structure
- ✅ **Prevents errors:** TypeScript catches typos like `response.studnets` (typo)
- ✅ **IDE support:** Autocomplete works perfectly

**4. Data Transfer Interfaces (For CSV)**

```typescript
// CsvItem.ts
export interface CsvItem {
  teacherEmail: string;
  teacherName: string;
  studentEmail: string;
  studentName: string;
  classCode: string;
  classname: string;
  subjectCode: string;
  subjectName: string;
  toDelete: string;
}

// ValidationError.ts
export interface ValidationError {
  row: number;
  field: string;
  value: unknown;
  message: string;
}
```

**Why:**

- ✅ **CSV parsing safety:** Parser knows expected columns
- ✅ **Validation clarity:** Error structure is well-defined
- ✅ **Maintainability:** If CSV structure changes, TypeScript catches all affected code

**5. Internal Business Logic Interfaces**

Used within services for type-safe data manipulation:

```typescript
// WorkloadReportService.ts
interface SubjectWorkload {
  subjectCode: string;
  subjectName: string;
  numberOfClasses: number;
}

interface WorkloadReport {
  [teacherName: string]: SubjectWorkload[]; // Index signature
}

interface TeacherClassSubjectRelation {
  Teacher?: {
    id: number;
    name: string;
  };
  Subject?: {
    id: number;
    code: string;
    name: string;
  };
  Class?: {
    id: number;
  };
}
```

**Why:**

- ✅ **Complex data structures:** Index signatures like `[teacherName: string]` for dynamic keys
- ✅ **Optional properties:** `Teacher?:` indicates this might not exist (from joins)
- ✅ **Business logic safety:** Prevents errors when aggregating data

**6. Transaction Interface**

```typescript
// ITransaction.ts
export interface ITransaction {
  commit: () => Promise<void>;
  rollback: () => Promise<void>;
}
```

**Why:**

- ✅ **Abstraction:** Services don't need to know Sequelize transaction implementation
- ✅ **Testability:** Easy to mock transactions in tests
- ✅ **Future-proof:** Can switch ORMs without changing service code

---

**Key TypeScript Patterns in This Project:**

**1. Generic Interfaces with Type Parameters**

```typescript
// IRepository<T> is generic - works with any model type
interface IRepository<T> {
  findOne(options: {...}): Promise<T | null>;  // T can be Teacher, Student, etc.
}

// When used:
class TeacherRepository implements IRepository<Teacher> {
  // Now TypeScript knows findOne returns Promise<Teacher | null>
}
```

**2. Optional Properties with `?`**

```typescript
interface TeacherAttributes {
  id: number;
  email: string;
  createdAt?: Date; // Optional - might not exist when creating
}
```

**3. Index Signatures for Dynamic Keys**

```typescript
interface WorkloadReport {
  [teacherName: string]: SubjectWorkload[]; // Any string key is valid
}

// Usage:
const report: WorkloadReport = {
  "John Doe": [...],
  "Jane Smith": [...],
  // Can add any teacher name dynamically
};
```

**4. Union Types with `|`**

```typescript
findOne(options: {...}): Promise<T | null>
// Returns either T OR null (not both)
```

**5. Utility Types**

```typescript
create(data: Partial<T>): Promise<T>
// Partial<T> makes all properties of T optional
// Useful for create/update operations where you don't need all fields
```

---

**Interview Questions You Might Face:**

**Q: "Why use interfaces instead of types?"**

**A:** "In this project, we use interfaces because:

1. **Extendability:** Interfaces can be extended and merged (declaration merging)
2. **OOP compatibility:** Works well with classes (implements keyword)
3. **Clearer intent:** Interface says 'this is a contract', type says 'this is a shape'

However, **type aliases** are also valid and we could use them interchangeably for simple cases."

**Q: "What's the benefit of IRepository interface if BaseRepository already implements everything?"**

**A:** "Great question! The benefits are:

1. **Contract documentation:** The interface clearly states 'this is what every repository must do'
2. **Multiple implementations:** We could create a MockRepository for testing without extending BaseRepository
3. **Dependency injection:** Services can depend on IRepository interface, not concrete implementations
4. **Future flexibility:** Could switch to a different base implementation without changing dependent code"

**Q: "Why not use DTOs instead of these interfaces?"**

**A:** "These interfaces serve a different purpose:

- **Model interfaces:** Define database entities (what Sequelize needs)
- **Response interfaces:** Define API response shapes (what controllers return)
- **DTOs:** Would define API request/response transformation layer

For this project's size, using interfaces directly is simpler. DTOs would add value if:

- We needed complex validation
- API and database schemas diverge
- We needed to support multiple API versions"

**Q: "What's the difference between `interface` and `class` in TypeScript?"**

**A:**

| Feature            | Interface                          | Class                                  |
| ------------------ | ---------------------------------- | -------------------------------------- |
| **Exists at**      | Compile-time only (disappears)     | Runtime (actual JavaScript code)       |
| **Purpose**        | Type checking and contracts        | Blueprint for creating objects         |
| **Can implement**  | No                                 | Yes, can contain logic                 |
| **Can extend**     | Yes (multiple interfaces)          | Yes (single class)                     |
| **Memory**         | Zero runtime cost                  | Uses memory for instances              |
| **When to use**    | Defining shapes, contracts         | Need instances with behavior           |
| **Example in app** | `IRepository<T>` - just a contract | `BaseRepository` - actual              |
|                    |                                    | implementation                         |
|                    | `CsvItem` - just data shape        | `Teacher extends Model` - has behavior |

**Real example from our project:**

```typescript
// Interface - no runtime cost, just type checking
export interface IRepository<T> {
  findOne(...): Promise<T | null>;
}

// Class - generates actual JavaScript code
export class BaseRepository<T> implements IRepository<T> {
  constructor(protected model: ModelCtor<T>) {}  // Has state

  async findOne(...) {  // Has behavior
    return this.model.findOne(options);
  }
}
```

---

**Summary: Why Interfaces Matter in This Project**

1. ✅ **Catch errors at compile-time** instead of runtime
2. ✅ **IDE autocomplete and intellisense** work perfectly
3. ✅ **Self-documenting code** - interfaces show intent
4. ✅ **Refactoring safety** - TypeScript catches breaking changes
5. ✅ **Team collaboration** - interfaces act as contracts between developers
6. ✅ **Testability** - easy to create mocks that match interfaces

**The project uses interfaces everywhere possible to maximize TypeScript's benefits while keeping code clean and maintainable.**

**45. What does `Partial<T>` mean in TypeScript?**

- **Answer:** Makes all properties of type `T` optional.

  ```typescript
  interface Student {
    id: number;
    name: string;
    email: string;
  }

  // BaseRepository uses Partial<T> for create:
  async create(data: Partial<Student>) {
    // data can be: { name: "John" }  // ✅ email not required
    // or: { name: "John", email: "..." }  // ✅ also OK
  }
  ```

**46. What is `ModelCtor<T>` in Sequelize with TypeScript?**

- **Answer:** It's a type that represents a Sequelize Model constructor/class.

  ```typescript
  // BaseRepository constructor
  constructor(protected model: ModelCtor<T>) {}

  // When creating ClassRepository:
  super(Class as ModelCtor<Class>);
  // Tells TypeScript: "Class is a Sequelize Model constructor"
  ```

---
