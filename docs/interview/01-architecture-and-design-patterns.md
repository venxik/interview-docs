# Architecture & Design Patterns

This guide covers architectural decisions, design patterns, and trade-offs in the project. Understanding these concepts is crucial for explaining the "why" behind the project's structure.

---

## Topics Covered

- Layered Monolithic Architecture
- Repository Pattern
- Dependency Injection (DI)
- Singleton Pattern
- Template Method Pattern
- Architectural decisions and reasoning
- Trade-offs and future improvements
- DTOs (Data Transfer Objects)

---

### **Category 1: Architecture & Design Patterns**

These questions test your understanding of the "why" behind the project's structure.

**1. Can you describe the architecture of this project?**

- **Answer:** "The project uses a **Layered Monolithic Architecture**. It's organized into four distinct layers as documented in `ARCHITECTURE.md`:
  - **API Layer (`src/api`):** Handles all HTTP requests, responses, and routing.
  - **Business Logic Layer (`src/business`):** Contains the core business rules and orchestrates operations.
  - **Data Access Layer (`src/database/repositories`):** Abstracts all database interactions.
  - **Database Layer (`src/database/models`):** Defines the data entities using the Sequelize ORM."

**2. Why was a Layered Architecture chosen? What are its benefits?**

- **Answer:** "The primary benefit, as stated in the documentation, is **Separation of Concerns (SoC)**. Each layer has a single, well-defined responsibility. This makes the codebase easier to understand, maintain, and test. For example, business logic isn't mixed with HTTP handling, and database queries are kept separate from business rules."

**3. The project uses the Repository Pattern. What is its purpose?**

- **Answer:** "The Repository Pattern is crucial here for several reasons mentioned in `ARCHITECTURE.md`:
  1.  **Abstracts Data Access:** The business layer (services) doesn't know if the database is MySQL or something else, or if it's using Sequelize or raw SQL. It just calls a method like `classRepository.findByCode()`.
  2.  **Improves Testability:** It's easy to mock the repository in tests. This allows for fast unit tests on the business logic without needing a real database connection, which is the project's primary testing strategy.
  3.  **Centralizes Query Logic:** All database queries for a specific entity (like `Class`) are in one place (`ClassRepository.ts`), making them easy to find and manage."

**4. What is the purpose of `BaseRepository.ts` and its `constructor()`?**

- **Answer:** "`BaseRepository.ts` uses the **Template Method Pattern** to provide common CRUD (Create, Read, Update, Delete) operations that all other repositories can inherit.
  - The `constructor(protected model: ModelCtor<T>)` is a form of **Dependency Injection (DI)**. It allows a specific Sequelize model (like `Student` or `Class`) to be passed in when a repository is created. This makes the `BaseRepository` generic and reusable for any model, avoiding code duplication (DRY principle)."

**4.1. How is Dependency Injection implemented in this project? Can you explain the pattern in detail?**

- **Answer:** "This project uses **Constructor-Based Dependency Injection** in the Repository layer. Let me walk through the implementation:

**The Pattern:**

```typescript
// src/database/repositories/BaseRepository.ts:19
export abstract class BaseRepository<T extends Model>
  implements IRepository<T>
{
  constructor(protected model: ModelCtor<T>) {}
  // ↑ This is Dependency Injection!
  // The dependency (Sequelize model) is "injected" via the constructor
}
```

**How it's used in concrete repositories:**

```typescript
// src/database/repositories/ClassRepository.ts:9-11
export class ClassRepository extends BaseRepository<Class> {
  constructor() {
    super(Class as ModelCtor<Class>); // Inject the Class model
  }
}

// src/database/repositories/StudentRepository.ts:6-8
export class StudentRepository extends BaseRepository<Student> {
  constructor() {
    super(Student as ModelCtor<Student>); // Inject the Student model
  }
}
```

**Why is this Dependency Injection?**

1. **Dependency:** `BaseRepository` depends on a Sequelize model to perform database operations
2. **Injection:** Instead of creating the model inside `BaseRepository` (tight coupling), we pass it in from outside
3. **Inversion of Control:** The caller controls which model to use, not the `BaseRepository`

