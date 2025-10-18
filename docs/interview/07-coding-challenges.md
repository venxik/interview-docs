# Coding Challenges

This guide contains algorithm problems and coding challenges with detailed solutions.

---

## Topics Covered

- Array manipulation (duplicates, grouping, sorting)
- String processing (CSV parsing, validation)
- Algorithm implementation
- Code optimization techniques
- Real-world problem-solving

---

## **Category 16: Coding & Design Challenges**

These questions test your practical problem-solving skills through coding exercises and system design scenarios.

---

**58. Write a function that finds duplicate emails in an array of student objects.**

- **Answer:**

```typescript
interface Student {
  id: number;
  name: string;
  email: string;
}

// Solution 1: Using Set (O(n) time, O(n) space)
function findDuplicateEmails(students: Student[]): string[] {
  const seen = new Set<string>();
  const duplicates = new Set<string>();

  for (const student of students) {
    if (seen.has(student.email)) {
      duplicates.add(student.email);
    } else {
      seen.add(student.email);
    }
  }

  return Array.from(duplicates);
}

// Solution 2: Using Map to count occurrences
function findDuplicateEmailsWithCount(students: Student[]): Map<string, number> {
  const emailCount = new Map<string, number>();

  // Count occurrences
  for (const student of students) {
    emailCount.set(
      student.email,
      (emailCount.get(student.email) || 0) + 1
    );
  }

  // Filter duplicates (count > 1)
  const duplicates = new Map<string, number>();
  for (const [email, count] of emailCount.entries()) {
    if (count > 1) {
      duplicates.set(email, count);
    }
  }

  return duplicates;
}

// Solution 3: Using reduce (functional approach)
function findDuplicateEmailsFunctional(students: Student[]): string[] {
  const emailCount = students.reduce((acc, student) => {
    acc[student.email] = (acc[student.email] || 0) + 1;
    return acc;
  }, {} as Record<string, number>);

  return Object.keys(emailCount).filter(email => emailCount[email] > 1);
}

// Test cases
const students: Student[] = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
  { id: 3, name: 'Charlie', email: 'alice@example.com' }, // Duplicate
  { id: 4, name: 'David', email: 'david@example.com' },
  { id: 5, name: 'Eve', email: 'bob@example.com' }, // Duplicate
];

console.log(findDuplicateEmails(students));
// Output: ['alice@example.com', 'bob@example.com']
```

**Complexity Analysis:**

- **Time Complexity:** O(n) where n = number of students
- **Space Complexity:** O(n) for the Set/Map storage

**This is relevant to the project:** In `DataUploadService.ts:270`, we check for duplicate emails:

```typescript
const emailSet = new Set(students.map((s) => s.email));
if (emailSet.size !== students.length) {
  errors.push("Duplicate student emails detected");
}
```

---

**59. Implement a function to group students by their class.**

- **Answer:**

```typescript
interface Student {
  id: number;
  name: string;
  email: string;
  classCode: string;
}

// Solution 1: Using reduce
function groupStudentsByClass(students: Student[]): Record<string, Student[]> {
  return students.reduce((acc, student) => {
    if (!acc[student.classCode]) {
      acc[student.classCode] = [];
    }
    acc[student.classCode].push(student);
    return acc;
  }, {} as Record<string, Student[]>);
}

// Solution 2: Using Map (better type safety)
function groupStudentsByClassMap(students: Student[]): Map<string, Student[]> {
  const groups = new Map<string, Student[]>();

  for (const student of students) {
    if (!groups.has(student.classCode)) {
      groups.set(student.classCode, []);
    }
    groups.get(student.classCode)!.push(student);
  }

  return groups;
}

// Solution 3: Using forEach (imperative)
function groupStudentsByClassImperative(students: Student[]): Record<string, Student[]> {
  const groups: Record<string, Student[]> = {};

  students.forEach(student => {
    if (!groups[student.classCode]) {
      groups[student.classCode] = [];
    }
    groups[student.classCode].push(student);
  });

  return groups;
}

// Test case
const students: Student[] = [
  { id: 1, name: 'Alice', email: 'alice@example.com', classCode: 'P1-1' },
  { id: 2, name: 'Bob', email: 'bob@example.com', classCode: 'P1-2' },
  { id: 3, name: 'Charlie', email: 'charlie@example.com', classCode: 'P1-1' },
  { id: 4, name: 'David', email: 'david@example.com', classCode: 'P1-2' },
];

console.log(groupStudentsByClass(students));
// Output:
// {
//   'P1-1': [
//     { id: 1, name: 'Alice', email: 'alice@example.com', classCode: 'P1-1' },
//     { id: 3, name: 'Charlie', email: 'charlie@example.com', classCode: 'P1-1' }
//   ],
//   'P1-2': [
//     { id: 2, name: 'Bob', email: 'bob@example.com', classCode: 'P1-2' },
//     { id: 4, name: 'David', email: 'david@example.com', classCode: 'P1-2' }
//   ]
// }
```

