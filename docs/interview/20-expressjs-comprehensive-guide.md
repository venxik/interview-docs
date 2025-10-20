# Express.js Comprehensive Guide & Interview Questions

## Table of Contents
1. [Express.js Fundamentals](#expressjs-fundamentals)
2. [Routing](#routing)
3. [Middleware](#middleware)
4. [Request & Response](#request--response)
5. [Error Handling](#error-handling)
6. [Database Integration](#database-integration)
7. [Authentication & Authorization](#authentication--authorization)
8. [File Uploads](#file-uploads)
9. [Security Best Practices](#security-best-practices)
10. [Testing](#testing)
11. [Performance & Optimization](#performance--optimization)
12. [Common Interview Questions](#common-interview-questions)

---

## Express.js Fundamentals

### What is Express.js?

**Express** is a minimal and flexible Node.js web application framework that provides:
- ✅ Routing
- ✅ Middleware
- ✅ HTTP utility methods
- ✅ Template engines (optional)
- ✅ Static file serving

**Why Express?**

```
Plain Node.js:
❌ Verbose HTTP handling
❌ Manual routing
❌ No middleware system
❌ Complex request parsing

Express:
✅ Simple, intuitive API
✅ Powerful routing
✅ Middleware ecosystem
✅ Built-in parsers
✅ Large community
```

### Basic Express App

```typescript
import express, { Request, Response } from 'express';

const app = express();
const port = 3000;

// Middleware
app.use(express.json());  // Parse JSON bodies
app.use(express.urlencoded({ extended: true }));  // Parse URL-encoded bodies

// Routes
app.get('/', (req: Request, res: Response) => {
  res.send('Hello World!');
});

app.get('/api/students', (req: Request, res: Response) => {
  res.json({ students: [] });
});

// Start server
app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
```

### Project Structure (Your School Admin System)

```
typescript/src/
├── api/
│   ├── controllers/          # Request handlers
│   │   ├── StudentController.ts
│   │   ├── ClassController.ts
│   │   └── index.ts
│   │
│   ├── middleware/           # Custom middleware
│   │   ├── auth.ts
│   │   ├── errorHandler.ts
│   │   ├── validation.ts
│   │   └── index.ts
│   │
│   └── routes/               # Route definitions
│       ├── students.ts
│       ├── classes.ts
│       └── index.ts
│
├── services/                 # Business logic
│   ├── StudentService.ts
│   ├── ClassService.ts
│   └── index.ts
│
├── database/
│   ├── models/               # Sequelize models
│   ├── repositories/         # Database access layer
│   └── config/
│
├── config/                   # Configuration
│   ├── database.ts
│   ├── logger.ts
│   └── index.ts
│
├── shared/
│   ├── types/
│   ├── errors/
│   └── utils/
│
├── app.ts                    # Express app setup
├── router.ts                 # Main router
└── server.ts                 # Server entry point
```

---

## Routing

### Basic Routes

```typescript
// router.ts
import express, { Router } from 'express';

const router: Router = express.Router();

// GET /api/students
router.get('/students', (req, res) => {
  res.json({ students: [] });
});

// POST /api/students
router.post('/students', (req, res) => {
  const student = req.body;
  res.status(201).json({ student });
});

// GET /api/students/:id
router.get('/students/:id', (req, res) => {
  const { id } = req.params;
  res.json({ student: { id } });
});

// PUT /api/students/:id
router.put('/students/:id', (req, res) => {
  const { id } = req.params;
  const updates = req.body;
  res.json({ student: { id, ...updates } });
});

// DELETE /api/students/:id
router.delete('/students/:id', (req, res) => {
  const { id } = req.params;
  res.status(204).send();
});

export default router;
```

### Route Parameters

```typescript
// Single parameter
app.get('/students/:id', (req, res) => {
  const { id } = req.params;
  res.json({ id });
});

// Multiple parameters
app.get('/classes/:classId/students/:studentId', (req, res) => {
  const { classId, studentId } = req.params;
  res.json({ classId, studentId });
});

// Optional parameter
app.get('/students/:id?', (req, res) => {
  if (req.params.id) {
    // Get specific student
  } else {
    // Get all students
  }
});

// Regular expression pattern
app.get(/^\/students\/(\d+)$/, (req, res) => {
  const id = req.params[0];  // Only matches numeric IDs
  res.json({ id });
});
```

### Query Parameters

```typescript
// GET /api/students?grade=10&page=2&limit=20
app.get('/students', (req, res) => {
  const { grade, page = 1, limit = 10 } = req.query;

  res.json({
    grade,
    page: parseInt(page as string),
    limit: parseInt(limit as string)
  });
});
```

### Router Modularity

```typescript
// routes/students.ts
import express from 'express';
import { StudentController } from '../controllers/StudentController';

const router = express.Router();

router.get('/', StudentController.list);
router.post('/', StudentController.create);
router.get('/:id', StudentController.get);
router.put('/:id', StudentController.update);
router.delete('/:id', StudentController.delete);

export default router;

// routes/index.ts
import express from 'express';
import studentRoutes from './students';
import classRoutes from './classes';

const router = express.Router();

router.use('/students', studentRoutes);
router.use('/classes', classRoutes);

export default router;

// app.ts
import router from './routes';

app.use('/api', router);

// Result:
// GET /api/students
// POST /api/students
// GET /api/students/:id
// etc.
```

### Route Grouping

```typescript
// Group related routes
const studentRouter = express.Router();

studentRouter.get('/', list);
studentRouter.post('/', create);

studentRouter.route('/:id')
  .get(get)
  .put(update)
  .delete(remove);

app.use('/api/students', studentRouter);
```

---

## Middleware

### What is Middleware?

**Middleware** functions have access to:
- Request object (`req`)
- Response object (`res`)
- Next middleware function (`next`)

**Flow:**

```
Request → Middleware 1 → Middleware 2 → Route Handler → Response
```

### Built-in Middleware

```typescript
import express from 'express';

const app = express();

// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

// Serve static files
app.use(express.static('public'));

// Example: public/images/logo.png → http://localhost:3000/images/logo.png
```

### Third-Party Middleware

```typescript
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';
import compression from 'compression';

// CORS (Cross-Origin Resource Sharing)
app.use(cors({
  origin: 'http://localhost:3001',
  credentials: true
}));

// Security headers
app.use(helmet());

// HTTP request logger
app.use(morgan('combined'));

// Gzip compression
app.use(compression());
```

### Custom Middleware

```typescript
// Logger middleware
const logger = (req: Request, res: Response, next: NextFunction) => {
  console.log(`${req.method} ${req.path}`);
  next();  // Must call next() to continue
};

app.use(logger);

// Timing middleware
const timer = (req: Request, res: Response, next: NextFunction) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.path} - ${duration}ms`);
  });

  next();
};

app.use(timer);

// Authentication middleware
const authenticate = (req: Request, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  try {
    const decoded = verifyToken(token);
    req.user = decoded;  // Attach user to request
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Use on specific routes
app.get('/api/students', authenticate, (req, res) => {
  // req.user is available here
  res.json({ user: req.user });
});
```

### Middleware Types

```typescript
// 1. Application-level middleware
app.use((req, res, next) => {
  console.log('Application middleware');
  next();
});

// 2. Router-level middleware
const router = express.Router();

router.use((req, res, next) => {
  console.log('Router middleware');
  next();
});

// 3. Error-handling middleware (4 parameters!)
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err);
  res.status(500).json({ error: err.message });
});

// 4. Built-in middleware
app.use(express.json());

// 5. Third-party middleware
app.use(cors());
```

### Middleware Execution Order

```typescript
// Order matters!
app.use(middleware1);  // Runs first
app.use(middleware2);  // Runs second

app.get('/students', handler);  // Runs last

// Example flow:
const app = express();

// 1. Logger (runs for all routes)
app.use((req, res, next) => {
  console.log('1. Logger');
  next();
});

// 2. Auth check (runs for /api/* routes)
app.use('/api', (req, res, next) => {
  console.log('2. Auth check');
  next();
});

// 3. Route handler
app.get('/api/students', (req, res) => {
  console.log('3. Route handler');
  res.json({ students: [] });
});

// Request to /api/students:
// Output: "1. Logger" → "2. Auth check" → "3. Route handler"
```

---

## Request & Response

### Request Object

```typescript
app.get('/students', (req: Request, res: Response) => {
  // Route parameters
  req.params.id  // /students/:id

  // Query parameters
  req.query.page  // /students?page=2

  // Request body
  req.body  // POST/PUT data

  // Headers
  req.headers.authorization
  req.get('Content-Type')

  // Cookies (requires cookie-parser)
  req.cookies.sessionId

  // Request properties
  req.method  // GET, POST, etc.
  req.path  // /students
  req.url  // /students?page=2
  req.protocol  // http or https
  req.hostname  // localhost
  req.ip  // Client IP address

  // Custom properties (set by middleware)
  req.user  // Set by auth middleware
});
```

### Response Object

```typescript
app.get('/students', (req: Request, res: Response) => {
  // Send JSON
  res.json({ students: [] });

  // Send text
  res.send('Hello World');

  // Send status code
  res.status(404).json({ error: 'Not found' });
  res.sendStatus(204);  // No content

  // Redirect
  res.redirect('/login');
  res.redirect(301, 'https://example.com');

  // Set headers
  res.set('Content-Type', 'application/json');
  res.set({
    'X-Custom-Header': 'value',
    'Cache-Control': 'no-cache'
  });

  // Download file
  res.download('/path/to/file.pdf');

  // Send file
  res.sendFile('/path/to/file.pdf');

  // Render template (if using template engine)
  res.render('students', { students: [] });
});
```

### Status Codes

```typescript
// Success
res.status(200).json({ data });  // OK
res.status(201).json({ created });  // Created
res.status(204).send();  // No Content

// Client Errors
res.status(400).json({ error: 'Bad Request' });
res.status(401).json({ error: 'Unauthorized' });
res.status(403).json({ error: 'Forbidden' });
res.status(404).json({ error: 'Not Found' });
res.status(409).json({ error: 'Conflict' });
res.status(422).json({ error: 'Validation failed' });

// Server Errors
res.status(500).json({ error: 'Internal Server Error' });
res.status(503).json({ error: 'Service Unavailable' });
```

---

## Error Handling

### Basic Error Handling

```typescript
// Synchronous error
app.get('/students/:id', (req, res) => {
  const student = findStudent(req.params.id);

  if (!student) {
    return res.status(404).json({ error: 'Student not found' });
  }

  res.json(student);
});

// Asynchronous error (async/await)
app.get('/students/:id', async (req, res, next) => {
  try {
    const student = await db.Student.findByPk(req.params.id);

    if (!student) {
      return res.status(404).json({ error: 'Student not found' });
    }

    res.json(student);
  } catch (error) {
    next(error);  // Pass to error handler
  }
});
```

### Centralized Error Handler

```typescript
// Custom error class
class ErrorBase extends Error {
  statusCode: number;
  errorCode: string;

  constructor(message: string, statusCode: number, errorCode: string) {
    super(message);
    this.statusCode = statusCode;
    this.errorCode = errorCode;
  }
}

// Error handler middleware (must be last!)
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err);

  if (err instanceof ErrorBase) {
    return res.status(err.statusCode).json({
      error: err.message,
      code: err.errorCode
    });
  }

  // Unexpected errors
  res.status(500).json({
    error: 'Internal Server Error',
    code: 'INTERNAL_ERROR'
  });
});

// Usage
app.get('/students/:id', async (req, res, next) => {
  try {
    const student = await db.Student.findByPk(req.params.id);

    if (!student) {
      throw new ErrorBase('Student not found', 404, 'STUDENT_NOT_FOUND');
    }

    res.json(student);
  } catch (error) {
    next(error);
  }
});
```

### Error Handler from Your Project

```typescript
// api/middleware/globalErrorHandler.ts
import { ErrorRequestHandler } from 'express';
import { StatusCodes } from 'http-status-codes';
import ErrorBase from '@errors/ErrorBase';
import ErrorCodes from '@constants/ErrorCodes';

const globalErrorHandler: ErrorRequestHandler = (err, req, res, next) => {
  // Don't send response if headers already sent
  if (res.headersSent) {
    return next(err);
  }

  // Handle malformed JSON
  if (err.type === 'entity.parse.failed') {
    return res.status(StatusCodes.BAD_REQUEST).json({
      errorCode: ErrorCodes.MALFORMED_JSON_ERROR_CODE,
      message: 'Malformed json',
    });
  }

  // Handle custom errors
  if (err instanceof ErrorBase) {
    return res.status(err.getHttpStatusCode()).json({
      errorCode: err.getErrorCode(),
      message: err.getMessage(),
    });
  }

  // Unexpected errors
  return res.status(StatusCodes.INTERNAL_SERVER_ERROR).json({
    errorCode: ErrorCodes.RUNTIME_ERROR_CODE,
    message: 'Internal Server Error',
  });
};

export default globalErrorHandler;
```

### Async Error Wrapper

```typescript
// Utility to avoid try/catch in every route
const asyncHandler = (fn: Function) => {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Usage
app.get('/students', asyncHandler(async (req, res) => {
  const students = await db.Student.findAll();  // No try/catch needed!
  res.json(students);
}));
```

---

## Database Integration

### Sequelize Integration (Your Project)

```typescript
// database/config/database.ts
import { Sequelize } from 'sequelize';

const sequelize = new Sequelize(
  process.env.DB_SCHEMA!,
  process.env.DB_USER!,
  process.env.DB_PW!,
  {
    dialect: 'mysql',
    host: process.env.DB_HOST,
    port: parseInt(process.env.DB_PORT!),
    pool: {
      max: 10,
      min: 1,
      acquire: 30000,
      idle: 10000,
    },
    logging: false,  // Disable SQL logging
  }
);

export default sequelize;

// Test connection
sequelize.authenticate()
  .then(() => console.log('Database connected'))
  .catch(err => console.error('Unable to connect:', err));
```

### Repository Pattern (Your Project)

```typescript
// database/repositories/BaseRepository.ts
export class BaseRepository<T extends Model> {
  protected model: ModelStatic<T>;

  constructor(model: ModelStatic<T>) {
    this.model = model;
  }

  async findAll(options?: FindOptions): Promise<T[]> {
    return this.model.findAll(options);
  }

  async findOne(options: FindOptions): Promise<T | null> {
    return this.model.findOne(options);
  }

  async findByPk(id: number | string): Promise<T | null> {
    return this.model.findByPk(id);
  }

  async create(data: any): Promise<T> {
    return this.model.create(data);
  }

  async update(id: number | string, data: any): Promise<T | null> {
    await this.model.update(data, { where: { id } });
    return this.findByPk(id);
  }

  async delete(id: number | string): Promise<boolean> {
    const deleted = await this.model.destroy({ where: { id } });
    return deleted > 0;
  }
}

// database/repositories/StudentRepository.ts
import { BaseRepository } from './BaseRepository';
import Student from '../models/Student';

export class StudentRepository extends BaseRepository<Student> {
  constructor() {
    super(Student);
  }

  async findByEmail(email: string): Promise<Student | null> {
    return this.findOne({ where: { email } });
  }

  async findByGrade(grade: number): Promise<Student[]> {
    return this.findAll({ where: { grade } });
  }
}
```

### Controller Pattern (Your Project)

```typescript
// api/controllers/StudentController.ts
import { Request, Response, NextFunction } from 'express';
import { StudentService } from '@services/StudentService';

export class StudentController {
  private studentService: StudentService;

  constructor() {
    this.studentService = new StudentService();
  }

  list = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const { grade, page = 1, limit = 10 } = req.query;

      const students = await this.studentService.listStudents({
        grade: grade ? parseInt(grade as string) : undefined,
        page: parseInt(page as string),
        limit: parseInt(limit as string),
      });

      res.json(students);
    } catch (error) {
      next(error);
    }
  };

  get = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const { id } = req.params;
      const student = await this.studentService.getStudent(parseInt(id));

      if (!student) {
        return res.status(404).json({ error: 'Student not found' });
      }

      res.json(student);
    } catch (error) {
      next(error);
    }
  };

  create = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const student = await this.studentService.createStudent(req.body);
      res.status(201).json(student);
    } catch (error) {
      next(error);
    }
  };

  update = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const { id } = req.params;
      const student = await this.studentService.updateStudent(
        parseInt(id),
        req.body
      );

      if (!student) {
        return res.status(404).json({ error: 'Student not found' });
      }

      res.json(student);
    } catch (error) {
      next(error);
    }
  };

  delete = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const { id } = req.params;
      const deleted = await this.studentService.deleteStudent(parseInt(id));

      if (!deleted) {
        return res.status(404).json({ error: 'Student not found' });
      }

      res.status(204).send();
    } catch (error) {
      next(error);
    }
  };
}
```

---

## Authentication & Authorization

### JWT Authentication

```typescript
import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';