**Benefits of this DI pattern:**

1. ✅ **Generic/Reusable Code:** One `BaseRepository` works with any model (Student, Teacher, Class, etc.)
2. ✅ **Loose Coupling:** `BaseRepository` doesn't need to know about specific models at compile time
3. ✅ **Testability:** Can inject mock models for testing without touching real database
4. ✅ **DRY Principle:** Avoids duplicating CRUD logic across 6+ repositories
5. ✅ **Type Safety:** TypeScript generic `<T>` ensures type-safe operations

**Alternative WITHOUT Dependency Injection (Bad):**

```typescript
// ❌ Tight coupling - not reusable
export class ClassRepository {
  private model = Class; // Hard-coded dependency!

  async findAll() {
    return this.model.findAll(); // Always uses Class model
  }
}

// ❌ Would need to duplicate for every model:
export class StudentRepository {
  private model = Student; // Duplicated code!

  async findAll() {
    return this.model.findAll(); // Same logic, different model
  }
}
```

**Our approach WITH Dependency Injection (Good):**

```typescript
// ✅ Loose coupling - highly reusable
export abstract class BaseRepository<T extends Model> {
  constructor(protected model: ModelCtor<T>) {} // Injected!

  async findAll() {
    return this.model.findAll(); // Works with ANY model
  }
}

// ✅ Minimal code in concrete repositories
export class ClassRepository extends BaseRepository<Class> {
  constructor() {
    super(Class); // Just inject the model, inherit all methods
  }
}
```

**The Singleton Export Pattern:**

```typescript
// src/database/repositories/ClassRepository.ts:55
export default new ClassRepository();
// ↑ Single instance exported, model injected at creation time
```

**Why export as singleton?**

- Creates one instance of `ClassRepository` with `Class` model injected
- All imports share the same instance (memory efficient)
- The model dependency is resolved once at module load time

**How services use the injected repositories:**

```typescript
// src/services/DataUploadService.ts:4-9
import {
  teacherRepository, // Already has Teacher model injected
  studentRepository, // Already has Student model injected
  classRepository, // Already has Class model injected
} from "@repositories";

// Service doesn't need to know HOW repositories work internally
await teacherRepository.findByEmails(emails); // Just use them!
```

**This is a second layer of DI:**

- Repositories are injected into services (though implicitly via imports, not constructor injection in this project)
- Services depend on repository abstractions, not concrete implementations
- This separation enables easy testing (can mock repositories)

**DI in Testing:**

```typescript
// src/database/repositories/__tests__/ClassRepository.test.ts:11
jest.mock("@models"); // Mock the injected dependency

const mockClass = { id: 1, code: "P1-1", name: "Primary 1" };

// Spy on repository methods
jest.spyOn(repository, "findOne").mockResolvedValue(mockClass);

// Now tests run without real database - DI makes this possible!
```

**Key Takeaways:**

1. **Constructor Injection:** Dependencies passed via constructor (`constructor(protected model)`)
2. **Generic Types:** `<T>` makes the pattern reusable for any model type
3. **Protected Keyword:** Child classes can access the injected `model` property
4. **Singleton Pattern:** Combined with DI for efficient resource usage
5. **Testability:** DI makes mocking dependencies straightforward

**Interview Follow-up Questions:**

**Q: "Is this true Dependency Injection or just constructor parameters?"**

**A:** "It's true Constructor-Based DI because:

- The dependency (model) is provided externally, not created internally
- Follows the Dependency Inversion Principle (depend on abstractions, not concretions)
- Enables loose coupling and testability

The difference from a simple parameter is the **intent**: we're explicitly managing dependencies to improve code structure."

**Q: "Why not use a DI container like InversifyJS?"**

**A:** "For this project's size, manual DI is simpler and more transparent:

- ✅ **Simplicity:** No additional library, easier to understand
- ✅ **Explicit:** Clear where dependencies come from
- ✅ **Lightweight:** No framework overhead

DI containers add value for:

- Large projects with 100+ classes
- Complex dependency graphs
- Runtime dependency resolution
- Multiple implementations of the same interface"

**Q: "Could you use DI for service dependencies too?"**