**Complexity:**

- **Time Complexity:** O(n)
- **Space Complexity:** O(n)

**Interview Tip:** Mention that this is similar to SQL's `GROUP BY` operation, but done in-memory.

---

**60. Write a function to find the teacher with the most subjects assigned.**

- **Answer:**

```typescript
interface TeacherSubject {
  teacherId: number;
  teacherName: string;
  subjectCode: string;
}

// Solution 1: Using Map to count
function findTeacherWithMostSubjects(assignments: TeacherSubject[]): {
  teacherId: number;
  teacherName: string;
  subjectCount: number;
} | null {
  if (assignments.length === 0) return null;

  // Count subjects per teacher
  const teacherSubjectCount = new Map<number, { name: string; count: number }>();

  for (const assignment of assignments) {
    if (!teacherSubjectCount.has(assignment.teacherId)) {
      teacherSubjectCount.set(assignment.teacherId, {
        name: assignment.teacherName,
        count: 0,
      });
    }
    teacherSubjectCount.get(assignment.teacherId)!.count++;
  }

  // Find teacher with max count
  let maxTeacher = { teacherId: 0, teacherName: '', subjectCount: 0 };

  for (const [teacherId, data] of teacherSubjectCount.entries()) {
    if (data.count > maxTeacher.subjectCount) {
      maxTeacher = {
        teacherId,
        teacherName: data.name,
        subjectCount: data.count,
      };
    }
  }

  return maxTeacher;
}

// Solution 2: Using reduce
function findTeacherWithMostSubjectsReduce(assignments: TeacherSubject[]) {
  if (assignments.length === 0) return null;

  // Group and count
  const teacherCounts = assignments.reduce((acc, assignment) => {
    if (!acc[assignment.teacherId]) {
      acc[assignment.teacherId] = {
        name: assignment.teacherName,
        count: 0,
      };
    }
    acc[assignment.teacherId].count++;
    return acc;
  }, {} as Record<number, { name: string; count: number }>);

  // Find max
  const maxEntry = Object.entries(teacherCounts).reduce((max, [id, data]) => {
    return data.count > max.count
      ? { teacherId: Number(id), teacherName: data.name, subjectCount: data.count }
      : max;
  }, { teacherId: 0, teacherName: '', subjectCount: 0 });

  return maxEntry;
}

// Test case
const assignments: TeacherSubject[] = [
  { teacherId: 1, teacherName: 'Mr. Smith', subjectCode: 'MATH' },
  { teacherId: 1, teacherName: 'Mr. Smith', subjectCode: 'SCI' },
  { teacherId: 2, teacherName: 'Ms. Johnson', subjectCode: 'ENG' },
  { teacherId: 1, teacherName: 'Mr. Smith', subjectCode: 'HIST' },
  { teacherId: 3, teacherName: 'Dr. Lee', subjectCode: 'PHY' },
  { teacherId: 2, teacherName: 'Ms. Johnson', subjectCode: 'LIT' },
];

console.log(findTeacherWithMostSubjects(assignments));
// Output: { teacherId: 1, teacherName: 'Mr. Smith', subjectCount: 3 }
```

**Complexity:**

- **Time Complexity:** O(n) - single pass to count, single pass to find max
- **Space Complexity:** O(t) where t = number of unique teachers

---

**61. Reverse a string without using built-in reverse methods.**

- **Answer:**

```javascript
// Solution 1: Two-pointer swap
function reverseString1(str) {
  const arr = str.split('');
  let left = 0;
  let right = arr.length - 1;

  while (left < right) {
    // Swap
    [arr[left], arr[right]] = [arr[right], arr[left]];
    left++;
    right--;
  }

  return arr.join('');
}

// Solution 2: Using reduce
function reverseString2(str) {
  return str.split('').reduce((reversed, char) => char + reversed, '');
}

// Solution 3: Iterative (building from end)
function reverseString3(str) {
  let reversed = '';
  for (let i = str.length - 1; i >= 0; i--) {
    reversed += str[i];
  }
  return reversed;
}

// Solution 4: Recursive
function reverseString4(str) {
  if (str === '') return '';
  return reverseString4(str.substring(1)) + str[0];
}

// Test
console.log(reverseString1('hello')); // 'olleh'
console.log(reverseString2('world')); // 'dlrow'
console.log(reverseString3('TypeScript')); // 'tpircSepyT'
console.log(reverseString4('Node')); // 'edoN'
```

**Complexity Analysis:**

