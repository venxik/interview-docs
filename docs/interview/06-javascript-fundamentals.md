# JavaScript Fundamentals

This guide covers essential JavaScript concepts that are fundamental to understanding the project.

---

## Topics Covered

- Closures and scope
- Promises and async/await
- Event loop and call stack
- var, let, and const
- this keyword and binding
- Hoisting
- Array methods (map, filter, reduce)
- Equality operators (== vs ===)
- Destructuring and spread/rest operators

---

## **Category 15: JavaScript Technical Concepts**

These questions test your core JavaScript knowledge, which is fundamental for any Node.js/TypeScript developer.

---

**48. Explain closures in JavaScript. Can you provide an example from this project?**

- **Answer:** "A closure is a function that has access to variables in its outer (enclosing) function's scope, even after the outer function has finished executing. This happens because JavaScript maintains a reference to the outer scope.

**Simple Example:**

```javascript
function createCounter() {
  let count = 0; // Outer scope variable

  return function() { // Inner function (closure)
    count++; // Accesses outer scope variable
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
// 'count' is preserved even though createCounter() finished!
```

**Example from this project:**

While not explicitly used for closures, our middleware pattern demonstrates closure-like behavior:

```typescript
// src/api/middleware/rateLimiter.ts
export const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // This value is "closed over"
  max: 100,
  handler: (req: Request, res: Response) => {
    // This handler function has access to windowMs and max
    // even when it's executed later by Express
    Logger.warn(`Rate limit exceeded for IP: ${req.ip}`);
  }
});
```

**Why closures matter:**

1. ✅ **Data Privacy:** Variables are private to the closure
2. ✅ **State Preservation:** Maintain state between function calls
3. ✅ **Factory Functions:** Create multiple instances with their own state
4. ✅ **Callbacks:** Maintain access to outer scope in async operations"

---

**49. What is the difference between `var`, `let`, and `const`?**

- **Answer:** "These are three ways to declare variables, with different scoping and mutability rules:

| Feature | `var` | `let` | `const` |
|---------|-------|-------|---------|
| **Scope** | Function-scoped | Block-scoped | Block-scoped |
| **Hoisting** | Yes (initialized as `undefined`) | Yes (but in TDZ) | Yes (but in TDZ) |
| **Re-declaration** | Allowed | Not allowed | Not allowed |
| **Re-assignment** | Allowed | Allowed | Not allowed |
| **Temporal Dead Zone** | No | Yes | Yes |

**Function Scope vs Block Scope:**

```javascript
// var is function-scoped
function testVar() {
  if (true) {
    var x = 10;
  }
  console.log(x); // ✅ 10 (accessible outside if block)
}

// let/const are block-scoped
function testLet() {
  if (true) {
    let y = 10;
  }
  console.log(y); // ❌ ReferenceError: y is not defined
}
```

**Hoisting Behavior:**

```javascript
console.log(a); // undefined (var is hoisted and initialized)
var a = 5;

console.log(b); // ❌ ReferenceError (let is in TDZ)
let b = 5;
```

**In this project:**

We exclusively use `const` and `let` (TypeScript best practice):

```typescript
// src/business/DataUploadService.ts:125
const teachers = await teacherRepository.findByEmails(teacherEmails);
// ↑ Use const for values that won't be reassigned

// src/business/DataUploadService.ts:200
let errors: string[] = [];
// ↑ Use let when value will be reassigned

// ❌ Never use var in modern TypeScript/JavaScript
```

**Best Practice:**

1. Default to `const` (immutability)
2. Use `let` only when reassignment is needed
3. Never use `var` in modern code"

---

**50. Explain Promises and async/await. How do they work?**

- **Answer:** "Promises and async/await are mechanisms for handling asynchronous operations in JavaScript.

**Promises:**

A Promise is an object representing the eventual completion (or failure) of an asynchronous operation. It has three states:

1. **Pending:** Initial state, operation is ongoing
2. **Fulfilled:** Operation completed successfully
3. **Rejected:** Operation failed

```javascript
// Creating a Promise
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve({ data: 'Success!' }); // Fulfills the promise
    } else {
      reject(new Error('Failed!')); // Rejects the promise
    }
  }, 1000);
});

// Consuming a Promise
myPromise
  .then(result => console.log(result.data)) // Handles success
  .catch(error => console.error(error)) // Handles failure
  .finally(() => console.log('Done')); // Runs regardless
```

**Async/Await:**

`async/await` is syntactic sugar over Promises, making asynchronous code look synchronous:

```javascript
// With Promises (harder to read)
function getUser() {
  return fetch('/api/user')
    .then(response => response.json())
    .then(user => {
      return fetch(`/api/posts/${user.id}`);
    })
    .then(response => response.json())
    .then(posts => console.log(posts))
    .catch(error => console.error(error));
}

// With async/await (cleaner)
async function getUser() {
  try {
    const response = await fetch('/api/user');
    const user = await response.json();
    const postsResponse = await fetch(`/api/posts/${user.id}`);
    const posts = await postsResponse.json();
    console.log(posts);
  } catch (error) {
    console.error(error);
  }
}
```

**In this project (extensive usage):**

```typescript
// src/business/ClassService.ts:11
async getAllClasses(): Promise<Class[]> {
  // ↑ async function returns Promise<Class[]>
  return await this.classRepository.findAll();
  // ↑ await pauses execution until Promise resolves
}

// src/api/controllers/ClassController.ts:14
async getAllClasses(req: Request, res: Response, next: NextFunction) {
  try {
    const classes = await classService.getAllClasses();
    // ↑ Waits for Promise to resolve before continuing
    res.status(200).json(classes);
  } catch (error) {
    next(error); // If Promise rejects, catch block handles it
  }
}
```

**Key Rules:**

1. ✅ `await` can only be used inside `async` functions
2. ✅ `async` functions always return a Promise
3. ✅ Use `try/catch` to handle errors with async/await
4. ✅ Await pauses execution of the async function, not the entire program"

---

**51. What is the JavaScript Event Loop? How does Node.js handle asynchronous operations?**

- **Answer:** "The Event Loop is the mechanism that allows JavaScript to perform non-blocking I/O operations despite being single-threaded.

**How it works:**

```
┌───────────────────────────┐
│        Call Stack         │ ← Executes synchronous code
└───────────────────────────┘
              ↓
┌───────────────────────────┐
│      Web APIs / Node APIs │ ← Handles async operations
│   (setTimeout, fetch, fs) │   (timers, I/O, network)
└───────────────────────────┘
              ↓
┌───────────────────────────┐
│      Callback Queue       │ ← Queues completed callbacks
└───────────────────────────┘
              ↓
┌───────────────────────────┐
│       Event Loop          │ ← Moves callbacks to stack
└───────────────────────────┘
```

**Step-by-step execution:**

```javascript
console.log('1: Start');

setTimeout(() => {
  console.log('2: Timeout');
}, 0);

Promise.resolve().then(() => {
  console.log('3: Promise');
});

console.log('4: End');

// Output:
// 1: Start
// 4: End
// 3: Promise
// 2: Timeout
```

**Why this order?**

1. `console.log('1: Start')` → Call Stack (synchronous)
2. `setTimeout()` → Sent to Web API, registered callback
3. `Promise.resolve().then()` → Adds to **Microtask Queue** (higher priority)
4. `console.log('4: End')` → Call Stack (synchronous)
5. Call Stack empty → Event Loop checks **Microtask Queue** first
6. `console.log('3: Promise')` → Executed
7. Event Loop checks **Callback Queue** (lower priority)
8. `console.log('2: Timeout')` → Executed

**Two types of queues:**

1. **Microtask Queue** (higher priority):
   - Promise callbacks (`.then()`, `.catch()`, `.finally()`)
   - `process.nextTick()` in Node.js