**A:** "Absolutely! We could refactor services like this:

```typescript
// Current (implicit dependencies via imports):
import { teacherRepository } from "@repositories";

export class DataUploadService {
  async processData() {
    await teacherRepository.findByEmails(); // Direct import usage
  }
}

// With Constructor DI (explicit dependencies):
export class DataUploadService {
  constructor(
    private teacherRepository: TeacherRepository,
    private studentRepository: StudentRepository,
    private classRepository: ClassRepository,
  ) {}

  async processData() {
    await this.teacherRepository.findByEmails(); // Uses injected dependency
  }
}

// Create instance with injected dependencies:
export default new DataUploadService(
  teacherRepository,
  studentRepository,
  classRepository,
);
```

**Benefits:**

- ✅ Explicit dependencies in constructor (self-documenting)
- ✅ Easier to mock in tests
- ✅ More flexible (can swap implementations)

**Trade-off:**

- ❌ More boilerplate code
- ❌ May be overkill for small projects

For this project's current size, the implicit import approach is pragmatic.""

**5. Why are services and repositories exported as singletons (e.g., `export default new ClassService()`)?**

- **Answer:** "The `DESIGN_PATTERNS_SUMMARY.md` notes this is the **Singleton Pattern**. The main reasons are:
  - **Efficiency:** It ensures only one instance of a service or repository exists throughout the application, which can be more memory-efficient.
  - **State Management:** While not heavily used for state here, it provides a single, global access point for these stateless services."

---

### **Category 4: Architectural Decisions & Reasoning**

**15. Why are database transactions managed by the Controller and not the Service?**

- **Answer:** "That's a crucial architectural decision that ensures data integrity across complex operations. The transaction is started in the controller to ensure that a single API request, which represents a single **unit of work**, is fully **atomic** (i.e., it either succeeds completely or fails completely).
  - **The Problem with Service-Level Transactions:** Imagine a future feature to 'transfer a student', which might involve two services: `enrollmentService.unenroll()` and `enrollmentService.enroll()`. If the `unenroll` service started and committed its own transaction, but then the `enroll` service failed, you could not roll back the unenrolling of the student. The data would become inconsistent.
  - **The Controller-Level Solution:** By starting the transaction in the controller and passing it down to all the services involved in the operation, the entire process is wrapped in one transaction. If any service fails, the controller can roll back everything, ensuring the database remains in a valid state. This makes our services more reusable, composable, and keeps the business logic pure and free from infrastructure concerns."

---

### **Category 5: Trade-offs & Future Improvements**

**16. What are the pros and cons of the current "Layered Monolithic" architecture?**

- **Answer:**
  - **Pros:** It's simple, well-organized, and easy for a new developer to understand the flow of control (API -> Business -> Data). It's very effective for small-to-medium-sized applications like this one.
  - **Cons:** As the application grows, you might find that making a change to one feature requires modifying files in many different layers. It's also scaled as a single unit, meaning you can't scale the "student" service independently of the "report" service."

**17. The `FOLDER_STRUCTURE_GUIDE.md` proposes moving to a Domain-Driven or Clean Architecture. Why not just start with that?**

- **Answer:** "That's a great question about trade-offs. While Clean Architecture is powerful, it adds more layers of abstraction and boilerplate code. For a project of this size, a Layered Architecture is often a more pragmatic starting point—it's simpler and faster to develop with. The documentation wisely lays out an evolutionary path, suggesting that the project can migrate to a more complex architecture like DDD or Clean Architecture **when the need arises**, which is a sign of a mature development strategy."

**18. What are DTOs (Data Transfer Objects) and why doesn't this project use them?**