// Register
app.post('/api/auth/register', async (req, res) => {
  const { email, password, name } = req.body;

  // Hash password
  const passwordHash = await bcrypt.hash(password, 12);

  // Create user
  const user = await db.User.create({
    email,
    name,
    passwordHash,
  });

  // Generate JWT
  const token = jwt.sign(
    { userId: user.id, email: user.email },
    process.env.JWT_SECRET!,
    { expiresIn: '24h' }
  );

  res.status(201).json({ token, user: { id: user.id, email, name } });
});

// Login
app.post('/api/auth/login', async (req, res) => {
  const { email, password } = req.body;

  // Find user
  const user = await db.User.findOne({ where: { email } });
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Verify password
  const isValid = await bcrypt.compare(password, user.passwordHash);
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Generate JWT
  const token = jwt.sign(
    { userId: user.id, email: user.email, role: user.role },
    process.env.JWT_SECRET!,
    { expiresIn: '24h' }
  );

  res.json({ token, user: { id: user.id, email: user.email, name: user.name } });
});

// Auth middleware
const authenticate = (req: Request, res: Response, next: NextFunction) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const token = authHeader.substring(7);

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as any;
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Protected route
app.get('/api/students', authenticate, async (req, res) => {
  // req.user is available
  const students = await db.Student.findAll();
  res.json(students);
});
```

### Role-Based Authorization

```typescript
// Authorization middleware
const authorize = (...roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }

    next();
  };
};