2. **Callback Queue** (lower priority):
   - `setTimeout()`, `setInterval()`
   - I/O operations
   - UI rendering (in browsers)

**In this project (Node.js):**

```typescript
// src/server.ts:21
await dbConnection.authenticate();
// ↑ This is async! Event loop handles it:
// 1. Database connection is initiated
// 2. Node.js delegates to C++ bindings (libuv)
// 3. Event loop continues executing other code
// 4. When DB responds, callback is queued
// 5. Event loop picks it up and resolves the Promise

// Meanwhile, other code can run:
app.listen(port, () => {
  Logger.info(`Server is running on port ${port}`);
});
```

**Why this matters:**

1. ✅ **Non-blocking I/O:** Node.js can handle thousands of concurrent connections
2. ✅ **Single-threaded:** Simpler than multi-threaded models
3. ✅ **Performance:** Efficient for I/O-heavy operations (APIs, databases)
4. ⚠️ **CPU-intensive tasks block:** Heavy computation blocks the event loop"

---

**52. What is `this` in JavaScript? How does it differ in arrow functions vs regular functions?**

- **Answer:** "`this` refers to the context in which a function is executed. Its value depends on **how** the function is called.

**Regular Functions:**

`this` is determined by the **call-site** (how the function is invoked):

```javascript
const obj = {
  name: 'Alice',
  greet: function() {
    console.log(this.name); // 'this' refers to obj
  }
};

obj.greet(); // 'Alice'

const greetFunc = obj.greet;
greetFunc(); // undefined (this is now global object or undefined in strict mode)
```

**Arrow Functions:**

Arrow functions **don't have their own `this`**. They inherit `this` from the enclosing scope (lexical `this`):

```javascript
const obj = {
  name: 'Alice',
  greet: function() {
    const inner = () => {
      console.log(this.name); // 'this' inherited from greet()
    };
    inner();
  }
};

obj.greet(); // 'Alice'
```

**Comparison Table:**

| Feature | Regular Function | Arrow Function |
|---------|------------------|----------------|
| **`this` binding** | Dynamic (call-site) | Lexical (enclosing scope) |
| **Can be constructor** | Yes (`new Func()`) | No (TypeError) |
| **`arguments` object** | Yes | No (use `...args`) |
| **Best for** | Methods, constructors | Callbacks, inline functions |

**In this project:**

```typescript
// src/database/repositories/BaseRepository.ts:29
async findAll(): Promise<T[]> {
  return await this.model.findAll();
  // ↑ Regular method: 'this' refers to the repository instance
}

// src/api/middleware/rateLimiter.ts:439
handler: (req: Request, res: Response) => {
  Logger.warn(`Rate limit exceeded for IP: ${req.ip}`);
  // ↑ Arrow function: 'this' would inherit from outer scope
  // (not used here, but good for maintaining context)
}
```

**Common pitfall:**

```typescript
class Counter {
  count = 0;

  // ❌ Regular function loses 'this' when passed as callback
  incrementBad() {
    this.count++;
  }

  // ✅ Arrow function preserves 'this'
  incrementGood = () => {
    this.count++;
  }
}

const counter = new Counter();
setTimeout(counter.incrementBad, 1000); // ❌ 'this' is undefined
setTimeout(counter.incrementGood, 1000); // ✅ Works!
```

**Best Practice:**

1. Use regular functions for methods and constructors
2. Use arrow functions for callbacks and when you need to preserve `this`
3. In TypeScript classes, use arrow properties when passing methods as callbacks"

---

**53. Explain hoisting in JavaScript.**

- **Answer:** "Hoisting is JavaScript's behavior of moving variable and function **declarations** to the top of their scope during the compilation phase, before code execution.

**Function Hoisting:**

```javascript
// This works!
greet(); // 'Hello!'

function greet() {
  console.log('Hello!');
}
// ↑ Function declaration is hoisted (fully)
```