- **Answer:** **DTOs (Data Transfer Objects)** are simple objects used to transfer data between different layers of an application, particularly between the API layer and the service/business logic layer.

  **What are DTOs?**

  - Plain objects that define the shape of data for API requests and responses
  - Usually implemented as classes or interfaces
  - Contain only data (no business logic)
  - Often include validation rules

  **Example of DTOs in a typical project:**

  ```typescript
  // DTOs for a User API

  // Request DTO (what client sends)
  export class CreateUserDto {
    @IsString()
    @IsNotEmpty()
    name: string;

    @IsEmail()
    email: string;

    @IsString()
    @MinLength(8)
    password: string;
  }

  // Response DTO (what API returns)
  export class UserResponseDto {
    id: number;
    name: string;
    email: string;
    createdAt: Date;
    // password is excluded for security
  }

  // Controller using DTOs
  @Post('/users')
  async createUser(@Body() createUserDto: CreateUserDto): Promise<UserResponseDto> {
    const user = await userService.create(createUserDto);
    return new UserResponseDto(user);  // Transform to response DTO
  }
  ```

  **Why This Project Doesn't Use DTOs:**

  **1. Simple Data Structure**

  - This project has straightforward CRUD operations
  - Direct mapping between API requests and database models
  - No complex data transformations needed

  **Current approach in ClassController.ts:**

  ```typescript
  const { className } = req.body; // Direct extraction
  await classService.updateClassName(classCode, className);
  ```

  **With DTOs, it would be:**

  ```typescript
  const updateClassDto = new UpdateClassDto(req.body);
  await classService.updateClassName(classCode, updateClassDto);
  ```

  **2. Small Project Scale**

  - Adding DTOs would introduce overhead without significant benefit
  - For 5-6 controllers, manual validation is manageable
  - DTOs shine in larger projects with 50+ endpoints

  **3. Tight Coupling is Acceptable Here**

  - Using Sequelize models directly in controllers is fine for small projects
  - The project already has good separation between layers
  - Models serve dual purpose: database schema + data structure

  **When Would DTOs Be Beneficial Here?**

  **1. API Version Management**

  ```typescript
  // v1: Simple response
  export class StudentResponseDto_V1 {
    id: number;
    name: string;
  }

  // v2: More detailed response
  export class StudentResponseDto_V2 extends StudentResponseDto_V1 {
    email: string;
    enrolledClasses: string[];
  }

  // Database model stays same, but API responses differ
  ```

  **2. Different Response Formats**

  ```typescript
  // Internal use (has sensitive data)
  interface StudentEntity {
    id: number;
    name: string;
    email: string;
    password: string;
    socialSecurityNumber: string;
  }

  // Public API (sensitive data removed)
  class StudentDto {
    id: number;
    name: string;
    email: string;
    // password and SSN excluded
  }
  ```

  **3. Complex Business Logic**

  ```typescript
  // DTO with calculated fields
  class WorkloadReportDto {
    teacherName: string;
    totalClasses: number;
    totalStudents: number;
    subjectBreakdown: SubjectBreakdownDto[];

    // Calculated in DTO constructor
    averageClassSize: number;
    workloadScore: number;
  }
  ```

  **4. Input Validation**

  ```typescript
  // With class-validator
  export class CreateClassDto {
    @IsString()
    @Length(3, 10)
    code: string;

    @IsString()
    @Length(5, 100)
    name: string;

    // Automatic validation on request
  }

  // Current project: Manual validation
  if (!className || typeof className !== "string") {
    return res.status(400).json({ error: "Invalid className" });
  }
  ```

  **Pros and Cons of DTOs:**

  **Pros:**

  - ✅ **Type Safety:** Clear contract between API and client
  - ✅ **Validation:** Centralized validation with decorators
  - ✅ **Documentation:** Self-documenting API structure
  - ✅ **Decoupling:** API layer independent of database schema
  - ✅ **Versioning:** Easy to maintain multiple API versions
  - ✅ **Security:** Control exactly what data is exposed
  - ✅ **Transformation:** Convert data formats (snake_case ↔ camelCase)

  **Cons:**

  - ❌ **Boilerplate:** More code to write and maintain
  - ❌ **Mapping Overhead:** Need to map between entities and DTOs
  - ❌ **Learning Curve:** Team needs to understand DTO pattern
  - ❌ **Duplication:** Can feel redundant for simple CRUD
  - ❌ **Memory:** Extra objects created during transformations

  **Current Project Architecture (Without DTOs):**

  ```
  Request → Controller → Service → Repository → Database Model
                ↓
           Uses req.body directly
           Returns Sequelize Model
  ```

  **With DTOs Architecture:**

  ```
  Request → Controller → DTO Validation → Service → Entity/Domain Model → Repository → Database Model
                ↓                                      ↓
         Creates DTO from req.body              Transforms DTO to Entity
         Returns DTO to client                  Transforms Entity to DTO
  ```

  **When to Introduce DTOs:**

  **Scenario 1: API becomes public**

  ```typescript
  // Before: Internal API
  res.json(student); // Exposes all fields including password hash

  // After: Public API with DTO
  res.json(new StudentDto(student)); // Only safe fields exposed
  ```

  **Scenario 2: Adding API versioning**

  ```typescript
  // v1 endpoint
  @Get('/v1/students/:id')
  getStudentV1(): StudentDtoV1 { ... }

  // v2 endpoint (more fields)
  @Get('/v2/students/:id')
  getStudentV2(): StudentDtoV2 { ... }

  // Same database model, different DTOs
  ```

  **Scenario 3: Complex validation requirements**

  ```typescript
  export class UploadDataDto {
    @IsArray()
    @ValidateNested({ each: true })
    @ArrayMinSize(1)
    @ArrayMaxSize(1000)
    students: StudentDataDto[];

    @IsOptional()
    @IsBoolean()
    skipDuplicates?: boolean;

    // Much cleaner than manual if/else validation
  }
  ```

  **Practical Example: Adding DTOs to This Project**

  **Step 1: Install validation library**

  ```bash
  npm install class-validator class-transformer
  ```

  **Step 2: Create DTOs**

  ```typescript
  // src/api/dto/UpdateClassDto.ts
  import { IsString, Length } from "class-validator";

  export class UpdateClassDto {
    @IsString({ message: "className must be a string" })
    @Length(5, 100, {
      message: "className must be between 5 and 100 characters",
    })
    className: string;
  }
  ```

  **Step 3: Use in controller**

  ```typescript
  // Before (current)
  const { className } = req.body;
  if (!className || typeof className !== "string") {
    return res.status(400).json({ error: "Invalid className" });
  }

  // After (with DTO)
  const updateDto = plainToClass(UpdateClassDto, req.body);
  const errors = await validate(updateDto);
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }
  ```

  **Recommendation for This Project:**

  - **Current approach is fine** for the project's size and complexity
  - Consider adding DTOs if:
    - API becomes public-facing
    - Need to support multiple API versions
    - Validation logic becomes complex (5+ rules per endpoint)
    - Team grows beyond 5 developers
    - Planning to migrate to Clean Architecture