// Usage
app.post('/api/students', authenticate, authorize('ADMIN', 'TEACHER'), async (req, res) => {
  // Only ADMIN or TEACHER can create students
  const student = await db.Student.create(req.body);
  res.status(201).json(student);
});

app.delete('/api/students/:id', authenticate, authorize('ADMIN'), async (req, res) => {
  // Only ADMIN can delete students
  await db.Student.destroy({ where: { id: req.params.id } });
  res.status(204).send();
});
```

---

## File Uploads

### Multer (Your Project)

```typescript
// api/middleware/multer.ts
import multer from 'multer';
import path from 'path';

const FILE_LIMIT = 5 * 1024 * 1024;  // 5 MB

const diskStorage = multer.diskStorage({
  destination: '/tmp/school-administration-system-uploads',
  filename: (req, file, cb) => {
    const uniqueName = `${Date.now()}-${file.originalname}`;
    cb(null, uniqueName);
  },
});

const fileFilter = (req: Request, file: Express.Multer.File, cb: FileFilterCallback) => {
  if (file.mimetype === 'text/csv' || file.originalname.toLowerCase().endsWith('.csv')) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type. Only .csv files are allowed.'));
  }
};

const upload = multer({
  storage: diskStorage,
  fileFilter,
  limits: { fileSize: FILE_LIMIT },
});

