# Problem-Solving & Complexity

This guide covers problem-solving approaches, Big O notation, and computational complexity.

---

## Topics Covered

- Problem-solving methodology
- Big O notation and time complexity
- Space complexity
- Code optimization strategies
- Data structures and algorithms
- Debugging approaches

---

## **Category 17: Logic & Problem-Solving**

These questions test your analytical thinking, problem-solving approach, and understanding of complexity.

---

**66. Explain your approach to solving a coding problem you've never seen before.**

- **Answer:** "I follow a structured problem-solving approach:

**Step 1: Understand the Problem**

- Read the requirements carefully
- Ask clarifying questions:
  - What are the inputs and outputs?
  - What are the constraints (size limits, time limits)?
  - What are the edge cases?
  - Are there any special requirements?

**Step 2: Plan the Solution**

- Break down the problem into smaller sub-problems
- Identify the core algorithm or pattern:
  - Is it a search problem? (Binary search, DFS, BFS)
  - Is it a counting problem? (Hash map, frequency counter)
  - Is it an optimization problem? (Dynamic programming, greedy)
- Choose appropriate data structures:
  - Arrays for sequential data
  - Objects/Maps for key-value lookups
  - Sets for uniqueness checks
  - Stacks/Queues for specific traversal patterns

**Step 3: Consider Trade-offs**

- Time complexity vs space complexity
- Code readability vs performance
- Simple solution vs optimized solution

**Step 4: Implement**

- Start with a working solution (even if not optimal)
- Write clean, readable code
- Use meaningful variable names
- Add comments for complex logic

**Step 5: Test**

- Test with provided examples
- Test edge cases:
  - Empty input
  - Single element
  - Large input
  - Invalid input
- Test boundary conditions

**Step 6: Optimize**

- Analyze time and space complexity
- Look for redundant operations
- Consider better data structures
- Refactor if needed

**Example from this project:**

When implementing the CSV upload feature (`DataUploadService.ts`), I would:

1. **Understand:** Parse CSV, validate data, insert to database
2. **Plan:**
   - Parse CSV rows
   - Group by entity type (teachers, students, classes)
   - Validate each entity
   - Handle relationships (teacher-class-subject)
3. **Choose data structures:**
   - Maps for quick lookups (avoid duplicate queries)
   - Sets for duplicate detection
   - Arrays for batch operations
4. **Implement:** Layer by layer (parse → validate → insert)
5. **Test:** Empty CSV, invalid emails, missing relationships
6. **Optimize:** Batch database operations, reduce N+1 queries"

---

**67. What is Big O notation? Explain the common time complexities.**

- **Answer:** "Big O notation describes how an algorithm's runtime or space requirements grow as the input size increases. It focuses on the **worst-case scenario**.

**Common Time Complexities (from best to worst):**

| Notation | Name | Example | Description |
|----------|------|---------|-------------|
| O(1) | Constant | Array access `arr[5]` | Same time regardless of input size |
| O(log n) | Logarithmic | Binary search | Halves the problem each step |
| O(n) | Linear | Single loop | Grows directly with input size |
| O(n log n) | Linearithmic | Merge sort, Quick sort | Efficient sorting algorithms |
| O(n²) | Quadratic | Nested loops | Grows with square of input |
| O(n³) | Cubic | Triple nested loops | Very slow for large inputs |
| O(2ⁿ) | Exponential | Recursive fibonacci | Doubles with each input increase |
| O(n!) | Factorial | Generate all permutations | Extremely slow |

**Visual Growth (n = 100):**

- O(1): 1 operation
- O(log n): ~7 operations
- O(n): 100 operations
- O(n log n): ~700 operations
- O(n²): 10,000 operations
- O(2ⁿ): 1,267,650,600,228,229,401,496,703,205,376 operations (impossibly slow!)

**Examples:**

```javascript
// O(1) - Constant time
function getFirst(arr) {
  return arr[0]; // Always 1 operation
}

// O(n) - Linear time
function findMax(arr) {
  let max = arr[0];
  for (let i = 1; i < arr.length; i++) { // n operations
    if (arr[i] > max) max = arr[i];
  }
  return max;
}

// O(log n) - Logarithmic time
function binarySearch(sortedArr, target) {
  let left = 0, right = sortedArr.length - 1;
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (sortedArr[mid] === target) return mid;
    if (sortedArr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  return -1;
}

// O(n²) - Quadratic time
function findAllPairs(arr) {
  const pairs = [];
  for (let i = 0; i < arr.length; i++) {     // n times
    for (let j = i + 1; j < arr.length; j++) { // n times
      pairs.push([arr[i], arr[j]]);
    }
  }
  return pairs;
}

// O(n log n) - Linearithmic time
function mergeSort(arr) {
  if (arr.length <= 1) return arr;
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));   // log n divisions
  const right = mergeSort(arr.slice(mid));     // log n divisions
  return merge(left, right);                   // n operations per level
}
```

