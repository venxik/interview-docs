# Security Best Practices & Interview Questions

## Table of Contents
1. [OWASP Top 10 Security Risks](#owasp-top-10-security-risks)
2. [Authentication & Authorization](#authentication--authorization)
3. [Input Validation & Sanitization](#input-validation--sanitization)
4. [SQL Injection Prevention](#sql-injection-prevention)
5. [Cross-Site Scripting (XSS)](#cross-site-scripting-xss)
6. [Cross-Site Request Forgery (CSRF)](#cross-site-request-forgery-csrf)
7. [File Upload Security](#file-upload-security)
8. [Sensitive Data Protection](#sensitive-data-protection)
9. [API Security](#api-security)
10. [Database Security](#database-security)
11. [Error Handling & Logging](#error-handling--logging)
12. [Dependency Security](#dependency-security)
13. [Security Headers](#security-headers)
14. [Rate Limiting & DDoS Protection](#rate-limiting--ddos-protection)
15. [Common Security Interview Questions](#common-security-interview-questions)

---

## OWASP Top 10 Security Risks

### What is OWASP Top 10?
The **Open Web Application Security Project (OWASP) Top 10** is a standard awareness document representing the most critical security risks to web applications.

### 2021 OWASP Top 10:

1. **A01:2021 – Broken Access Control**
   - Users can act outside of their intended permissions
   - Example: Accessing another user's data by changing URL parameter

2. **A02:2021 – Cryptographic Failures**
   - Exposing sensitive data due to lack of encryption
   - Example: Storing passwords in plain text

3. **A03:2021 – Injection**
   - SQL, NoSQL, OS command injection
   - Example: SQL injection through unvalidated input

4. **A04:2021 – Insecure Design**
   - Missing or ineffective security controls
   - Example: No rate limiting on login endpoint

5. **A05:2021 – Security Misconfiguration**
   - Improperly configured security settings
   - Example: Default credentials, unnecessary features enabled

6. **A06:2021 – Vulnerable and Outdated Components**
   - Using libraries with known vulnerabilities
   - Example: Outdated npm packages with CVEs

7. **A07:2021 – Identification and Authentication Failures**
   - Broken authentication mechanisms
   - Example: Weak password requirements, no MFA

8. **A08:2021 – Software and Data Integrity Failures**
   - Code and infrastructure that doesn't protect against integrity violations
   - Example: Unsigned npm packages, auto-update without verification

9. **A09:2021 – Security Logging and Monitoring Failures**
   - Insufficient logging of security events
   - Example: Failed login attempts not logged

10. **A10:2021 – Server-Side Request Forgery (SSRF)**
    - Application fetches remote resource without validating URL
    - Example: User-supplied URL used in API call

---

## Authentication & Authorization

### Q1: What's the difference between Authentication and Authorization?

**Authentication**: Verifying WHO you are (identity)
**Authorization**: Verifying WHAT you can do (permissions)

**Example:**
```typescript
// Authentication - Verify user identity
async function authenticateUser(email: string, password: string): Promise<User> {
  const user = await User.findOne({ where: { email } });
  if (!user) throw new Error('User not found');

  const isValid = await bcrypt.compare(password, user.passwordHash);
  if (!isValid) throw new Error('Invalid password');

  return user; // ✅ User authenticated
}

// Authorization - Check user permissions
function authorizeTeacher(user: User, action: string): boolean {
  if (user.role !== 'TEACHER') {
    throw new Error('Unauthorized: Teacher role required');
  }
  return true; // ✅ User authorized
}
```

### Q2: How would you implement JWT authentication?

**JWT (JSON Web Token)** is a stateless authentication mechanism.

**Structure**: `header.payload.signature`

```typescript
import jwt from 'jsonwebtoken';

// Generate JWT token
function generateToken(user: User): string {
  const payload = {
    userId: user.id,
    email: user.email,
    role: user.role,
  };

  const secret = process.env.JWT_SECRET!;
  const options = {
    expiresIn: '24h', // Token expires in 24 hours
    issuer: 'school-admin-system',
  };

  return jwt.sign(payload, secret, options);
}

// Verify JWT token (middleware)
function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const token = authHeader.substring(7); // Remove 'Bearer '

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as JwtPayload;
    req.user = decoded; // Attach user info to request
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

// Usage in router
router.get('/api/teachers', authMiddleware, teacherController.list);
```

**Security Best Practices for JWT:**
- Store secret in environment variables (never hardcode)
- Use HTTPS to prevent token interception
- Set reasonable expiration times
- Don't store sensitive data in payload (it's base64 encoded, not encrypted)
- Implement token refresh mechanism
- Use strong signing algorithms (HS256, RS256)

### Q3: How would you store passwords securely?

**NEVER store plain text passwords!** Use bcrypt with salt.

```typescript
import bcrypt from 'bcrypt';

// Hash password (during registration)
async function hashPassword(plainPassword: string): Promise<string> {
  const saltRounds = 12; // Higher = more secure but slower
  const hash = await bcrypt.hash(plainPassword, saltRounds);
  return hash; // Store this in database
}

// Verify password (during login)
async function verifyPassword(plainPassword: string, hash: string): Promise<boolean> {
  return await bcrypt.compare(plainPassword, hash);
}

// Example in User model
class User extends Model {
  declare passwordHash: string;

  static async createUser(email: string, password: string) {
    const passwordHash = await hashPassword(password);
    return await User.create({ email, passwordHash });
  }

  async checkPassword(password: string): Promise<boolean> {
    return await verifyPassword(password, this.passwordHash);
  }
}
```

**Why bcrypt?**
- Includes salt automatically (prevents rainbow table attacks)
- Adaptive function (can increase rounds as computers get faster)
- Time-constant comparison (prevents timing attacks)

**Alternative**: Argon2 (winner of Password Hashing Competition)

---

## Input Validation & Sanitization

### Q4: Why is input validation important?

**Every input is potentially malicious until validated.**

Input validation prevents:
- SQL Injection
- XSS (Cross-Site Scripting)
- Command Injection
- Path Traversal
- Buffer Overflow

### Q5: What's the difference between validation and sanitization?

**Validation**: Checking if input meets criteria (reject if invalid)
**Sanitization**: Cleaning/modifying input to make it safe (modify then accept)

```typescript
// Validation - Reject invalid input
function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    throw new Error('Invalid email format');
  }
  return true;
}

// Sanitization - Clean input to make it safe
function sanitizeInput(input: string): string {
  // Remove HTML tags
  return input.replace(/<[^>]*>/g, '');
}

// Example from your project (csvValidator.ts:54-61)
if (row.teacherEmail && !EMAIL_REGEX.test(row.teacherEmail)) {
  errors.push({
    row: rowNumber,
    field: 'teacherEmail',
    value: row.teacherEmail,
    message: `Invalid email format: '${row.teacherEmail}'`,
  });
}
```

### Best Practices for Input Validation

```typescript
import validator from 'validator';

interface StudentInput {
  name: string;
  email: string;
  age: number;
}

function validateStudentInput(input: any): StudentInput {
  const errors: string[] = [];

  // Whitelist validation (better than blacklist)
  if (!input.name || typeof input.name !== 'string') {
    errors.push('Name must be a string');
  }
  if (input.name.length > 255) {
    errors.push('Name too long');
  }

  if (!validator.isEmail(input.email)) {
    errors.push('Invalid email');
  }

  if (!Number.isInteger(input.age) || input.age < 0 || input.age > 150) {
    errors.push('Age must be between 0 and 150');
  }

  if (errors.length > 0) {
    throw new Error(`Validation failed: ${errors.join(', ')}`);
  }

  return {
    name: validator.escape(input.name), // Sanitize
    email: validator.normalizeEmail(input.email)!,
    age: input.age,
  };
}
```

---

## SQL Injection Prevention

### Q6: What is SQL Injection and how do you prevent it?

**SQL Injection**: Malicious SQL code inserted through user input to manipulate database queries.

**Example Attack:**
```sql
-- Intended query
SELECT * FROM users WHERE email = 'user@example.com' AND password = 'pass123';

-- Malicious input: email = "admin@school.com' OR '1'='1"
SELECT * FROM users WHERE email = 'admin@school.com' OR '1'='1' AND password = 'pass123';
-- ↑ This returns all users because '1'='1' is always true
```

### Prevention Methods:

#### 1. **Use Parameterized Queries / Prepared Statements** ✅ (Best Practice)

```typescript
// ❌ VULNERABLE (string concatenation)
async function getUserByEmailVulnerable(email: string) {
  const query = `SELECT * FROM users WHERE email = '${email}'`;
  return await sequelize.query(query); // DANGEROUS!
}

// ✅ SAFE (parameterized query with Sequelize)
async function getUserByEmailSafe(email: string) {
  return await User.findOne({
    where: { email } // Sequelize automatically parameterizes
  });
}

// ✅ SAFE (raw query with parameters)
async function getUserByEmailRaw(email: string) {
  const query = 'SELECT * FROM users WHERE email = :email';
  return await sequelize.query(query, {
    replacements: { email }, // Safe parameter binding
    type: QueryTypes.SELECT,
  });
}
```

**Why it works**: Database treats parameters as data, not executable code.

#### 2. **Use ORM (Sequelize) Properly**

```typescript
// ✅ SAFE - ORM handles escaping
const students = await Student.findAll({
  where: {
    email: userInput, // Automatically parameterized
    name: {
      [Op.like]: `%${userInput}%` // Still safe with Op
    }
  }
});

// ❌ DANGEROUS - Direct SQL in where clause
const students = await Student.findAll({
  where: sequelize.literal(`email = '${userInput}'`) // VULNERABLE!
});
```

#### 3. **Input Validation** (Defense in Depth)

```typescript
function validateEmail(email: string): string {
  // Only allow specific characters
  if (!/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(email)) {
    throw new Error('Invalid email format');
  }
  return email;
}
```

#### 4. **Least Privilege Principle**

```sql
-- Database user should only have necessary permissions
-- Don't use root user in application!
GRANT SELECT, INSERT, UPDATE ON school_db.* TO 'app_user'@'localhost';
REVOKE DROP, DELETE ON school_db.* FROM 'app_user'@'localhost';
```

---

## Cross-Site Scripting (XSS)

### Q7: What is XSS and how do you prevent it?

**XSS (Cross-Site Scripting)**: Injecting malicious JavaScript into web pages viewed by other users.

### Types of XSS:

1. **Stored XSS**: Malicious script stored in database
2. **Reflected XSS**: Script reflected back from user input
3. **DOM-based XSS**: Vulnerability in client-side JavaScript

### Example Attack:

```javascript
// User submits this as their name:
const maliciousName = "<script>alert('XSS Attack!');</script>";

// If rendered without escaping:
<div>Welcome, <script>alert('XSS Attack!');</script></div>
// ↑ Script executes in victim's browser
```

### Prevention Methods:

#### 1. **Output Encoding/Escaping**

```typescript
// ❌ VULNERABLE
function renderWelcome(name: string) {
  return `<h1>Welcome, ${name}!</h1>`; // Dangerous!
}

// ✅ SAFE (HTML escape)
import DOMPurify from 'isomorphic-dompurify';

function renderWelcomeSafe(name: string) {
  const escaped = escapeHtml(name);
  return `<h1>Welcome, ${escaped}!</h1>`;
}

function escapeHtml(unsafe: string): string {
  return unsafe
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
}

// Example:
escapeHtml("<script>alert('xss')</script>")
// Returns: "&lt;script&gt;alert(&#039;xss&#039;)&lt;/script&gt;"
// Displays as text, not executed
```

#### 2. **Content Security Policy (CSP) Header**

```typescript
// In app.ts
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';"
  );
  next();
});
```

**CSP prevents**:
- Inline scripts from executing
- Scripts from untrusted domains
- eval() usage

#### 3. **HTTP-only Cookies**

```typescript
// Prevent JavaScript from accessing cookies
res.cookie('sessionId', token, {
  httpOnly: true, // ✅ Cookie not accessible via document.cookie
  secure: true,   // ✅ Only sent over HTTPS
  sameSite: 'strict', // ✅ CSRF protection
});
```

#### 4. **Input Validation**

```typescript
function validateStudentName(name: string): string {
  // Only allow letters, spaces, hyphens
  if (!/^[a-zA-Z\s-]+$/.test(name)) {
    throw new Error('Name contains invalid characters');
  }
  if (name.length > 255) {
    throw new Error('Name too long');
  }
  return name;
}
```

---

## Cross-Site Request Forgery (CSRF)

### Q8: What is CSRF and how do you prevent it?

**CSRF (Cross-Site Request Forgery)**: Attacker tricks victim's browser into making unwanted requests to a site where victim is authenticated.

### Example Attack:

```html
<!-- Attacker's malicious website -->
<img src="https://school-admin.com/api/students/123/delete" />

<!-- When victim (logged-in admin) visits attacker's site,
     their browser automatically sends cookies,
     and the delete request succeeds! -->
```

### Prevention Methods:

#### 1. **CSRF Tokens** (Best for traditional web apps)

```typescript
import csrf from 'csurf';

// Setup CSRF protection
const csrfProtection = csrf({ cookie: true });

app.post('/api/students', csrfProtection, (req, res) => {
  // Request must include valid CSRF token
  // Token sent in: req.body._csrf or req.headers['csrf-token']
  studentController.create(req, res);
});

// Generate token for forms
app.get('/form', csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});
```

#### 2. **SameSite Cookie Attribute** ✅ (Modern approach)

```typescript
res.cookie('sessionId', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict', // ✅ Cookie NOT sent on cross-site requests
});
```

**Values**:
- `strict`: Never sent cross-site (most secure)
- `lax`: Sent on top-level navigation (GET only)
- `none`: Always sent (requires `secure: true`)

#### 3. **Double Submit Cookie Pattern**

```typescript
// Send random token in both cookie and request header
app.use((req, res, next) => {
  const token = req.cookies.csrfToken || generateRandomToken();
  res.cookie('csrfToken', token, { sameSite: 'strict' });

  if (['POST', 'PUT', 'DELETE'].includes(req.method)) {
    if (req.headers['x-csrf-token'] !== token) {
      return res.status(403).json({ error: 'Invalid CSRF token' });
    }
  }
  next();
});
```

#### 4. **Verify Origin/Referer Headers**

```typescript
function verifyOrigin(req: Request, res: Response, next: NextFunction) {
  const origin = req.headers.origin || req.headers.referer;
  const allowedOrigins = ['https://school-admin.com'];

  if (!origin || !allowedOrigins.some(allowed => origin.startsWith(allowed))) {
    return res.status(403).json({ error: 'Invalid origin' });
  }
  next();
}

app.post('/api/*', verifyOrigin, ...);
```

---

## File Upload Security

### Q9: What security risks exist with file uploads?

**Risks:**
1. Malicious file execution (uploaded PHP, JSP files)
2. Path traversal (../../etc/passwd)
3. DoS via large files
4. Malware/viruses
5. Content spoofing (fake MIME types)

### Security Measures in Your Project:

```typescript
// From multer.ts:7-43
const FILE_LIMIT = 5 * 1024 * 1024; // 5 MB ✅ Size limit

const fileFilter = (req, file, callback) => {
  // ✅ Validate MIME type
  if (file.mimetype === 'text/csv' ||
      file.originalname.toLowerCase().endsWith('.csv')) {
    callback(null, true);
  } else {
    callback(new Error('Invalid file type. Only .csv files are allowed.'));
  }
};

const upload = multer({
  storage: diskStorage,
  fileFilter: fileFilter,
  limits: { fileSize: FILE_LIMIT }, // ✅ Enforce size limit
});
```

### Best Practices:

```typescript
import path from 'path';
import crypto from 'crypto';

// 1. ✅ Validate file type by content (not just extension)
import fileType from 'file-type';

async function validateFileContent(filePath: string) {
  const type = await fileType.fromFile(filePath);
  if (type?.mime !== 'text/csv') {
    throw new Error('Invalid file content');
  }
}

// 2. ✅ Rename files to prevent path traversal
function generateSafeFilename(originalName: string): string {
  const ext = path.extname(originalName);
  const randomName = crypto.randomBytes(16).toString('hex');
  return `${randomName}${ext}`;
}

// 3. ✅ Store files outside web root
const UPLOAD_DIR = '/var/uploads'; // NOT in /public or /static

// 4. ✅ Scan files for viruses (in production)
import ClamScan from 'clamscan';

async function scanFile(filePath: string) {
  const clamscan = await new ClamScan().init();
  const { isInfected } = await clamscan.isInfected(filePath);
  if (isInfected) {
    fs.unlinkSync(filePath);
    throw new Error('Malicious file detected');
  }
}

// 5. ✅ Set proper permissions
fs.chmodSync(filePath, 0o600); // Read/write for owner only
```

### Complete Secure Upload Handler:

```typescript
import fs from 'fs/promises';
import path from 'path';

async function secureFileUpload(req: Request, res: Response) {
  try {
    const file = req.file;
    if (!file) {
      return res.status(400).json({ error: 'No file provided' });
    }

    // 1. Validate size
    if (file.size > 5 * 1024 * 1024) {
      await fs.unlink(file.path);
      return res.status(400).json({ error: 'File too large' });
    }

    // 2. Validate content type
    await validateFileContent(file.path);

    // 3. Generate safe filename
    const safeFilename = generateSafeFilename(file.originalname);
    const safePath = path.join(UPLOAD_DIR, safeFilename);

    // 4. Move file to secure location
    await fs.rename(file.path, safePath);

    // 5. Set restrictive permissions
    await fs.chmod(safePath, 0o600);

    // 6. Process file (parse CSV, etc.)
    const data = await processCsvFile(safePath);

    // 7. Delete file after processing
    await fs.unlink(safePath);

    res.json({ success: true, recordsProcessed: data.length });
  } catch (error) {
    // Clean up on error
    if (req.file?.path) {
      await fs.unlink(req.file.path).catch(() => {});
    }
    throw error;
  }
}
```

---

## Sensitive Data Protection

### Q10: How should you handle sensitive data?

**Sensitive Data Includes:**
- Passwords
- API keys
- Database credentials
- Personal Identifiable Information (PII)
- Payment information
- Session tokens

### Best Practices:

#### 1. **Environment Variables** ✅

```typescript
// ❌ NEVER hardcode secrets
const dbPassword = 'mypassword123'; // WRONG!

// ✅ Use environment variables
const dbPassword = process.env.DB_PASSWORD;

// ✅ Validate required env vars on startup
function validateEnvironment() {
  const required = ['DB_PASSWORD', 'JWT_SECRET', 'API_KEY'];
  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}

validateEnvironment();
```

**Your project (database.ts:10):**
```typescript
DB_PW = 'password', // ❌ Should not have default value!
```

**Improved version:**
```typescript
const DB_PW = process.env.DB_PW;
if (!DB_PW) {
  throw new Error('DB_PW environment variable is required');
}
```

#### 2. **Encryption at Rest**

```typescript
import crypto from 'crypto';

// Encrypt sensitive fields before storing in database
class EncryptionService {
  private algorithm = 'aes-256-gcm';
  private key = Buffer.from(process.env.ENCRYPTION_KEY!, 'hex'); // 32 bytes

  encrypt(plaintext: string): string {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(this.algorithm, this.key, iv);

    let encrypted = cipher.update(plaintext, 'utf8', 'hex');
    encrypted += cipher.final('hex');

    const authTag = cipher.getAuthTag();

    // Return: iv:authTag:encrypted
    return `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted}`;
  }

  decrypt(ciphertext: string): string {
    const [ivHex, authTagHex, encrypted] = ciphertext.split(':');
    const iv = Buffer.from(ivHex, 'hex');
    const authTag = Buffer.from(authTagHex, 'hex');

    const decipher = crypto.createDecipheriv(this.algorithm, this.key, iv);
    decipher.setAuthTag(authTag);

    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');

    return decrypted;
  }
}

// Usage in model
class Student extends Model {
  async setSSN(ssn: string) {
    const encrypted = encryptionService.encrypt(ssn);
    this.ssnEncrypted = encrypted;
  }

  async getSSN(): Promise<string> {
    return encryptionService.decrypt(this.ssnEncrypted);
  }
}
```

#### 3. **Encryption in Transit (HTTPS/TLS)**

```typescript
import https from 'https';
import fs from 'fs';

// Production: Use HTTPS
const httpsOptions = {
  key: fs.readFileSync('/path/to/private-key.pem'),
  cert: fs.readFileSync('/path/to/certificate.pem'),
};

const server = https.createServer(httpsOptions, app);
server.listen(443);

// Development: Redirect HTTP to HTTPS
app.use((req, res, next) => {
  if (!req.secure && process.env.NODE_ENV === 'production') {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});
```

#### 4. **Data Masking in Logs**

```typescript
// ❌ NEVER log sensitive data
logger.info('User login', { email, password }); // WRONG!

// ✅ Mask sensitive fields
function maskSensitiveData(obj: any): any {
  const sensitive = ['password', 'ssn', 'creditCard', 'apiKey'];
  const masked = { ...obj };

  for (const key of Object.keys(masked)) {
    if (sensitive.some(s => key.toLowerCase().includes(s))) {
      masked[key] = '***REDACTED***';
    }
  }

  return masked;
}

logger.info('User login', maskSensitiveData({ email, password }));
// Logs: { email: 'user@example.com', password: '***REDACTED***' }
```

#### 5. **Secure Delete**

```typescript
// When deleting sensitive data, overwrite first
async function secureDelete(userId: number) {
  // 1. Overwrite sensitive fields
  await User.update(
    {
      password: crypto.randomBytes(32).toString('hex'),
      ssn: crypto.randomBytes(16).toString('hex'),
      email: `deleted_${Date.now()}@deleted.com`,
    },
    { where: { id: userId } }
  );

  // 2. Then delete
  await User.destroy({ where: { id: userId } });
}
```

---

## API Security

### Q11: How do you secure REST APIs?

#### 1. **Authentication & Authorization**

```typescript
// JWT middleware
const authMiddleware = async (req: Request, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Role-based authorization
const requireRole = (role: string) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (req.user.role !== role) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
};

// Usage
router.post('/api/teachers', authMiddleware, requireRole('ADMIN'), createTeacher);
```

#### 2. **Rate Limiting**

```typescript
import rateLimit from 'express-rate-limit';

// General API rate limit
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Max 100 requests per window
  message: 'Too many requests, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

// Strict rate limit for login (prevent brute force)
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // Max 5 login attempts per 15 minutes
  skipSuccessfulRequests: true, // Don't count successful logins
});

app.use('/api/', apiLimiter);
app.post('/api/login', loginLimiter, loginController.login);
```

#### 3. **Input Validation**

```typescript
import { body, validationResult } from 'express-validator';

router.post(
  '/api/students',
  [
    body('name').isString().trim().isLength({ min: 1, max: 255 }),
    body('email').isEmail().normalizeEmail(),
    body('age').isInt({ min: 0, max: 150 }),
  ],
  (req: Request, res: Response) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // Process valid input
  }
);
```

#### 4. **CORS Configuration**

```typescript
import cors from 'cors';

// ❌ Dangerous: Allow all origins
app.use(cors()); // DON'T do this in production!

// ✅ Secure: Whitelist specific origins
const corsOptions = {
  origin: (origin, callback) => {
    const allowedOrigins = [
      'https://school-admin.com',
      'https://app.school-admin.com',
    ];

    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true, // Allow cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
};

app.use(cors(corsOptions));
```

#### 5. **Request Size Limiting**

```typescript
import express from 'express';

app.use(express.json({ limit: '10kb' })); // Limit JSON payload
app.use(express.urlencoded({ extended: true, limit: '10kb' }));
```

#### 6. **API Versioning**

```typescript
// Support multiple API versions
app.use('/api/v1', routerV1);
app.use('/api/v2', routerV2);

// Deprecation headers
app.use('/api/v1', (req, res, next) => {
  res.setHeader('X-API-Deprecated', 'true');
  res.setHeader('X-API-Sunset', '2024-12-31');
  next();
});
```

---

## Database Security

### Q12: How do you secure databases?

#### 1. **Use Parameterized Queries** (Covered in SQL Injection section)

#### 2. **Least Privilege Principle**

```sql
-- Create separate users for different purposes
CREATE USER 'app_readonly'@'localhost' IDENTIFIED BY 'strongpassword';
GRANT SELECT ON school_db.* TO 'app_readonly'@'localhost';

CREATE USER 'app_write'@'localhost' IDENTIFIED BY 'strongpassword';
GRANT SELECT, INSERT, UPDATE ON school_db.* TO 'app_write'@'localhost';

-- Never use root in application!
```

#### 3. **Connection Security**

```typescript
// Use SSL/TLS for database connections
const sequelize = new Sequelize(DB_SCHEMA, DB_USER, DB_PW, {
  dialect: 'mysql',
  host: DB_HOST,
  dialectOptions: {
    ssl: {
      require: true,
      rejectUnauthorized: true, // Verify server certificate
      ca: fs.readFileSync('/path/to/ca-cert.pem'),
    },
  },
});
```

#### 4. **Encrypt Sensitive Columns**

```typescript
// Sequelize hooks for automatic encryption
Student.beforeCreate(async (student) => {
  if (student.ssn) {
    student.ssn = await encrypt(student.ssn);
  }
});

Student.afterFind((students) => {
  if (Array.isArray(students)) {
    students.forEach(s => {
      if (s.ssn) s.ssn = decrypt(s.ssn);
    });
  } else if (students?.ssn) {
    students.ssn = decrypt(students.ssn);
  }
});
```

#### 5. **Database Auditing**

```typescript
// Track who changed what and when
class AuditLog extends Model {
  declare userId: number;
  declare action: string;
  declare tableName: string;
  declare recordId: number;
  declare changes: object;
  declare timestamp: Date;
}

// Sequelize hook to log changes
Student.afterUpdate(async (student, options) => {
  await AuditLog.create({
    userId: options.userId, // Pass from middleware
    action: 'UPDATE',
    tableName: 'students',
    recordId: student.id,
    changes: student.changed(),
    timestamp: new Date(),
  });
});
```

#### 6. **Connection Pooling Limits**

```typescript
// Prevent connection exhaustion
const sequelize = new Sequelize(DB_SCHEMA, DB_USER, DB_PW, {
  pool: {
    max: 10,        // Maximum connections
    min: 1,         // Minimum connections
    acquire: 30000, // Max time to get connection (ms)
    idle: 10000,    // Max idle time before release
  },
});
```

---

## Error Handling & Logging

### Q13: How should you handle errors securely?

#### 1. **Don't Leak Sensitive Information**

```typescript
// ❌ BAD: Exposes internal details
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack, // ❌ Exposes file paths, code structure
    query: req.query, // ❌ May contain sensitive data
  });
});

// ✅ GOOD: Generic error messages
app.use((err, req, res, next) => {
  // Log full error server-side
  logger.error('Request failed', {
    error: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
  });

  // Send generic message to client
  res.status(err.statusCode || 500).json({
    error: 'An error occurred processing your request',
    errorCode: err.code || 'INTERNAL_ERROR',
  });
});
```

**Your project (globalErrorHandler.ts:26-30):**
```typescript
return res.status(StatusCodes.INTERNAL_SERVER_ERROR).send({
  errorCode: ErrorCodes.RUNTIME_ERROR_CODE,
  message: 'Internal Server Error', // ✅ Good: Generic message
});
```

#### 2. **Structured Logging**

```typescript
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
  ],
});

// Log security events
logger.warn('Failed login attempt', {
  email: req.body.email,
  ip: req.ip,
  userAgent: req.headers['user-agent'],
  timestamp: new Date(),
});
```

#### 3. **Security Event Monitoring**

```typescript
// Track suspicious activities
const securityEvents = [
  'FAILED_LOGIN',
  'UNAUTHORIZED_ACCESS',
  'SQL_INJECTION_ATTEMPT',
  'XSS_ATTEMPT',
  'RATE_LIMIT_EXCEEDED',
];

function logSecurityEvent(event: string, details: object) {
  logger.warn('SECURITY_EVENT', {
    event,
    ...details,
    timestamp: new Date(),
  });

  // Alert if threshold exceeded
  if (shouldAlert(event)) {
    sendAlert(event, details);
  }
}

// Usage
app.use((err, req, res, next) => {
  if (err.message.includes("'") || err.message.includes('--')) {
    logSecurityEvent('SQL_INJECTION_ATTEMPT', {
      url: req.url,
      ip: req.ip,
      payload: req.body,
    });
  }
  next(err);
});
```

---

## Dependency Security

### Q14: How do you manage dependency vulnerabilities?

#### 1. **Regular Audits**

```bash
# Check for known vulnerabilities
npm audit

# Fix automatically (if possible)
npm audit fix

# Force fix (may introduce breaking changes)
npm audit fix --force
```

#### 2. **Automated Scanning**

```json
// package.json - Add to scripts
{
  "scripts": {
    "security-check": "npm audit && npm outdated"
  }
}
```

**Use tools:**
- **Snyk**: `npx snyk test`
- **OWASP Dependency-Check**
- **GitHub Dependabot**: Auto PRs for vulnerable dependencies

#### 3. **Lock File Usage**

```bash
# Ensure exact versions are installed
npm ci  # Uses package-lock.json (better for CI/CD)
```

**Why**: Prevents supply chain attacks where dependencies are replaced with malicious versions.

#### 4. **Minimize Dependencies**

```typescript
// ❌ BAD: Using entire library for one function
import _ from 'lodash'; // Entire library (70KB+)

// ✅ GOOD: Import only what you need
import debounce from 'lodash/debounce'; // Just debounce (2KB)
```

#### 5. **Verify Package Integrity**

```bash
# Check package checksum
npm install --integrity
```

---

## Security Headers

### Q15: What security headers should you set?

```typescript
import helmet from 'helmet';

// Use helmet for common security headers
app.use(helmet());

// Or configure individually:
app.use((req, res, next) => {
  // 1. Prevent clickjacking
  res.setHeader('X-Frame-Options', 'DENY');

  // 2. Prevent MIME sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');

  // 3. Enable XSS filter
  res.setHeader('X-XSS-Protection', '1; mode=block');

  // 4. Force HTTPS
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');

  // 5. Content Security Policy
  res.setHeader('Content-Security-Policy', "default-src 'self'; script-src 'self'");

  // 6. Referrer policy
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');

  // 7. Permissions policy
  res.setHeader('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');

  next();
});
```

**Headers Explained:**

| Header | Purpose | Example Value |
|--------|---------|---------------|
| `X-Frame-Options` | Prevent clickjacking | `DENY` or `SAMEORIGIN` |
| `X-Content-Type-Options` | Prevent MIME sniffing | `nosniff` |
| `X-XSS-Protection` | Enable browser XSS filter | `1; mode=block` |
| `Strict-Transport-Security` | Force HTTPS | `max-age=31536000` |
| `Content-Security-Policy` | Control resource loading | `default-src 'self'` |
| `Referrer-Policy` | Control referer header | `no-referrer` |

---

## Rate Limiting & DDoS Protection

### Q16: How do you prevent abuse and DoS attacks?

#### 1. **Rate Limiting**

```typescript
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import Redis from 'ioredis';

const redis = new Redis();

// IP-based rate limiting
const limiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:',
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Max requests per window
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      error: 'Too many requests, please slow down',
      retryAfter: req.rateLimit.resetTime,
    });
  },
});

app.use('/api/', limiter);
```

#### 2. **Distributed Rate Limiting** (for multiple servers)

```typescript
// Use Redis for centralized rate limiting
import { RateLimiterRedis } from 'rate-limiter-flexible';

const rateLimiter = new RateLimiterRedis({
  storeClient: redis,
  keyPrefix: 'rl',
  points: 100, // Number of requests
  duration: 60, // Per 60 seconds
  blockDuration: 60, // Block for 60 seconds if exceeded
});

app.use(async (req, res, next) => {
  try {
    await rateLimiter.consume(req.ip);
    next();
  } catch (error) {
    res.status(429).json({ error: 'Too Many Requests' });
  }
});
```

#### 3. **Request Timeout**

```typescript
import timeout from 'connect-timeout';

// Kill slow requests
app.use(timeout('5s')); // 5 second timeout

app.use((req, res, next) => {
  if (!req.timedout) next();
});
```

#### 4. **Payload Size Limits**

```typescript
// Already covered in API Security
app.use(express.json({ limit: '10kb' }));
```

#### 5. **Connection Limits**

```typescript
// Limit concurrent connections
import toobusy from 'toobusy-js';

app.use((req, res, next) => {
  if (toobusy()) {
    res.status(503).send('Server too busy');
  } else {
    next();
  }
});
```

---

## Common Security Interview Questions

### Q17: Walk me through how you would secure a REST API from scratch.

**Answer:**

"I would implement security in layers, following defense in depth principle:

**1. Transport Layer:**
- Use HTTPS/TLS for all communications
- Set up SSL certificates (Let's Encrypt for free certs)
- Force HTTPS redirects

**2. Authentication:**
- Implement JWT-based authentication
- Store JWT secret in environment variables
- Set reasonable expiration times (24h for access token)
- Implement refresh token mechanism

**3. Authorization:**
- Role-based access control (RBAC)
- Verify user permissions for each endpoint
- Follow principle of least privilege

**4. Input Validation:**
- Validate all inputs using express-validator
- Use parameterized queries to prevent SQL injection
- Sanitize user inputs to prevent XSS

**5. Rate Limiting:**
- General API: 100 req/15min per IP
- Login endpoint: 5 req/15min (brute force protection)
- Use Redis for distributed rate limiting

**6. Security Headers:**
- Implement helmet.js for standard headers
- Content-Security-Policy to prevent XSS
- CORS whitelist for allowed origins

**7. Error Handling:**
- Never expose internal details in error messages
- Log detailed errors server-side only
- Return generic error messages to clients

**8. Logging & Monitoring:**
- Log all security events (failed logins, unauthorized access)
- Set up alerts for suspicious patterns
- Regular security audit reviews

**9. Dependencies:**
- Regular npm audit checks
- Automated Dependabot for vulnerability updates
- Minimal dependencies principle

**10. Testing:**
- Security testing in CI/CD
- Penetration testing before production
- OWASP ZAP for automated security scans

In my school administration project, I implemented items 1-7, and would add 8-10 in a production environment."

### Q18: How would you prevent and detect a SQL injection attack?

**Answer:**

"**Prevention (Primary Defense):**

1. **Parameterized Queries**: Use ORMs like Sequelize that automatically parameterize queries. In my project, all database queries use Sequelize's built-in methods which prevent SQL injection.

```typescript
// Safe: Sequelize parameterizes automatically
await Student.findOne({ where: { email } });
```

2. **Input Validation**: Validate all inputs against expected patterns before using in queries.

3. **Least Privilege**: Database user should only have necessary permissions, not admin rights.

**Detection:**

1. **Pattern Matching**: Monitor for SQL keywords in input fields:
```typescript
if (input.match(/(\bOR\b|\bAND\b|--|;|'|\")/i)) {
  logSecurityEvent('POSSIBLE_SQL_INJECTION', { input, ip: req.ip });
}
```

2. **Error Monitoring**: Sudden increase in database errors could indicate injection attempts.

3. **WAF (Web Application Firewall)**: Tools like ModSecurity can detect and block SQL injection patterns.

4. **Security Logging**: Log all database queries with suspicious patterns for forensic analysis.

In my project, prevention is the focus through Sequelize ORM usage and input validation via csvValidator.ts."

### Q19: Explain the difference between encryption, hashing, and encoding.

**Answer:**

**Encoding**: Transforms data into different format, reversible without key (NOT for security)
- Purpose: Data transmission compatibility
- Examples: Base64, URL encoding
- Reversible: Yes (no key needed)
- Use case: Encode binary image to send in JSON

**Hashing**: One-way function, irreversible
- Purpose: Data integrity, password storage
- Examples: SHA-256, bcrypt
- Reversible: No (by design)
- Use case: Store passwords - you never decrypt, only compare hashes

```typescript
// Hashing (one-way)
const hash = await bcrypt.hash('password123', 12);
// hash: $2b$12$KOEz...  (can't get 'password123' back)
const isValid = await bcrypt.compare('password123', hash); // true
```

**Encryption**: Two-way, reversible with key
- Purpose: Confidentiality of sensitive data
- Examples: AES-256, RSA
- Reversible: Yes (with decryption key)
- Use case: Store SSN - you need to retrieve original value

```typescript
// Encryption (two-way)
const encrypted = encrypt('123-45-6789', key);
// encrypted: "a7f3e9..."
const original = decrypt(encrypted, key);
// original: "123-45-6789"
```

**When to use what:**
- Passwords → Hashing (bcrypt, Argon2)
- Sensitive data you need to retrieve → Encryption (AES)
- Data transmission → Encoding (Base64)"

### Q20: How do you prevent XSS attacks in a React application?

**Answer:**

"**React has built-in XSS protection**, but you can still introduce vulnerabilities:

**1. React's Default Protection:**
React automatically escapes values in JSX:
```jsx
const name = "<script>alert('xss')</script>";
return <div>{name}</div>;
// ✅ Safe: Renders as text, not executed
```

**2. Dangerous Patterns to Avoid:**
```jsx
// ❌ DANGEROUS: Bypasses React's escaping
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ❌ DANGEROUS: Direct DOM manipulation
useEffect(() => {
  document.getElementById('msg').innerHTML = userInput;
}, []);
```

**3. Safe Alternatives:**
```jsx
// ✅ Use DOMPurify if you MUST render HTML
import DOMPurify from 'dompurify';

<div dangerouslySetInnerHTML={{
  __html: DOMPurify.sanitize(userInput)
}} />
```

**4. Server-Side Protection:**
Even though React escapes, validate on backend:
```typescript
function sanitizeInput(input: string): string {
  return input.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');
}
```

**5. Content Security Policy:**
```typescript
res.setHeader('Content-Security-Policy',
  "script-src 'self'; object-src 'none';"
);
```

**6. HTTP-only Cookies:**
```typescript
res.cookie('session', token, {
  httpOnly: true, // Prevent JavaScript access
  secure: true,
  sameSite: 'strict',
});
```

The key is: trust React's default escaping, never use dangerouslySetInnerHTML with user input, and always validate server-side."

---

## Summary & Interview Tips

### Key Security Principles to Remember:

1. **Defense in Depth**: Multiple layers of security
2. **Least Privilege**: Minimum necessary permissions
3. **Fail Securely**: Deny by default
4. **Never Trust Input**: Validate everything
5. **Keep it Simple**: Complexity is the enemy of security
6. **Security by Design**: Not an afterthought

### Common Security Mistakes in Your Project:

From your codebase analysis:

1. ✅ **Good**: Using Sequelize ORM (SQL injection protection)
2. ✅ **Good**: Input validation in csvValidator.ts
3. ✅ **Good**: File upload restrictions (CSV only, size limit)
4. ✅ **Good**: Generic error messages in globalErrorHandler.ts
5. ❌ **Issue**: Hardcoded default password in database.ts:10
6. ❌ **Issue**: No authentication/authorization middleware
7. ❌ **Issue**: CORS allows all origins (app.ts:11)
8. ❌ **Issue**: No rate limiting
9. ❌ **Issue**: No security headers (helmet)

### What to Say in Interview:

"In my school administration project, I implemented several security measures:
- SQL injection prevention through Sequelize ORM parameterized queries
- Input validation for CSV uploads with strict file type checking
- File size limits to prevent DoS attacks
- Proper error handling that doesn't leak internal details

If this were production, I would add:
- JWT authentication with role-based access control
- Rate limiting to prevent brute force attacks
- Security headers using helmet.js
- Move all secrets to environment variables with validation
- Implement CORS whitelist instead of allowing all origins
- Add comprehensive security logging and monitoring
- Regular dependency audits with npm audit

I understand security is an ongoing process requiring multiple layers of defense."

---

## Additional Resources

- **OWASP Cheat Sheets**: https://cheatsheetseries.owasp.org/
- **Node.js Security Best Practices**: https://nodejs.org/en/docs/guides/security/
- **Express.js Security**: https://expressjs.com/en/advanced/best-practice-security.html
- **Sequelize Security**: https://sequelize.org/docs/v6/core-concepts/raw-queries/#replacements

**Practice Labs:**
- OWASP WebGoat
- HackTheBox
- PortSwigger Web Security Academy