export default upload;

// Usage
import upload from '@middleware/multer';

app.post('/api/upload', upload.single('file'), (req, res) => {
  const file = req.file;

  if (!file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }

  res.json({
    filename: file.filename,
    path: file.path,
    size: file.size,
  });
});

// Multiple files
app.post('/api/upload-multiple', upload.array('files', 5), (req, res) => {
  const files = req.files as Express.Multer.File[];

  res.json({
    files: files.map(f => ({
      filename: f.filename,
      size: f.size,
    })),
  });
});
```

### File Upload to S3

```typescript
import AWS from 'aws-sdk';
import multer from 'multer';
import multerS3 from 'multer-s3';

const s3 = new AWS.S3();

const upload = multer({
  storage: multerS3({
    s3,
    bucket: 'my-bucket',
    acl: 'private',
    metadata: (req, file, cb) => {
      cb(null, { uploadedBy: req.user.id });
    },
    key: (req, file, cb) => {
      const uniqueName = `${Date.now()}-${file.originalname}`;
      cb(null, `uploads/${uniqueName}`);
    },
  }),
  limits: { fileSize: 5 * 1024 * 1024 },
});

app.post('/api/upload', upload.single('file'), (req, res) => {
  const file = req.file as Express.MulterS3.File;

  res.json({
    url: file.location,  // S3 URL
    key: file.key,
  });
});
```

---

## Security Best Practices

### 1. Helmet (Security Headers)

```typescript
import helmet from 'helmet';