- **Solution 1:** O(n) time, O(n) space
- **Solution 2:** O(n) time, O(n) space
- **Solution 3:** O(n) time, O(n) space (string concatenation)
- **Solution 4:** O(n) time, O(n) space (recursion stack)

**Interview Note:** Solution 1 (two-pointer) is most commonly expected in interviews.

---

**62. Check if a string is a palindrome.**

- **Answer:**

```javascript
// Solution 1: Two-pointer (most efficient)
function isPalindrome1(str) {
  // Normalize: lowercase and remove non-alphanumeric
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  let left = 0;
  let right = cleaned.length - 1;

  while (left < right) {
    if (cleaned[left] !== cleaned[right]) {
      return false;
    }
    left++;
    right--;
  }

  return true;
}

// Solution 2: Reverse and compare
function isPalindrome2(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  const reversed = cleaned.split('').reverse().join('');
  return cleaned === reversed;
}

// Solution 3: Recursive
function isPalindrome3(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');

  function check(s, left, right) {
    if (left >= right) return true;
    if (s[left] !== s[right]) return false;
    return check(s, left + 1, right - 1);
  }

  return check(cleaned, 0, cleaned.length - 1);
}

// Test cases
console.log(isPalindrome1('racecar')); // true
console.log(isPalindrome1('hello')); // false
console.log(isPalindrome1('A man, a plan, a canal: Panama')); // true
console.log(isPalindrome1('race a car')); // false
```

**Complexity:**

- **Time Complexity:** O(n)
- **Space Complexity:** O(n) for cleaned string (O(1) if in-place comparison allowed)

---

**63. Find the first non-repeating character in a string.**

- **Answer:**

```javascript
// Solution 1: Using Map (O(n) time, O(n) space)
function firstNonRepeating1(str) {
  const charCount = new Map();

  // Count occurrences
  for (const char of str) {
    charCount.set(char, (charCount.get(char) || 0) + 1);
  }

  // Find first with count 1
  for (const char of str) {
    if (charCount.get(char) === 1) {
      return char;
    }
  }

  return null; // All characters repeat
}

// Solution 2: Using Object
function firstNonRepeating2(str) {
  const charCount = {};

  for (const char of str) {
    charCount[char] = (charCount[char] || 0) + 1;
  }

  for (const char of str) {
    if (charCount[char] === 1) {
      return char;
    }
  }

  return null;
}

// Test cases
console.log(firstNonRepeating1('leetcode')); // 'l'
console.log(firstNonRepeating1('loveleetcode')); // 'v'
console.log(firstNonRepeating1('aabb')); // null
```

**Why two passes?**

- First pass: Count all occurrences
- Second pass: Find first character with count = 1 (preserves order)

**Interview Tip:** Mention that you need two passes to maintain the original order.

---

**64. Design a workload report system. What data structures would you use?**

- **Answer:** "Based on this project's `WorkloadReportService`, here's how I would design it:

**Requirements:**

1. Show each teacher's name
2. List all classes they teach
3. Show how many classes per subject

**Data Structures:**

```typescript
// Input: From database queries
interface TeacherClassSubject {
  teacherId: number;
  teacherName: string;
  teacherEmail: string;
  classCode: string;
  className: string;
  subjectCode: string;
  subjectName: string;
}

// Output: Aggregated report
interface WorkloadReport {
  teacherName: string;
  teacherEmail: string;
  subjects: {
    subjectName: string;
    numberOfClasses: number;
  }[];
}

// Implementation approach:
function generateWorkloadReport(data: TeacherClassSubject[]): WorkloadReport[] {
  // Step 1: Group by teacher (using Map for O(1) lookups)
  const teacherMap = new Map<number, {
    name: string;
    email: string;
    subjects: Map<string, Set<string>>; // subjectName -> Set of classCodes
  }>();

  // Step 2: Populate the map
  for (const row of data) {
    if (!teacherMap.has(row.teacherId)) {
      teacherMap.set(row.teacherId, {
        name: row.teacherName,
        email: row.teacherEmail,
        subjects: new Map(),
      });
    }

    const teacher = teacherMap.get(row.teacherId)!;

    if (!teacher.subjects.has(row.subjectName)) {
      teacher.subjects.set(row.subjectName, new Set());
    }

    // Use Set to automatically handle duplicate classes
    teacher.subjects.get(row.subjectName)!.add(row.classCode);
  }

  // Step 3: Transform to output format
  const reports: WorkloadReport[] = [];

  for (const teacher of teacherMap.values()) {
    const subjectList = [];

    for (const [subjectName, classes] of teacher.subjects.entries()) {
      subjectList.push({
        subjectName,
        numberOfClasses: classes.size, // Set.size gives unique count
      });
    }

    reports.push({
      teacherName: teacher.name,
      teacherEmail: teacher.email,
      subjects: subjectList,
    });
  }

  return reports;
}
```