**In this project:**

```typescript
// src/business/DataUploadService.ts:125
const teachers = await teacherRepository.findByEmails(teacherEmails);
// Assuming findByEmails uses SQL WHERE IN: O(n) where n = number of emails

// src/business/DataUploadService.ts:270
const emailSet = new Set(students.map((s) => s.email));
// map(): O(n), Set creation: O(n) → Total: O(n)

// Nested loops (O(n²)) - should be avoided:
for (const student of students) {           // n times
  for (const cls of classes) {             // m times
    // O(n × m) - can be slow for large datasets
  }
}
```

**Interview Tip:**

- Always state the complexity of your solution
- Mention both time AND space complexity
- If asked to optimize, focus on reducing the highest order term (n² → n log n → n)"

---

**68. How would you optimize a function that is running slowly?**

- **Answer:** "I would follow a systematic debugging approach:

**Step 1: Measure and Identify the Bottleneck**

```javascript
// Use console.time() to measure
console.time('functionName');
slowFunction();
console.timeEnd('functionName'); // Output: functionName: 523.456ms

// Or use performance.now() for more precision
const start = performance.now();
slowFunction();
const end = performance.now();
console.log(`Execution time: ${end - start}ms`);
```

**Step 2: Analyze the Algorithm**

- What's the current time complexity?
- Are there nested loops? (O(n²) or higher)
- Are there redundant database queries?
- Are we doing unnecessary work?

**Step 3: Common Optimization Strategies**

**A) Use Better Data Structures**

```javascript
// ❌ Bad: O(n) lookup for each item
const students = [...]; // Array
function findStudentByEmail(email) {
  return students.find(s => s.email === email); // O(n)
}

// ✅ Good: O(1) lookup
const studentMap = new Map(students.map(s => [s.email, s]));
function findStudentByEmail(email) {
  return studentMap.get(email); // O(1)
}
```

**B) Avoid Redundant Operations**

```javascript
// ❌ Bad: Recalculating in every iteration
for (let i = 0; i < arr.length; i++) { // arr.length evaluated each time
  // ...
}

// ✅ Good: Calculate once
const len = arr.length;
for (let i = 0; i < len; i++) {
  // ...
}
```

**C) Use Caching/Memoization**

```javascript
// ❌ Bad: Recalculating same values
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2); // O(2ⁿ) - extremely slow!
}

// ✅ Good: Cache results
const cache = {};
function fibonacci(n) {
  if (n <= 1) return n;
  if (cache[n]) return cache[n];
  cache[n] = fibonacci(n - 1) + fibonacci(n - 2);
  return cache[n];
} // O(n) - much faster!
```

**D) Batch Database Operations**

```javascript
// ❌ Bad: N queries (N+1 problem)
for (const student of students) {
  await db.query('SELECT * FROM classes WHERE id = ?', [student.classId]);
}

// ✅ Good: 1 query
const classIds = students.map(s => s.classId);
await db.query('SELECT * FROM classes WHERE id IN (?)', [classIds]);
```

**E) Early Returns**

```javascript
// ❌ Bad: Checking all elements
function hasEmail(students, email) {
  let found = false;
  for (const student of students) {
    if (student.email === email) found = true;
  }
  return found;
}

// ✅ Good: Return as soon as found
function hasEmail(students, email) {
  for (const student of students) {
    if (student.email === email) return true;
  }
  return false;
}
```

**Step 4: Profile with Tools**

- Chrome DevTools Performance tab
- Node.js `--prof` flag for profiling
- Use `console.profile()` and `console.profileEnd()`

**In this project:**

```typescript
// src/business/DataUploadService.ts
// Good optimization: Batch fetch instead of individual queries
const teacherEmails = [...new Set(teacherClassSubjects.map(tcs => tcs.teacherEmail))];
const teachers = await teacherRepository.findByEmails(teacherEmails);
// ↑ 1 query instead of N queries
```

**Interview Tip:** Always measure before optimizing. Premature optimization is the root of all evil."

---

**69. Explain the difference between O(n) and O(n²) with a practical example.**

- **Answer:** "Let me demonstrate with a practical example:

**Scenario:** Finding duplicate emails in a list of students

**O(n) Solution - Using Set:**

```javascript
function findDuplicatesLinear(students) {
  const seen = new Set();
  const duplicates = new Set();

  for (const student of students) { // Loop once: n iterations
    if (seen.has(student.email)) {  // Set lookup: O(1)
      duplicates.add(student.email);
    } else {
      seen.add(student.email);
    }
  }

  return Array.from(duplicates);
}

// Time complexity: O(n)
// Space complexity: O(n) - storing in Set
```