app.use(helmet());

// Or configure individually
app.use(helmet.contentSecurityPolicy());
app.use(helmet.dnsPrefetchControl());
app.use(helmet.frameguard());
app.use(helmet.hidePoweredBy());
app.use(helmet.hsts());
app.use(helmet.ieNoOpen());
app.use(helmet.noSniff());
app.use(helmet.xssFilter());
```

### 2. CORS

```typescript
import cors from 'cors';

// Allow all origins (development only!)
app.use(cors());

// Production (whitelist)
app.use(cors({
  origin: ['https://school-admin.com', 'https://app.school-admin.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```

### 3. Rate Limiting

```typescript
import rateLimit from 'express-rate-limit';

// General API rate limit
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,  // Max 100 requests per window
  message: 'Too many requests, please try again later',
});

app.use('/api/', apiLimiter);

// Strict rate limit for login
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,  // Max 5 login attempts
  skipSuccessfulRequests: true,
});

app.post('/api/auth/login', loginLimiter, loginHandler);
```

### 4. Input Validation

```typescript
import { body, validationResult } from 'express-validator';

app.post(
  '/api/students',
  [
    body('name').isString().trim().isLength({ min: 1, max: 255 }),
    body('email').isEmail().normalizeEmail(),
    body('grade').isInt({ min: 1, max: 12 }),
  ],
  (req: Request, res: Response) => {
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    // Process valid input
    const student = req.body;
    res.json(student);
  }
);
```

### 5. SQL Injection Prevention

```typescript
// ❌ Vulnerable (string concatenation)
const query = `SELECT * FROM students WHERE email = '${req.query.email}'`;
await db.query(query);

// ✅ Safe (parameterized query)
const student = await db.Student.findOne({
  where: { email: req.query.email }
});

// ✅ Safe (with Sequelize)
const students = await db.Student.findAll({
  where: {
    grade: req.query.grade,
    name: { [Op.like]: `%${req.query.search}%` }
  }
});
```

### 6. XSS Prevention

```typescript
import DOMPurify from 'isomorphic-dompurify';

app.post('/api/comments', (req, res) => {
  // Sanitize HTML input
  const clean = DOMPurify.sanitize(req.body.comment);

  const comment = await db.Comment.create({ content: clean });
  res.json(comment);
});
```

---

## Testing

### Unit Testing (Jest)

```typescript
// controllers/StudentController.test.ts
import { StudentController } from './StudentController';
import { StudentService } from '@services/StudentService';

jest.mock('@services/StudentService');

describe('StudentController', () => {
  let controller: StudentController;
  let mockService: jest.Mocked<StudentService>;
  let req: any;
  let res: any;
  let next: any;

  beforeEach(() => {
    mockService = new StudentService() as jest.Mocked<StudentService>;
    controller = new StudentController();
    (controller as any).studentService = mockService;

    req = { params: {}, query: {}, body: {} };
    res = {
      json: jest.fn(),
      status: jest.fn().mockReturnThis(),
    };
    next = jest.fn();
  });

  describe('list', () => {
    it('should return list of students', async () => {
      const students = [
        { id: 1, name: 'John', email: 'john@school.com' },
        { id: 2, name: 'Jane', email: 'jane@school.com' },
      ];

      mockService.listStudents.mockResolvedValue(students);

      await controller.list(req, res, next);

      expect(mockService.listStudents).toHaveBeenCalledWith({
        grade: undefined,
        page: 1,
        limit: 10,
      });
      expect(res.json).toHaveBeenCalledWith(students);
    });
  });

  describe('get', () => {
    it('should return student by id', async () => {
      const student = { id: 1, name: 'John', email: 'john@school.com' };

      mockService.getStudent.mockResolvedValue(student);
      req.params.id = '1';

      await controller.get(req, res, next);

      expect(mockService.getStudent).toHaveBeenCalledWith(1);
      expect(res.json).toHaveBeenCalledWith(student);
    });

    it('should return 404 if student not found', async () => {
      mockService.getStudent.mockResolvedValue(null);
      req.params.id = '999';

      await controller.get(req, res, next);

      expect(res.status).toHaveBeenCalledWith(404);
      expect(res.json).toHaveBeenCalledWith({ error: 'Student not found' });
    });
  });
});
```

### Integration Testing (Supertest)

```typescript
// app.test.ts
import request from 'supertest';
import app from './app';
import sequelize from './database/config/database';

beforeAll(async () => {
  await sequelize.sync({ force: true });
});

afterAll(async () => {
  await sequelize.close();
});

describe('Student API', () => {
  describe('GET /api/students', () => {
    it('should return empty array', async () => {
      const res = await request(app).get('/api/students');

      expect(res.status).toBe(200);
      expect(res.body).toEqual([]);
    });
  });

  describe('POST /api/students', () => {
    it('should create student', async () => {
      const student = {
        name: 'John Doe',
        email: 'john@school.com',
        grade: 10,
      };

      const res = await request(app)
        .post('/api/students')
        .send(student);

      expect(res.status).toBe(201);
      expect(res.body).toMatchObject(student);
      expect(res.body.id).toBeDefined();
    });

    it('should return 400 for invalid data', async () => {
      const res = await request(app)
        .post('/api/students')
        .send({ name: 'John' });  // Missing email

      expect(res.status).toBe(400);
    });
  });

  describe('GET /api/students/:id', () => {
    it('should return student', async () => {
      // Create student
      const created = await request(app)
        .post('/api/students')
        .send({ name: 'John', email: 'john@school.com', grade: 10 });

      // Get student
      const res = await request(app).get(`/api/students/${created.body.id}`);

      expect(res.status).toBe(200);
      expect(res.body.id).toBe(created.body.id);
    });

    it('should return 404 for non-existent student', async () => {
      const res = await request(app).get('/api/students/999');

      expect(res.status).toBe(404);
    });
  });
});
```

---

## Performance & Optimization

### 1. Connection Pooling

```typescript
// Database connection pool
const sequelize = new Sequelize(DB_NAME, DB_USER, DB_PASSWORD, {
  pool: {
    max: 10,      // Maximum connections
    min: 1,       // Minimum connections
    acquire: 30000,  // Max time to get connection (ms)
    idle: 10000,  // Max idle time before release
  },
});
```

### 2. Caching

```typescript
import Redis from 'ioredis';

const redis = new Redis();

// Cache middleware
const cacheMiddleware = (duration: number) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    const key = `cache:${req.originalUrl}`;

    const cached = await redis.get(key);
    if (cached) {
      return res.json(JSON.parse(cached));
    }

    // Intercept res.json to cache response
    const originalJson = res.json.bind(res);
    res.json = (body: any) => {
      redis.setex(key, duration, JSON.stringify(body));
      return originalJson(body);
    };

    next();
  };
};

// Usage
app.get('/api/students', cacheMiddleware(300), async (req, res) => {
  const students = await db.Student.findAll();
  res.json(students);
});
```

### 3. Compression

```typescript
import compression from 'compression';

app.use(compression());

// Compress responses (gzip)
// 100 KB → 10 KB (90% reduction)
```

### 4. Pagination

```typescript
app.get('/api/students', async (req, res) => {
  const page = parseInt(req.query.page as string) || 1;
  const limit = parseInt(req.query.limit as string) || 10;
  const offset = (page - 1) * limit;

  const { count, rows } = await db.Student.findAndCountAll({
    limit,
    offset,
  });

  res.json({
    data: rows,
    pagination: {
      currentPage: page,
      totalPages: Math.ceil(count / limit),
      totalItems: count,
      itemsPerPage: limit,
    },
  });
});
```

### 5. Database Indexing

```typescript
// Add indexes to frequently queried columns
Student.init({
  email: {
    type: DataTypes.STRING,
    unique: true,
    allowNull: false,
  },
  name: {
    type: DataTypes.STRING,
    allowNull: false,
    // Add index for faster searches
    indexes: [{ fields: ['name'] }],
  },
  grade: {
    type: DataTypes.INTEGER,
    // Add index for filtering
    indexes: [{ fields: ['grade'] }],
  },
});
```

### 6. Async/Await Best Practices

```typescript
// ❌ Sequential (slow)
const student = await db.Student.findByPk(id);
const classes = await db.Class.findAll({ where: { studentId: id } });
const grades = await db.Grade.findAll({ where: { studentId: id } });
// Total: 300ms

// ✅ Parallel (fast)
const [student, classes, grades] = await Promise.all([
  db.Student.findByPk(id),
  db.Class.findAll({ where: { studentId: id } }),
  db.Grade.findAll({ where: { studentId: id } }),
]);
// Total: 100ms (max of three queries)
```

---

## Common Interview Questions

### Q1: What is Express.js and why use it over plain Node.js?

**Answer:**

"Express.js is a minimal web framework for Node.js that simplifies building web applications and APIs.

**Plain Node.js:**
```typescript
import http from 'http';

const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/students') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ students: [] }));
  } else if (req.method === 'POST' && req.url === '/students') {
    let body = '';
    req.on('data', chunk => { body += chunk; });
    req.on('end', () => {
      const data = JSON.parse(body);
      res.end(JSON.stringify(data));
    });
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});
```

❌ Verbose, manual routing, manual body parsing

**Express:**
```typescript
import express from 'express';

const app = express();
app.use(express.json());

app.get('/students', (req, res) => {
  res.json({ students: [] });
});

app.post('/students', (req, res) => {
  res.json(req.body);  // Body already parsed!
});
```

✅ Clean, simple, built-in features

**Benefits:**
- Routing system
- Middleware ecosystem
- Built-in body parsers
- Template engines
- Error handling
- Large community"

### Q2: Explain middleware in Express and give examples

**Answer:**

"Middleware functions have access to `req`, `res`, and `next`. They execute in sequence before the route handler.

**Flow:**
```
Request → Middleware 1 → Middleware 2 → Route Handler → Response
```

**Example 1: Logger**
```typescript
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();  // Must call next() to continue
};

app.use(logger);
```

**Example 2: Authentication**
```typescript
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;

  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  try {
    req.user = verifyToken(token);
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Use on specific routes
app.get('/api/students', authenticate, (req, res) => {
  // req.user available here
  res.json({ user: req.user });
});
```

**Types of middleware:**
1. **Application-level:** `app.use(middleware)`
2. **Router-level:** `router.use(middleware)`
3. **Error-handling:** 4 parameters `(err, req, res, next)`
4. **Built-in:** `express.json()`, `express.static()`
5. **Third-party:** `cors()`, `helmet()`

**Order matters:**
```typescript
app.use(logger);       // Runs first
app.use(authenticate); // Runs second
app.get('/students', handler);  // Runs last
```

In my school admin system, I use middleware for:
- Authentication (JWT verification)
- Error handling (centralized)
- Request logging
- File upload (Multer)
- Body parsing"

### Q3: How do you handle errors in Express?

**Answer:**

"I use a centralized error handler with custom error classes.

**1. Custom Error Class:**
```typescript
class ErrorBase extends Error {
  constructor(
    public message: string,
    public statusCode: number,
    public errorCode: string
  ) {
    super(message);
  }
}
```

**2. Error Handler Middleware (must have 4 parameters!):**
```typescript
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err);

  if (err instanceof ErrorBase) {
    return res.status(err.statusCode).json({
      error: err.message,
      code: err.errorCode
    });
  }

  // Unexpected errors
  res.status(500).json({
    error: 'Internal Server Error',
    code: 'INTERNAL_ERROR'
  });
});
```

**3. Use in routes:**
```typescript
app.get('/students/:id', async (req, res, next) => {
  try {
    const student = await db.Student.findByPk(req.params.id);

    if (!student) {
      throw new ErrorBase('Student not found', 404, 'NOT_FOUND');
    }

    res.json(student);
  } catch (error) {
    next(error);  // Pass to error handler
  }
});
```

**4. Async error wrapper (avoid try/catch):**
```typescript
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Usage
app.get('/students', asyncHandler(async (req, res) => {
  const students = await db.Student.findAll();
  res.json(students);
}));
```

**Benefits:**
- Centralized error handling
- Consistent error format
- Clean route handlers
- Easy to add error logging/monitoring"

### Q4: How would you implement authentication in Express?

**Answer:**

"I use **JWT (JSON Web Tokens)** with bcrypt for password hashing.

**1. Register:**
```typescript
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';

app.post('/api/auth/register', async (req, res) => {
  const { email, password, name } = req.body;

  // Hash password (12 rounds)
  const passwordHash = await bcrypt.hash(password, 12);

  // Create user
  const user = await db.User.create({ email, name, passwordHash });

  // Generate JWT
  const token = jwt.sign(
    { userId: user.id, email },
    process.env.JWT_SECRET!,
    { expiresIn: '24h' }
  );

  res.status(201).json({ token, user: { id: user.id, email, name } });
});
```

**2. Login:**
```typescript
app.post('/api/auth/login', async (req, res) => {
  const { email, password } = req.body;

  const user = await db.User.findOne({ where: { email } });
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  const isValid = await bcrypt.compare(password, user.passwordHash);
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  const token = jwt.sign(
    { userId: user.id, email, role: user.role },
    process.env.JWT_SECRET!,
    { expiresIn: '24h' }
  );

  res.json({ token, user: { id: user.id, email, name: user.name } });
});
```

**3. Auth middleware:**
```typescript
const authenticate = (req, res, next) => {
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
```

**4. Protected routes:**
```typescript
app.get('/api/students', authenticate, async (req, res) => {
  const students = await db.Student.findAll();
  res.json(students);
});
```

**5. Role-based authorization:**
```typescript
const authorize = (...roles) => (req, res, next) => {
  if (!roles.includes(req.user.role)) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  next();
};

app.delete('/api/students/:id', authenticate, authorize('ADMIN'), handler);
```

**Security:**
- ✅ Passwords hashed with bcrypt
- ✅ JWT stored in HTTP-only cookie or localStorage
- ✅ Tokens expire (24h)
- ✅ Secrets in environment variables"

### Q5: How do you optimize Express performance?

**Answer:**

"I use several strategies:

**1. Caching (Redis):**
```typescript
const redis = new Redis();

app.get('/api/students', async (req, res) => {
  const cached = await redis.get('students');
  if (cached) {
    return res.json(JSON.parse(cached));
  }

  const students = await db.Student.findAll();
  await redis.setex('students', 300, JSON.stringify(students));
  res.json(students);
});
```

**2. Compression:**
```typescript
import compression from 'compression';
app.use(compression());  // Gzip responses (90% size reduction)
```

**3. Database optimization:**
```typescript
// Add indexes
CREATE INDEX idx_student_email ON students(email);

// Connection pooling
const sequelize = new Sequelize({
  pool: { max: 10, min: 1 }
});

// Pagination
const students = await db.Student.findAll({
  limit: 10,
  offset: (page - 1) * 10
});
```

**4. Parallel queries:**
```typescript
// ❌ Sequential (300ms)
const student = await db.Student.findByPk(id);
const classes = await db.Class.findAll({ where: { studentId: id } });

// ✅ Parallel (100ms)
const [student, classes] = await Promise.all([
  db.Student.findByPk(id),
  db.Class.findAll({ where: { studentId: id } })
]);
```

**5. Load balancing (PM2):**
```bash
pm2 start server.js -i max  # Cluster mode (use all CPUs)
```

**6. Monitoring:**
```typescript
// Track slow queries
const start = Date.now();
const result = await query();
const duration = Date.now() - start;

if (duration > 100) {
  console.warn('Slow query:', duration);
}
```

**Results in my project:**
- API response time: 500ms → 50ms (10x faster)
- Memory usage: 80% → 40%
- Throughput: 100 RPS → 1000 RPS"

### Q6: What's the difference between app.use() and app.get()?

**Answer:**

"**`app.use()`** - Middleware for ALL HTTP methods

```typescript
// Runs for ALL methods (GET, POST, PUT, DELETE, etc.)
app.use('/api/students', middleware);

// Matches:
// GET /api/students
// POST /api/students
// PUT /api/students/:id
// etc.
```

**`app.get()`** - Route handler for GET only

```typescript
// Only runs for GET requests
app.get('/api/students', handler);

// Matches:
// GET /api/students ✅

// Doesn't match:
// POST /api/students ❌
// PUT /api/students ❌
```

**Key differences:**

| Feature | app.use() | app.get() |
|---------|-----------|-----------|
| Purpose | Middleware | Route handler |
| HTTP Methods | ALL | GET only |
| Path matching | Prefix | Exact |
| Use case | Middleware, sub-routers | Route endpoints |

**Examples:**

```typescript
// Middleware (runs for all routes)
app.use(express.json());
app.use(logger);

// Mount router
app.use('/api', apiRouter);

// Route handlers
app.get('/students', getStudents);
app.post('/students', createStudent);
```

**Path matching difference:**

```typescript
app.use('/api', middleware);
// Matches: /api, /api/students, /api/classes

app.get('/api', handler);
// Matches: /api only
// Doesn't match: /api/students
```"

---

## Best Practices Summary

### Architecture Checklist

```
☐ Use MVC pattern (Models, Views, Controllers)
☐ Separate routes, controllers, services
☐ Use repository pattern for database access
☐ Keep routes thin, logic in services
☐ Use dependency injection
```

### Security Checklist

```
☐ Use helmet for security headers
☐ Implement rate limiting
☐ Use CORS with whitelist
☐ Validate all input
☐ Hash passwords (bcrypt)
☐ Use JWT for authentication
☐ Store secrets in environment variables
☐ Implement HTTPS in production
```

### Performance Checklist

```
☐ Use connection pooling
☐ Implement caching (Redis)
☐ Use compression
☐ Add database indexes
☐ Implement pagination
☐ Use parallel queries
☐ Enable clustering (PM2)
```

### Error Handling Checklist

```
☐ Use centralized error handler
☐ Create custom error classes
☐ Log errors appropriately
☐ Never expose stack traces to clients
☐ Use async error wrapper
```

---

**Express.js is powerful because of its simplicity and flexibility. The middleware system makes it easy to add features, and the ecosystem provides solutions for almost every need!**