**Variable Hoisting (var):**

```javascript
console.log(x); // undefined (not ReferenceError!)
var x = 5;
console.log(x); // 5

// What JavaScript actually does:
var x; // Declaration hoisted
console.log(x); // undefined
x = 5; // Assignment stays in place
console.log(x); // 5
```

**Variable Hoisting (let/const):**

```javascript
console.log(y); // ❌ ReferenceError: Cannot access 'y' before initialization
let y = 5;

// Why? Temporal Dead Zone (TDZ)
// let/const ARE hoisted, but remain uninitialized until declaration line
```

**Function Expressions (NOT hoisted):**

```javascript
greet(); // ❌ TypeError: greet is not a function

var greet = function() {
  console.log('Hello!');
};

// Why? Only the variable declaration is hoisted:
var greet; // Hoisted
greet(); // greet is undefined at this point
greet = function() { ... }; // Assignment happens here
```

**Visualizing Hoisting:**

```javascript
// What you write:
console.log(a);
var a = 10;
hello();
var hello = function() { console.log('Hi'); };
world();
function world() { console.log('World'); }

// What JavaScript executes:
function world() { console.log('World'); } // Function declaration hoisted
var a; // var declaration hoisted
var hello; // var declaration hoisted
console.log(a); // undefined
a = 10;
hello(); // TypeError (hello is undefined)
hello = function() { console.log('Hi'); };
world(); // 'World' (works!)
```

**In this project:**

We avoid hoisting issues by using TypeScript/ES6+ best practices:

```typescript
// ✅ Import statements (always at top)
import { Request, Response } from 'express';

// ✅ Use const/let (block-scoped, TDZ prevents issues)
const classService = new ClassService();

// ✅ Class methods (clear structure)
export class ClassService {
  async getAllClasses() { ... }
}

// ❌ No var declarations
// ❌ No reliance on hoisting behavior
```

**Key Takeaways:**

1. **Function declarations** → Fully hoisted (can call before declaration)
2. **var** → Declaration hoisted, initialized as `undefined`
3. **let/const** → Hoisted but in TDZ (cannot access before declaration)
4. **Function expressions** → NOT hoisted (treated as variable assignment)
5. **Best practice:** Declare variables at the top of their scope to avoid confusion"

---

**54. What are JavaScript Array methods like `map()`, `filter()`, `reduce()`? When would you use each?**

- **Answer:** "These are higher-order functions that operate on arrays without mutating the original array.

**`map()` - Transform each element:**

```javascript
// Use when: You want to transform each element
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8]
console.log(numbers); // [1, 2, 3, 4] (original unchanged)
```

**`filter()` - Select elements that meet a condition:**

```javascript
// Use when: You want to keep only some elements
const numbers = [1, 2, 3, 4, 5, 6];
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4, 6]
```

**`reduce()` - Reduce array to a single value:**

```javascript
// Use when: You want to accumulate/aggregate values
const numbers = [1, 2, 3, 4];
const sum = numbers.reduce((accumulator, current) => {
  return accumulator + current;
}, 0); // 0 is the initial value
console.log(sum); // 10

// How it works:
// 1st call: accumulator = 0, current = 1 → returns 1
// 2nd call: accumulator = 1, current = 2 → returns 3
// 3rd call: accumulator = 3, current = 3 → returns 6
// 4th call: accumulator = 6, current = 4 → returns 10
```

**Other useful methods:**

```javascript
// find() - First element that matches
const users = [{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }];
const user = users.find(u => u.id === 2);
console.log(user); // { id: 2, name: 'Bob' }

// some() - Does at least one element match?
const hasEven = [1, 3, 5, 6].some(num => num % 2 === 0);
console.log(hasEven); // true

// every() - Do all elements match?
const allEven = [2, 4, 6].every(num => num % 2 === 0);
console.log(allEven); // true

// forEach() - Execute function for each element (no return value)
[1, 2, 3].forEach(num => console.log(num)); // Logs: 1, 2, 3
```

