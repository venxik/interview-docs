# Development Workflow

This guide covers the practical development workflow, including adding new features and understanding barrel files.

---

## Topics Covered

- Step-by-step feature development process
- Barrel files (index.ts) and import patterns
- Named vs default exports
- Clean import strategies

---

### **Category 2: Practical Development & Workflow**

These questions test your ability to work within the existing structure.

**6. If you had to add a new feature, like "Get a list of subjects for a specific teacher," what would be your step-by-step process?**

- **Answer:** "I would follow the established workflow outlined in the documentation:
  1.  **Repository:** First, I'd add a new method in `TeacherClassSubjectRepository.ts` to find all subjects linked to a given teacher ID.
  2.  **Service:** Next, I'd create a new method in a relevant service, perhaps `WorkloadReportService.ts` or a new `TeacherService.ts`. This service would call the new repository method.
  3.  **Controller:** I'd then create a new route handler in `ReportController.ts` or a new `TeacherController.ts` to define the API endpoint (e.g., `GET /api/teachers/:teacherId/subjects`). This controller would call the service method.
  4.  **Router:** I would wire up the new controller endpoint in `router.ts`.
  5.  **Testing:** Crucially, I would write a unit test for the new service method, mocking the repository to ensure the business logic is correct without needing a database."

**7. What is the purpose of the `index.ts` (barrel) files, and why do they use the syntax `export { default as SomeName } from './SomeFile'`?**

- **Answer:** "Those `index.ts` files are called 'barrel files'. Their main purpose is to aggregate and re-export modules from a directory, making imports cleaner.

  - For example, instead of `import { classRepository } from '../repositories/ClassRepository'; import { studentRepository } from '../repositories/StudentRepository';`, you can do a single import: `import { classRepository, studentRepository } from '@repositories';`.

  The `export { default as SomeName } from './SomeFile'` syntax is crucial to making barrel files effective:

  1.  **It creates a named export from a default export.** Most service and repository files in this project export a `default` singleton instance (e.g., `export default new ClassService()`). This syntax takes that default export and re-exports it as a named export (e.g., `classService`).
  2.  **Consistent Imports:** It allows developers to use a consistent import style everywhere. They can always use named imports (`import { ... }`) from barrel files, instead of having to remember which modules have a default export and which don't.
  3.  **Avoids Default Export Conflicts:** A module can only have one `default` export. If a barrel file tried to re-export `default` from multiple files, it would be impossible. By converting each `default` into a unique named export, the barrel file can safely export dozens of modules.
  4.  **Flexibility (Exporting Both Class and Instance):** This pattern is often extended to export both the class (for type-hinting or extension) and its singleton instance. \* _Example File (`ClassService.ts`):_
      `typescript
export class ClassService { /* ... */ }
export default new ClassService();
` \* _Barrel File (`index.ts`):_
      `typescript
export { ClassService, default as classService } from './ClassService';
` \* _Consumer Code:_
      `typescript
// Import the instance for use
import { classService } from '@services';
// Import the class for type-hinting or extension
import { ClassService } from '@services';
`
      In short, this pattern is fundamental to making the project's imports clean, scalable, and easy to use."

### **Category 7: How to Add a New Feature (Practical Walkthrough)**

This section provides a step-by-step guide for adding a new feature, following the project's established architecture and conventions.

**Example Scenario:** We will add a new feature to **get a teacher's details by their email address**.

**The Goal:** Create a new API endpoint `GET /api/teachers?email=some.email@school.com`.

---

#### **Step 1: The Data Access Layer (Repository)**

First, we need a way to query the database for a teacher by their email.

1.  **Locate the Repository:** The correct file is `src/database/repositories/TeacherRepository.ts`.
2.  **Add the Method:** We'll add a new method `findByEmail` to this class.

```typescript
// src/database/repositories/TeacherRepository.ts

import { Teacher } from "@models";
import { BaseRepository } from "./BaseRepository";

export class TeacherRepository extends BaseRepository<Teacher> {
  constructor(model: typeof Teacher) {
    super(model);
  }

  // ... existing methods ...

  /**
   * Finds a single teacher by their email address.
   * @param {string} email The email to search for.
   * @returns {Promise<Teacher | null>} The teacher instance or null if not found.
   */
  async findByEmail(email: string): Promise<Teacher | null> {
    return this.findOne({ where: { email } });
  }
}
```

---