**Why these data structures?**

1. **Map<number, ...>** for teachers:
   - O(1) lookup by teacherId
   - Better than array for grouping operations

2. **Map<string, Set<string>>** for subjects:
   - Map for O(1) subject lookup
   - Set for automatic deduplication of class codes

3. **Set<string>** for classes:
   - Automatically handles duplicate class assignments
   - O(1) add operation
   - `.size` gives unique count

**Complexity:**

- **Time:** O(n) where n = number of rows
- **Space:** O(t × s × c) where t = teachers, s = subjects per teacher, c = classes per subject

**Alternative Approach (SQL):**

```sql
SELECT
  t.name AS teacherName,
  t.email AS teacherEmail,
  s.name AS subjectName,
  COUNT(DISTINCT tcs.class_id) AS numberOfClasses
FROM teachers t
JOIN teacher_class_subject tcs ON t.id = tcs.teacher_id
JOIN subjects s ON tcs.subject_id = s.id
GROUP BY t.id, s.id
ORDER BY t.name, s.name;
```

**Interview Tip:** Mention that you could solve this either:

1. **In-memory (JavaScript):** Good for additional processing/transformation
2. **In-database (SQL GROUP BY):** More efficient for large datasets"

---

**65. How would you implement a CSV parser from scratch?**

- **Answer:** "Here's a simplified CSV parser implementation:

```typescript
interface ParsedCSV {
  headers: string[];
  rows: Record<string, string>[];
}

function parseCSV(csvString: string): ParsedCSV {
  const lines = csvString.trim().split('\n');

  if (lines.length === 0) {
    return { headers: [], rows: [] };
  }

  // Parse header row
  const headers = parseLine(lines[0]);

  // Parse data rows
  const rows: Record<string, string>[] = [];

  for (let i = 1; i < lines.length; i++) {
    const values = parseLine(lines[i]);

    // Skip empty rows
    if (values.length === 0 || (values.length === 1 && values[0] === '')) {
      continue;
    }

    // Create object from headers and values
    const row: Record<string, string> = {};
    for (let j = 0; j < headers.length; j++) {
      row[headers[j]] = values[j] || ''; // Handle missing values
    }

    rows.push(row);
  }

  return { headers, rows };
}

function parseLine(line: string): string[] {
  const result: string[] = [];
  let current = '';
  let inQuotes = false;

  for (let i = 0; i < line.length; i++) {
    const char = line[i];

    if (char === '"') {
      // Handle escaped quotes ("")
      if (inQuotes && line[i + 1] === '"') {
        current += '"';
        i++; // Skip next quote
      } else {
        inQuotes = !inQuotes; // Toggle quote state
      }
    } else if (char === ',' && !inQuotes) {
      // End of field (only if not inside quotes)
      result.push(current.trim());
      current = '';
    } else {
      current += char;
    }
  }

  // Add last field
  result.push(current.trim());

  return result;
}

// Test cases
const csv1 = `name,email,age
Alice,alice@example.com,25
Bob,bob@example.com,30`;

console.log(parseCSV(csv1));
// Output:
// {
//   headers: ['name', 'email', 'age'],
//   rows: [
//     { name: 'Alice', email: 'alice@example.com', age: '25' },
//     { name: 'Bob', email: 'bob@example.com', age: '30' }
//   ]
// }

// Test with quoted fields
const csv2 = `name,address
Alice,"123 Main St, Apt 4"
Bob,"456 Oak Ave"`;

console.log(parseCSV(csv2));
// Handles commas inside quotes correctly
```

**Edge Cases to Handle:**

1. ✅ Quoted fields with commas: `"Main St, Apt 4"`
2. ✅ Escaped quotes: `"He said ""Hello"""`
3. ✅ Empty fields: `Alice,,25`
4. ✅ Trailing commas: `Alice,alice@example.com,`
5. ✅ Empty rows
6. ✅ Different line endings (`\r\n` vs `\n`)

**In this project:**

We use the `csv-parse` library (`src/utils/csvParser.ts`), but understanding how it works shows deeper knowledge:

```typescript
// Actual project usage (simplified):
import { parse } from 'csv-parse/sync';

const records = parse(fileContent, {
  columns: true, // Use first row as headers
  skip_empty_lines: true,
  trim: true,
});
```

**Complexity:**

- **Time:** O(n × m) where n = rows, m = average row length
- **Space:** O(n × m) for storing parsed data

**Interview Tip:** Mention that production CSV parsers must handle:

- Different encodings (UTF-8, Latin-1)
- Different delimiters (comma, semicolon, tab)
- Different quote characters
- BOM (Byte Order Mark) at file start
- Streaming for large files"

---