**In this project:**

```typescript
// src/business/DataUploadService.ts:176-178
const subjectsWithCode = row.subjects
  .split(",")
  .map((code) => code.trim()); // Transform: split string and trim each
// ↑ Use map() to clean up each subject code

// src/business/DataUploadService.ts:270
const emailSet = new Set(students.map((s) => s.email));
// ↑ Use map() to extract just emails from student objects

// src/business/DataUploadService.ts:183-186
const subjectIds = foundSubjects
  .filter((s) => s !== null) // Filter: keep only valid subjects
  .map((s) => s!.id); // Transform: extract IDs
// ↑ Chaining: filter out nulls, then map to IDs

// Hypothetical reduce() example:
const totalStudents = classes.reduce((total, cls) => {
  return total + cls.studentCount;
}, 0);
```

**Comparison Table:**

| Method | Input | Output | Use Case |
|--------|-------|--------|----------|
| `map()` | Array | Array (same length) | Transform each element |
| `filter()` | Array | Array (≤ length) | Select matching elements |
| `reduce()` | Array | Single value | Aggregate/accumulate |
| `find()` | Array | Single element or undefined | Find first match |
| `some()` | Array | Boolean | Test if any match |
| `every()` | Array | Boolean | Test if all match |

**Best Practice:**

1. ✅ Prefer these methods over `for` loops (more declarative)
2. ✅ Chain methods for complex transformations
3. ✅ They don't mutate the original array
4. ⚠️ Remember they create new arrays (memory consideration for large datasets)"

---

**55. What is the difference between `==` and `===` in JavaScript?**

- **Answer:** "These are equality operators with different comparison behaviors:

**`==` (Loose Equality) - Performs type coercion:**

```javascript
5 == '5'        // true (string '5' coerced to number)
true == 1       // true (boolean coerced to number)
null == undefined // true (special case)
0 == false      // true (both coerced to 0)
'' == false     // true (both coerced to 0)
[] == false     // true (array coerced to '')
```

**`===` (Strict Equality) - No type coercion:**

```javascript
5 === '5'       // false (different types)
true === 1      // false (different types)
null === undefined // false (different types)
0 === false     // false (different types)
'' === ''       // true (same type and value)
[] === []       // false (different object references)
```

**Type Coercion Rules (confusing!):**

```javascript
// These are all true with ==, but false with ===
console.log('' == '0');          // false
console.log(0 == '');            // true  ← Confusing!
console.log(0 == '0');           // true  ← Confusing!
console.log(false == 'false');   // false
console.log(false == '0');       // true  ← Confusing!
console.log(' \t\r\n ' == 0);    // true  ← Very confusing!
```

**Comparison Table:**

| Feature | `==` | `===` |
|---------|------|-------|
| **Name** | Loose/Abstract Equality | Strict Equality |
| **Type Coercion** | Yes | No |
| **Checks Type** | No | Yes |
| **Checks Value** | Yes (after coercion) | Yes |
| **Recommended** | ❌ Avoid | ✅ Use always |

**Object Comparison (both operators):**

```javascript
const obj1 = { name: 'Alice' };
const obj2 = { name: 'Alice' };
const obj3 = obj1;

console.log(obj1 == obj2);  // false (different references)
console.log(obj1 === obj2); // false (different references)
console.log(obj1 === obj3); // true (same reference)
```

**In this project:**

```typescript
// We always use === (TypeScript/ESLint enforces this)

// src/business/DataUploadService.ts:265
if (foundSubjects.length !== subjectsWithCode.length) {
  // ↑ Strict equality (===) is implied in !== operator
}

// src/database/repositories/BaseRepository.ts:54
const data = instance.get({ plain: true });
if (data === null) { // ✅ Strict equality
  return null;
}

// ❌ Never use == in modern TypeScript/JavaScript code
```