**O(n²) Solution - Nested Loops:**

```javascript
function findDuplicatesQuadratic(students) {
  const duplicates = new Set();

  for (let i = 0; i < students.length; i++) {        // n iterations
    for (let j = i + 1; j < students.length; j++) {  // n iterations
      if (students[i].email === students[j].email) {
        duplicates.add(students[i].email);
      }
    }
  }

  return Array.from(duplicates);
}

// Time complexity: O(n²)
// Space complexity: O(k) where k = number of duplicates
```

**Performance Comparison:**

| Students (n) | O(n) operations | O(n²) operations | Difference |
|--------------|-----------------|------------------|------------|
| 10 | 10 | 100 | 10x slower |
| 100 | 100 | 10,000 | 100x slower |
| 1,000 | 1,000 | 1,000,000 | 1,000x slower |
| 10,000 | 10,000 | 100,000,000 | 10,000x slower |

**Real execution time example:**

```javascript
const students = Array.from({ length: 10000 }, (_, i) => ({
  id: i,
  name: `Student ${i}`,
  email: `student${i % 100}@example.com` // Introduces duplicates
}));

console.time('O(n) - Linear');
findDuplicatesLinear(students);
console.timeEnd('O(n) - Linear');
// Output: ~5ms

console.time('O(n²) - Quadratic');
findDuplicatesQuadratic(students);
console.timeEnd('O(n²) - Quadratic');
// Output: ~500ms (100x slower!)
```

**Why the huge difference?**

**O(n) approach:**
- Visits each student once
- Set lookup/add is O(1)
- Total: n × 1 = n operations

**O(n²) approach:**
- For each student, compares with all other students
- Student 1: compares with 9,999 others
- Student 2: compares with 9,998 others
- ...
- Total: ~50,000,000 comparisons!

**Visual representation (n = 5):**

```
O(n):     [1][2][3][4][5]  → 5 operations

O(n²):    [1] → compare with [2][3][4][5]  → 4 comparisons
          [2] → compare with [3][4][5]     → 3 comparisons
          [3] → compare with [4][5]        → 2 comparisons
          [4] → compare with [5]           → 1 comparison
          Total: 4+3+2+1 = 10 operations
```

**Key Takeaway:** O(n²) grows exponentially worse as input increases, while O(n) grows linearly."

---

**70. What are the trade-offs between different data structures?**

- **Answer:** "Here are the key trade-offs:

**Array vs Object/Map vs Set:**

| Operation | Array | Object | Map | Set |
|-----------|-------|--------|-----|-----|
| **Access by index** | O(1) | N/A | N/A | N/A |
| **Access by key** | O(n) | O(1) | O(1) | N/A |
| **Insert at end** | O(1) | O(1) | O(1) | O(1) |
| **Insert at start** | O(n) | O(1) | O(1) | O(1) |
| **Delete** | O(n) | O(1) | O(1) | O(1) |
| **Search** | O(n) | O(1) by key | O(1) by key | O(1) |
| **Maintain order** | ✅ Yes | ❌ No | ✅ Yes (insertion order) | ✅ Yes (insertion order) |
| **Allow duplicates** | ✅ Yes | ❌ No (keys unique) | ❌ No (keys unique) | ❌ No |
| **Use case** | Sequential data | Key-value pairs | Key-value with iteration | Unique values |

**Detailed Examples:**

**1. Array - Best for sequential data**

```javascript
const students = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

// ✅ Good for:
students.push(newStudent);        // O(1) - add to end
students[0];                      // O(1) - access by index
students.map(s => s.name);        // Transformation

// ❌ Bad for:
students.find(s => s.id === 5);   // O(n) - search by property
students.shift();                 // O(n) - remove from start
```

**2. Object - Best for key-value lookups (string keys only)**

```javascript
const studentById = {
  '1': { id: 1, name: 'Alice' },
  '2': { id: 2, name: 'Bob' }
};

// ✅ Good for:
studentById['1'];                 // O(1) - instant lookup
studentById['3'] = newStudent;    // O(1) - add/update
delete studentById['1'];          // O(1) - remove

// ❌ Bad for:
// Keys must be strings (numbers converted)
// No guaranteed order
// Inherits from Object.prototype (can have conflicts)
```

**3. Map - Best for key-value with any key type**

```javascript
const studentMap = new Map();
studentMap.set(1, { id: 1, name: 'Alice' });
studentMap.set({ key: 'obj' }, 'value'); // Objects as keys!

// ✅ Good for:
studentMap.get(1);                // O(1) - lookup
studentMap.has(1);                // O(1) - check existence
studentMap.delete(1);             // O(1) - remove
studentMap.size;                  // Number of entries
// Maintains insertion order
for (const [key, value] of studentMap) { ... } // Easy iteration

// ❌ Bad for:
// Slightly more memory than Object
// Not JSON-serializable directly
```