**18.1. How would you implement DTOs with validation in this project? (Step-by-Step Implementation Guide)**

- **Answer:** "For a complete step-by-step implementation guide for adding DTOs with the `class-validator` library, including validation middleware, testing patterns, and migration strategy, see the detailed implementation section in Question 18 above. The guide covers:
  - Installing required packages
  - Creating DTO directory structure
  - Building validation middleware
  - Creating DTOs for each endpoint
  - Updating controllers to use DTOs
  - Testing strategies
  - Common validation decorators reference
  - Custom validators
  - Benefits summary
  - Migration strategy"

**19. What is one key improvement you would suggest for this system?**

- **Answer:** "The `ARCHITECTURE.md` file provides a fantastic list of potential enhancements. I would pick one and elaborate:
  - **"I would add a dedicated validation layer.** Currently, request validation isn't standardized. I'd introduce a library like `Joi` or `class-validator` in the API layer (potentially with DTOs). This would ensure that bad data (e.g., an invalid email format, a missing field) is rejected immediately with a clear `400 Bad Request` error, before it even reaches the business logic. This makes the services more robust."

---

## Key Takeaways

- The project uses a **Layered Monolithic Architecture** for simplicity and clear separation of concerns
- **Repository Pattern** abstracts data access and improves testability
- **Dependency Injection** is implemented through constructor injection in repositories
- **Singleton Pattern** ensures single instances of services and repositories
- **Template Method Pattern** in BaseRepository provides reusable CRUD operations
- Controller-level transactions ensure atomicity across multiple service calls
- DTOs are not used due to project simplicity, but would be beneficial for larger projects
- The architecture prioritizes pragmatism and evolutionary design