**null vs undefined:**

```javascript
let x;
console.log(x == null);   // true (undefined == null)
console.log(x === null);  // false (undefined !== null)
console.log(x === undefined); // true

// Best practice for checking null/undefined:
if (x == null) { ... }    // Checks both null and undefined
// or
if (x === null || x === undefined) { ... } // More explicit
// or (TypeScript)
if (x) { ... } // Falsy check (also catches 0, '', false)
```

**Best Practice:**

1. ✅ **Always use `===`** (and `!==`) in TypeScript/JavaScript
2. ❌ Avoid `==` (and `!=`) - unpredictable type coercion
3. ✅ ESLint rule: `eqeqeq: ['error', 'always']`
4. ✅ TypeScript strict mode helps prevent issues"

---

**56. What is destructuring in JavaScript? Provide examples.**

- **Answer:** "Destructuring is a syntax for extracting values from arrays or properties from objects into distinct variables.

**Object Destructuring:**

```javascript
// Without destructuring
const user = { name: 'Alice', age: 25, city: 'NYC' };
const name = user.name;
const age = user.age;

// With destructuring
const { name, age } = user;
console.log(name); // 'Alice'
console.log(age);  // 25

// Renaming variables
const { name: userName, age: userAge } = user;
console.log(userName); // 'Alice'

// Default values
const { name, country = 'USA' } = user;
console.log(country); // 'USA' (user.country doesn't exist)

// Nested destructuring
const user = {
  name: 'Alice',
  address: { city: 'NYC', zip: '10001' }
};
const { address: { city, zip } } = user;
console.log(city); // 'NYC'
```

**Array Destructuring:**

```javascript
// Without destructuring
const numbers = [1, 2, 3, 4, 5];
const first = numbers[0];
const second = numbers[1];

// With destructuring
const [first, second] = numbers;
console.log(first);  // 1
console.log(second); // 2

// Skip elements
const [first, , third] = numbers;
console.log(third); // 3

// Rest operator
const [first, ...rest] = numbers;
console.log(rest); // [2, 3, 4, 5]

// Swapping variables
let a = 1, b = 2;
[a, b] = [b, a];
console.log(a, b); // 2, 1
```

**Function Parameter Destructuring:**

```javascript
// Object parameters
function greet({ name, age }) {
  console.log(`Hello ${name}, you are ${age} years old`);
}
greet({ name: 'Alice', age: 25 }); // Hello Alice, you are 25 years old

// With default values
function createUser({ name, age = 18, role = 'user' }) {
  return { name, age, role };
}
createUser({ name: 'Bob' }); // { name: 'Bob', age: 18, role: 'user' }
```

**In this project (extensive usage):**

```typescript
// Import destructuring
import { Request, Response, NextFunction } from 'express';
// ↑ Destructuring named exports

// src/api/controllers/ClassController.ts:21
async getClassByCode(req: Request, res: Response, next: NextFunction) {
  const { classCode } = req.params; // Destructuring req.params
  // ↑ Instead of: const classCode = req.params.classCode
}

// src/business/DataUploadService.ts:115
async processCsvData(filePath: string): Promise<ProcessResult> {
  const { teachers, students, classes, subjects } = parsedData;
  // ↑ Destructuring returned object
}

// src/database/models/index.ts:42
const {
  DB_HOST = "localhost",
  DB_PORT = "3306",
  DB_NAME = "school",
  DB_USER = "root",
  DB_PASSWORD = "",
} = process.env;
// ↑ Destructuring with default values

// Array destructuring in tests
const [firstClass, secondClass] = await classRepository.findAll();
```

**Rest vs Spread (related syntax):**

```javascript
// Rest (gather)
const { name, ...otherProps } = { name: 'Alice', age: 25, city: 'NYC' };
console.log(otherProps); // { age: 25, city: 'NYC' }

// Spread (scatter)
const user = { name: 'Alice', age: 25 };
const updatedUser = { ...user, age: 26 }; // Create new object
console.log(updatedUser); // { name: 'Alice', age: 26 }
```