**4. Set - Best for unique values**

```javascript
const uniqueEmails = new Set();
uniqueEmails.add('alice@example.com');
uniqueEmails.add('bob@example.com');
uniqueEmails.add('alice@example.com'); // Ignored (duplicate)

// ✅ Good for:
uniqueEmails.has('alice@example.com'); // O(1) - check membership
uniqueEmails.add(email);               // O(1) - add
uniqueEmails.delete(email);            // O(1) - remove
uniqueEmails.size;                     // Count unique items

// ❌ Bad for:
// No way to access by index
// No key-value pairs (values only)
```

**In this project:**

```typescript
// Array - For ordered lists
const students = await studentRepository.findAll();

// Map - For quick lookups (avoiding N+1 queries)
const teacherMap = new Map();
teachers.forEach(t => teacherMap.set(t.email, t));

// Set - For duplicate detection
const emailSet = new Set(students.map(s => s.email));
if (emailSet.size !== students.length) {
  // Duplicates exist!
}

// Object - For simple config/grouping
const config = {
  host: 'localhost',
  port: 3306,
};
```

**Decision Tree:**

1. **Need order and index access?** → Use **Array**
2. **Need fast lookups by key?**
   - String keys only? → Use **Object**
   - Any type of keys? → Use **Map**
3. **Need unique values only?** → Use **Set**
4. **Need to count occurrences?** → Use **Map<T, number>**
5. **Need to group by property?** → Use **Map<K, V[]>** or **Object**

**Trade-off Summary:**

- **Array:** Best for ordered data, worst for searching
- **Object:** Fast lookups, but limited to string keys
- **Map:** Most flexible, slightly more memory
- **Set:** Best for uniqueness, can't access by index"

---

**71. How would you debug a "Maximum call stack size exceeded" error?**

- **Answer:** "This error means you have infinite recursion or very deep recursion. Here's my debugging approach:

**Step 1: Understand the cause**

```javascript
// Example of infinite recursion:
function badRecursion(n) {
  return badRecursion(n); // ❌ No base case!
}
badRecursion(5); // Stack overflow!
```

**Step 2: Check for a missing base case**

```javascript
// ❌ Bad: Missing base case
function factorial(n) {
  return n * factorial(n - 1); // Never stops!
}

// ✅ Good: Has base case
function factorial(n) {
  if (n <= 1) return 1; // Base case stops recursion
  return n * factorial(n - 1);
}
```

**Step 3: Check for incorrect recursive calls**

```javascript
// ❌ Bad: Wrong recursion (n doesn't decrease)
function countdown(n) {
  console.log(n);
  if (n === 0) return;
  countdown(n); // Should be countdown(n - 1)!
}

// ✅ Good: Decreases toward base case
function countdown(n) {
  console.log(n);
  if (n === 0) return;
  countdown(n - 1);
}
```

**Step 4: Add debugging logs**

```javascript
function debugRecursion(n, depth = 0) {
  console.log(`Depth: ${depth}, n: ${n}`);

  if (depth > 100) {
    throw new Error(`Recursion too deep! n = ${n}`);
  }

  if (n <= 0) return 0;
  return n + debugRecursion(n - 1, depth + 1);
}
```

**Step 5: Convert to iteration if possible**

```javascript
// Recursive (can cause stack overflow for large n)
function sumRecursive(n) {
  if (n <= 0) return 0;
  return n + sumRecursive(n - 1);
}

// Iterative (no stack overflow risk)
function sumIterative(n) {
  let sum = 0;
  for (let i = 1; i <= n; i++) {
    sum += i;
  }
  return sum;
}
```

**Step 6: Use tail recursion optimization (if supported)**

```javascript
// Not tail-recursive (operation after recursive call)
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // Multiplication happens AFTER recursion
}

// Tail-recursive (no operation after recursive call)
function factorialTail(n, accumulator = 1) {
  if (n <= 1) return accumulator;
  return factorialTail(n - 1, n * accumulator); // Recursion is LAST operation
}
```

**Common scenarios in this project:**

```typescript
// Potential issue: Circular references in models
class Student {
  class: Class; // Has reference to Class
}

class Class {
  students: Student[]; // Has reference to Students
}

// When serializing to JSON:
JSON.stringify(student);
// ❌ Can cause stack overflow if circular reference exists!

// ✅ Solution: Use JSON with proper handling
JSON.stringify(student, (key, value) => {
  if (key === 'class' && value?.students) {
    return { ...value, students: undefined }; // Break circular reference
  }
  return value;
});
```

**Interview Tip:**

1. Check for missing base case
2. Verify recursive call progresses toward base case
3. Consider converting to iteration
4. For deep recursion, consider tail recursion or iteration"

---