**Best Practice:**

1. ✅ Use destructuring for cleaner code
2. ✅ Especially useful in function parameters
3. ✅ Combine with default values for optional parameters
4. ✅ Makes imports cleaner (`import { X, Y } from ...`)
5. ⚠️ Don't over-nest (hard to read)"

---

**57. What is the spread operator (`...`)? How is it different from rest parameters?**

- **Answer:** "The `...` syntax serves two purposes depending on context:

**1. Spread Operator (expanding/scattering):**

Expands an iterable (array, object) into individual elements:

```javascript
// Array spread
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Copying arrays
const original = [1, 2, 3];
const copy = [...original]; // Creates shallow copy
copy.push(4);
console.log(original); // [1, 2, 3] (unchanged)

// Object spread
const user = { name: 'Alice', age: 25 };
const updatedUser = { ...user, age: 26, city: 'NYC' };
// { name: 'Alice', age: 26, city: 'NYC' }

// Function arguments
const numbers = [1, 2, 3];
Math.max(...numbers); // Same as Math.max(1, 2, 3)
```

**2. Rest Parameters (gathering/collecting):**

Collects multiple elements into an array:

```javascript
// Function rest parameters
function sum(...numbers) { // Gather all arguments into 'numbers' array
  return numbers.reduce((total, num) => total + num, 0);
}
sum(1, 2, 3, 4); // 10

// Destructuring rest
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(rest); // [3, 4, 5]

const { name, ...otherProps } = { name: 'Alice', age: 25, city: 'NYC' };
console.log(otherProps); // { age: 25, city: 'NYC' }
```

**Comparison Table:**

| Feature | Spread Operator | Rest Parameters |
|---------|----------------|-----------------|
| **Direction** | Expands/scatters | Collects/gathers |
| **Context** | Function calls, array/object literals | Function parameters, destructuring |
| **Result** | Individual elements | Array/object |
| **Example** | `[...arr]` | `function(...args)` |

**Visual Explanation:**

```javascript
// Spread: ONE → MANY
const arr = [1, 2, 3];
console.log(...arr); // 1 2 3 (three separate arguments)

// Rest: MANY → ONE
function log(...args) { // Many arguments → one array
  console.log(args); // [1, 2, 3]
}
log(1, 2, 3);
```

**In this project:**

```typescript
// Spread operator usage

// src/database/models/index.ts (likely pattern)
const config = {
  ...baseConfig,
  dialect: 'mysql',
}; // Merge objects

// Hypothetical: Creating new objects with updates
const updatedStudent = {
  ...existingStudent,
  name: 'New Name',
}; // Keep all properties, override name

// Rest parameters (not heavily used, but could be):
function logErrors(...errors: string[]) {
  errors.forEach(err => Logger.error(err));
}
logErrors('Error 1', 'Error 2', 'Error 3');
```

**Common Use Cases:**

```javascript
// 1. Cloning arrays/objects (shallow)
const arrCopy = [...original];
const objCopy = { ...original };

// 2. Merging arrays/objects
const merged = [...arr1, ...arr2];
const combined = { ...obj1, ...obj2 }; // obj2 overwrites obj1's properties

// 3. Adding elements
const newArr = [...oldArr, newElement];
const newObj = { ...oldObj, newKey: 'value' };

// 4. Function with variable arguments
function myFunc(...args) {
  // args is an array
}

// 5. Converting NodeList to Array
const divs = [...document.querySelectorAll('div')];
```

**Best Practice:**

1. ✅ Use spread for immutable updates (don't mutate original)
2. ✅ Use rest parameters for flexible function arguments
3. ⚠️ Spread creates **shallow** copies (nested objects are still references)
4. ✅ Prefer spread over `Array.concat()` or `Object.assign()`"

---
