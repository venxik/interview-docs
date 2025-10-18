# Scalability & Performance Guide

## Table of Contents
1. [Scalability Fundamentals](#scalability-fundamentals)
2. [Vertical vs Horizontal Scaling](#vertical-vs-horizontal-scaling)
3. [Performance Metrics](#performance-metrics)
4. [Bottleneck Identification](#bottleneck-identification)
5. [Application-Level Performance](#application-level-performance)
6. [Database Performance](#database-performance)
7. [Network Performance](#network-performance)
8. [Monitoring & Observability](#monitoring--observability)
9. [Performance Testing](#performance-testing)
10. [Common Scalability Patterns](#common-scalability-patterns)
11. [Interview Questions](#interview-questions)

---

## Scalability Fundamentals

### What is Scalability?

**Scalability:** Ability of a system to handle increased load without degrading performance

### Types of Scalability

#### 1. Vertical Scalability (Scale Up)

```
Small Server                 Large Server
┌─────────────┐             ┌─────────────┐
│ 2 CPU cores │   →         │ 32 CPU cores│
│ 4 GB RAM    │   →         │ 128 GB RAM  │
│ 100 GB SSD  │   →         │ 2 TB SSD    │
└─────────────┘             └─────────────┘

Add more resources to existing machine
```

**Pros:**
- ✅ Simple (no code changes)
- ✅ No data synchronization issues
- ✅ Lower licensing costs (1 server vs many)

**Cons:**
- ❌ Hardware limits (finite ceiling)
- ❌ Single point of failure
- ❌ Expensive (exponential cost)
- ❌ Downtime during upgrades

**Use When:**
- Early stage (< 10K users)
- Monolithic application
- Limited budget
- Simple deployment

#### 2. Horizontal Scalability (Scale Out)

```
1 Server                    Many Servers
┌─────────┐                ┌────┐ ┌────┐ ┌────┐
│         │    →           │    │ │    │ │    │
│         │    →           │    │ │    │ │    │
└─────────┘                └────┘ └────┘ └────┘

Add more machines
```

**Pros:**
- ✅ Nearly unlimited scaling
- ✅ Fault tolerance (redundancy)
- ✅ Cost-effective (commodity hardware)
- ✅ Zero-downtime deployments

**Cons:**
- ❌ Complex architecture
- ❌ Data consistency challenges
- ❌ Network latency
- ❌ More operational overhead

**Use When:**
- Growth expected (millions of users)
- Need high availability
- Stateless applications
- Cloud-native

### Scalability Cube (3 Dimensions)

```
         Y-Axis (Functional)
         Split by function
              ↑
              │
              │
    ┌─────────┼─────────┐
    │         │         │
    │    User Service   │
    │    Order Service  │
    │    Payment Service│
    │         │         │
    └─────────┼─────────┘
              │
              └────────→ X-Axis (Horizontal)
             /           Clone servers
            /
           ↙
    Z-Axis (Data)
    Partition data
```

**X-Axis: Horizontal Duplication**
- Clone identical instances
- Load balance across clones
- Example: 10 identical API servers

**Y-Axis: Functional Decomposition**
- Split by feature/function
- Microservices architecture
- Example: User service, Order service, Payment service

**Z-Axis: Data Partitioning**
- Split by data (sharding)
- Each instance handles subset of data
- Example: Users A-M on Server 1, N-Z on Server 2

---

## Vertical vs Horizontal Scaling

### Decision Matrix

| Factor | Vertical Scaling | Horizontal Scaling |
|--------|-----------------|-------------------|
| **Cost** | Exponential ($$$$) | Linear ($) |
| **Complexity** | Low | High |
| **Limit** | Hardware limit | Nearly unlimited |
| **Availability** | Single point of failure | Highly available |
| **Deployment** | Downtime required | Rolling updates |
| **Data Consistency** | Simple | Complex |
| **Best For** | Databases | Stateless web servers |

### Real-World Example: From 1K to 1M Users

#### Phase 1: 0-1K Users (Vertical Scaling)

```
┌──────────────────────┐
│   Single Server      │
│  ┌────────────────┐  │
│  │  Web App       │  │
│  │  (Node.js)     │  │
│  ├────────────────┤  │
│  │  Database      │  │
│  │  (MySQL)       │  │
│  └────────────────┘  │
│  4 cores, 8GB RAM    │
└──────────────────────┘

Cost: $50/month
```

#### Phase 2: 1K-10K Users (Separate DB + Cache)

```
┌─────────────┐     ┌──────────────┐
│  Web Server │────→│   Database   │
│  (Node.js)  │     │   (MySQL)    │
│  4 cores    │  ┌─→│   8 cores    │
└──────┬──────┘  │  └──────────────┘
       │         │
       └─────────┘
     Cache (Redis)
     2 cores

Cost: $200/month
```

#### Phase 3: 10K-100K Users (Horizontal Scaling Begins)

```
       ┌──────────────┐
       │Load Balancer │
       └──────┬───────┘
              │
    ┌─────────┼─────────┐
    │         │         │
┌───┴───┐ ┌───┴───┐ ┌───┴───┐
│Web    │ │Web    │ │Web    │  ← Horizontal scaling
│Server1│ │Server2│ │Server3│
└───┬───┘ └───┬───┘ └───┬───┘
    │         │         │
    └─────────┼─────────┘
              │
       ┌──────┴───────┐
       │    Redis     │
       └──────┬───────┘
              │
       ┌──────┴───────┐
       │   MySQL      │
       │  (Primary)   │
       └──────┬───────┘
              │
       ┌──────┴───────┐
       │ Read Replicas│
       └──────────────┘

Cost: $1,000/month
```

#### Phase 4: 100K-1M Users (Full Distribution)

```
       ┌─────────────┐
       │     DNS     │
       └──────┬──────┘
              │
       ┌──────┴──────┐
       │     CDN     │  ← Static content
       └──────┬──────┘
              │
       ┌──────┴──────┐
       │ Load Balancer│
       └──────┬──────┘
              │
    ┌─────────┼─────────────┐
    │         │             │
┌───┴────┐ ┌──┴─────┐ ┌────┴────┐
│API     │ │API     │ │API      │  ← Auto-scaling
│Servers │ │Servers │ │Servers  │     (10-100 servers)
│(Pool)  │ │(Pool)  │ │(Pool)   │
└───┬────┘ └───┬────┘ └────┬────┘
    │          │           │
    └──────────┼───────────┘
               │
        ┌──────┴──────┐
        │Redis Cluster│  ← Distributed cache
        │  (Sharded)  │
        └──────┬──────┘
               │
        ┌──────┴──────┐
        │Message Queue│  ← Async processing
        │  (Kafka)    │
        └──────┬──────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───┴────┐ ┌───┴────┐ ┌──┴─────┐
│DB      │ │DB      │ │DB      │  ← Database sharding
│Shard 1 │ │Shard 2 │ │Shard 3 │
│(Users  │ │(Users  │ │(Users  │
│ A-F)   │ │ G-M)   │ │ N-Z)   │
└────────┘ └────────┘ └────────┘
    │          │          │
  Replicas   Replicas   Replicas  ← Read replicas

Cost: $10,000+/month
```

---

## Performance Metrics

### Key Metrics to Track

#### 1. Latency

**Definition:** Time to complete a single request

```
Client Request ──┐
                 │  ← Latency
Response ────────┘

Common Targets:
- API endpoints: < 200ms (p95)
- Database queries: < 50ms (p95)
- Cache lookups: < 5ms (p95)
```

**Percentiles (Better than Average):**

```
100 requests, response times in ms:
[10, 10, 10, 10, 10, 10, 10, 10, 10, 10,  ← 90 fast requests
 10, 10, 10, 10, 10, 10, 10, 10, 10, 10,
 10, 10, 10, 10, 10, 10, 10, 10, 10, 10,
 10, 10, 10, 10, 10, 10, 10, 10, 10, 10,
 10, 10, 10, 10, 10, 10, 10, 10, 10, 10,
 10, 10, 10, 10, 10, 10, 10, 10, 10, 10,
 10, 10, 10, 10, 10, 10, 10, 10, 10, 10,
 10, 10, 10, 10, 10, 10, 10, 10, 10, 10,
 10, 10, 10, 10, 10, 10, 10, 10, 10, 10,
 50, 100, 200, 500, 1000, 2000, 5000, 10000, 20000, 30000]  ← 10 slow requests

Average: 650ms  ← Misleading! (skewed by outliers)
Median (p50): 10ms
p95: 2000ms  ← 95% of requests under 2s
p99: 20000ms ← 99% of requests under 20s

✅ Track percentiles (p50, p95, p99), not just average
```

#### 2. Throughput

**Definition:** Requests processed per unit time

```
Throughput = Requests per second (RPS) or Transactions per second (TPS)

Examples:
- Low: 10 RPS (small blog)
- Medium: 1,000 RPS (startup)
- High: 100,000 RPS (Netflix, Google)
```

#### 3. Error Rate

```
Error Rate = (Failed Requests / Total Requests) × 100%

Target: < 0.1% (99.9% success rate)

HTTP Status Codes:
- 2xx: Success ✅
- 4xx: Client errors (count separately)
- 5xx: Server errors ❌ (track this!)
```

#### 4. Availability

```
Availability = (Uptime / Total Time) × 100%

Service Level Objectives (SLOs):
┌──────────┬─────────────────┬────────────────────┐
│ SLO      │ Downtime/Year   │ Downtime/Month     │
├──────────┼─────────────────┼────────────────────┤
│ 99%      │ 3.65 days       │ 7.2 hours          │
│ 99.9%    │ 8.76 hours      │ 43.2 minutes       │
│ 99.95%   │ 4.38 hours      │ 21.6 minutes       │
│ 99.99%   │ 52.56 minutes   │ 4.32 minutes       │
│ 99.999%  │ 5.26 minutes    │ 25.9 seconds       │
└──────────┴─────────────────┴────────────────────┘
```

#### 5. Resource Utilization

```
CPU Usage: < 70% average (leave headroom for spikes)
Memory Usage: < 80% (avoid swapping)
Disk I/O: < 80% capacity
Network Bandwidth: < 70%

High utilization → Time to scale!
```

### Performance Budget

**Set performance goals upfront:**

```typescript
// Performance budget example
const PERFORMANCE_BUDGET = {
  // Page load
  firstContentfulPaint: 1500,  // 1.5s
  timeToInteractive: 3000,     // 3s
  totalPageSize: 2000000,      // 2MB

  // API endpoints
  getUserProfile: 100,         // 100ms p95
  searchStudents: 200,         // 200ms p95
  enrollStudent: 500,          // 500ms p95

  // Database queries
  simpleQuery: 10,             // 10ms p95
  complexQuery: 50,            // 50ms p95
};

// Alert if budget exceeded
if (actualLatency > PERFORMANCE_BUDGET.getUserProfile) {
  alert('Performance budget exceeded!');
}
```

---

## Bottleneck Identification

### The 4 Horsemen of Performance Bottlenecks

#### 1. CPU-Bound

**Symptoms:**
- High CPU usage (> 80%)
- Slow computation-heavy operations
- Image/video processing slow

**Identification:**

```bash
# Check CPU usage
top
htop

# Profile CPU (Node.js)
node --prof app.js
node --prof-process isolate-*.log
```

**Solutions:**
- ✅ Optimize algorithms (O(n²) → O(n log n))
- ✅ Use caching (avoid recomputation)
- ✅ Horizontal scaling (more servers)
- ✅ Offload to workers (async processing)

**Example:**

```typescript
// ❌ CPU-intensive (blocks event loop)
app.get('/process-image', (req, res) => {
  const result = heavyImageProcessing(req.file);  // Blocks for 5 seconds
  res.json(result);
});

// ✅ Offload to worker
app.get('/process-image', async (req, res) => {
  const jobId = await queue.add('process-image', { file: req.file });
  res.json({ jobId, status: 'processing' });
});

// Worker processes in background (doesn't block API)
queue.process('process-image', async (job) => {
  return heavyImageProcessing(job.data.file);
});
```

#### 2. Memory-Bound

**Symptoms:**
- High memory usage (> 80%)
- Out of memory errors
- Frequent garbage collection pauses

**Identification:**

```bash
# Check memory usage
free -m
vmstat 1

# Node.js heap snapshot
node --inspect app.js
# Chrome DevTools → Memory → Take heap snapshot
```

**Solutions:**
- ✅ Fix memory leaks (unclosed connections, event listeners)
- ✅ Limit data size (pagination, streaming)
- ✅ Increase memory (vertical scaling)
- ✅ Use efficient data structures

**Example:**

```typescript
// ❌ Memory leak (global cache grows forever)
const cache = {};

app.get('/student/:id', async (req, res) => {
  const student = await Student.findByPk(req.params.id);
  cache[req.params.id] = student;  // Never cleaned up!
  res.json(student);
});

// ✅ Use Redis with TTL
app.get('/student/:id', async (req, res) => {
  let student = await redis.get(`student:${req.params.id}`);

  if (!student) {
    student = await Student.findByPk(req.params.id);
    await redis.setex(`student:${req.params.id}`, 3600, JSON.stringify(student));
  }

  res.json(JSON.parse(student));
});

// ❌ Load entire table into memory
const allStudents = await Student.findAll();  // 1 million records!

// ✅ Stream or paginate
const students = await Student.findAll({
  limit: 100,
  offset: (page - 1) * 100
});
```

#### 3. I/O-Bound (Disk)

**Symptoms:**
- High disk I/O wait (iowait > 20%)
- Slow database queries
- File operations slow

**Identification:**

```bash
# Check disk I/O
iostat -x 1

# Disk usage
df -h

# Find slow queries (MySQL)
SHOW FULL PROCESSLIST;
```

**Solutions:**
- ✅ Add indexes (database)
- ✅ Use SSD instead of HDD
- ✅ Cache frequently accessed data
- ✅ Optimize queries (reduce joins)

**Example:**

```typescript
// ❌ N+1 query problem (11 database queries)
const classes = await Class.findAll();  // 1 query

for (const cls of classes) {
  const students = await cls.getStudents();  // 10 queries
  console.log(students.length);
}

// ✅ Single query with JOIN
const classes = await Class.findAll({
  include: [{ model: Student }]  // 1 query with JOIN
});

for (const cls of classes) {
  console.log(cls.students.length);
}
```

#### 4. Network-Bound

**Symptoms:**
- High network latency
- Timeouts
- Slow external API calls

**Identification:**

```bash
# Check network latency
ping google.com
traceroute google.com

# Monitor network traffic
iftop
nethogs
```

**Solutions:**
- ✅ Use CDN (reduce distance)
- ✅ Compress responses (gzip)
- ✅ Reduce payload size (pagination)
- ✅ Connection pooling (reuse connections)
- ✅ Parallel requests (don't wait serially)

**Example:**

```typescript
// ❌ Serial external API calls (slow)
const user = await fetch('/api/user/123');
const orders = await fetch('/api/orders/123');
const reviews = await fetch('/api/reviews/123');
// Total: 300ms + 300ms + 300ms = 900ms

// ✅ Parallel requests
const [user, orders, reviews] = await Promise.all([
  fetch('/api/user/123'),
  fetch('/api/orders/123'),
  fetch('/api/reviews/123')
]);
// Total: max(300ms, 300ms, 300ms) = 300ms

// ✅ Compress responses
app.use(compression());  // gzip compression

// ✅ Reduce payload
// ❌ Send entire object
res.json(student);  // 5KB

// ✅ Send only needed fields
res.json({ id: student.id, name: student.name });  // 0.1KB
```

### Performance Profiling Tools

```
Browser:
- Chrome DevTools (Lighthouse, Performance tab)
- WebPageTest

Backend:
- Node.js: clinic, 0x, autocannon
- Python: cProfile, py-spy
- Java: JProfiler, VisualVM

Database:
- MySQL: EXPLAIN, slow query log
- PostgreSQL: EXPLAIN ANALYZE
- MongoDB: .explain()

Infrastructure:
- New Relic
- Datadog
- Grafana + Prometheus
```

---

## Application-Level Performance

### 1. Code Optimization

#### Algorithm Complexity

```typescript
// ❌ O(n²) - Slow for large datasets
function findDuplicates(students: Student[]): Student[] {
  const duplicates = [];

  for (let i = 0; i < students.length; i++) {
    for (let j = i + 1; j < students.length; j++) {
      if (students[i].email === students[j].email) {
        duplicates.push(students[i]);
      }
    }
  }

  return duplicates;
}
// 10,000 students → 100,000,000 comparisons!

// ✅ O(n) - Fast
function findDuplicates(students: Student[]): Student[] {
  const seen = new Set<string>();
  const duplicates = new Set<Student>();

  for (const student of students) {
    if (seen.has(student.email)) {
      duplicates.add(student);
    } else {
      seen.add(student.email);
    }
  }

  return Array.from(duplicates);
}
// 10,000 students → 10,000 operations
```

#### Avoid Unnecessary Work

```typescript
// ❌ Recompute on every request
app.get('/stats', async (req, res) => {
  const totalStudents = await Student.count();
  const totalClasses = await Class.count();
  const avgStudentsPerClass = totalStudents / totalClasses;

  res.json({ totalStudents, totalClasses, avgStudentsPerClass });
});

// ✅ Cache expensive computations
const CACHE_TTL = 300; // 5 minutes

app.get('/stats', async (req, res) => {
  const cached = await redis.get('stats');
  if (cached) return res.json(JSON.parse(cached));

  const stats = {
    totalStudents: await Student.count(),
    totalClasses: await Class.count(),
    avgStudentsPerClass: 0
  };
  stats.avgStudentsPerClass = stats.totalStudents / stats.totalClasses;

  await redis.setex('stats', CACHE_TTL, JSON.stringify(stats));
  res.json(stats);
});
```

### 2. Async Processing

```typescript
// ❌ Synchronous (blocks for 5 seconds)
app.post('/enroll-bulk', async (req, res) => {
  const { studentIds, classId } = req.body;

  for (const studentId of studentIds) {
    await enrollStudent(studentId, classId);  // 50ms each
    await sendEnrollmentEmail(studentId);     // 100ms each
  }
  // 1000 students × 150ms = 150 seconds!

  res.json({ message: 'Enrolled' });
});

// ✅ Async with queue
app.post('/enroll-bulk', async (req, res) => {
  const { studentIds, classId } = req.body;

  // Queue job for background processing
  const job = await enrollmentQueue.add('bulk-enroll', {
    studentIds,
    classId
  });

  res.json({ jobId: job.id, status: 'processing' });
  // Response in < 10ms!
});

// Background worker processes queue
enrollmentQueue.process('bulk-enroll', async (job) => {
  const { studentIds, classId } = job.data;

  // Process in parallel
  await Promise.all(
    studentIds.map(async (studentId) => {
      await enrollStudent(studentId, classId);
      await sendEnrollmentEmail(studentId);
    })
  );
});
```

### 3. Connection Pooling

```typescript
// ❌ Create new connection per request
async function getStudent(id: number) {
  const connection = await mysql.createConnection(dbConfig);  // Slow!
  const [rows] = await connection.execute('SELECT * FROM students WHERE id = ?', [id]);
  await connection.end();
  return rows[0];
}

// ✅ Use connection pool (reuse connections)
const pool = mysql.createPool({
  host: 'localhost',
  user: 'root',
  password: 'password',
  database: 'school',
  waitForConnections: true,
  connectionLimit: 10,      // Max connections
  queueLimit: 0
});

async function getStudent(id: number) {
  const [rows] = await pool.execute('SELECT * FROM students WHERE id = ?', [id]);
  return rows[0];  // Connection automatically returned to pool
}
```

### 4. Lazy Loading

```typescript
// ❌ Eager load everything (slow initial load)
class Student {
  constructor() {
    this.classes = await this.getClasses();         // Query 1
    this.teachers = await this.getTeachers();       // Query 2
    this.grades = await this.getGrades();           // Query 3
    this.attendance = await this.getAttendance();   // Query 4
  }
}

// ✅ Lazy load on demand
class Student {
  private _classes?: Class[];

  async getClasses(): Promise<Class[]> {
    if (!this._classes) {
      this._classes = await db.query('SELECT ...');  // Only when needed
    }
    return this._classes;
  }
}
```

---

## Database Performance

### 1. Query Optimization

```sql
-- ❌ Slow query (no index, SELECT *)
SELECT *
FROM students s
JOIN class_students cs ON s.id = cs.student_id
WHERE s.email = 'john@school.com';

-- Execution time: 500ms (full table scan)

-- ✅ Optimized query
-- 1. Add index
CREATE INDEX idx_student_email ON students(email);

-- 2. Select only needed columns
SELECT s.id, s.name, s.email
FROM students s
JOIN class_students cs ON s.id = cs.student_id
WHERE s.email = 'john@school.com';

-- Execution time: 5ms (index lookup)
```

### 2. Use EXPLAIN

```sql
EXPLAIN SELECT * FROM students WHERE email = 'john@school.com';

-- Check output:
-- ✅ type: const/ref (uses index)
-- ❌ type: ALL (full table scan)
-- ✅ key: idx_email (index used)
-- ❌ key: NULL (no index)
-- ✅ rows: 1 (few rows scanned)
-- ❌ rows: 100000 (many rows scanned)
```

### 3. Batch Operations

```typescript
// ❌ 1000 individual queries (slow)
for (const student of students) {
  await db.query('INSERT INTO students (name, email) VALUES (?, ?)',
    [student.name, student.email]
  );
}
// Time: 1000 × 5ms = 5000ms

// ✅ Single batch query (fast)
await db.query(
  'INSERT INTO students (name, email) VALUES ?',
  [students.map(s => [s.name, s.email])]
);
// Time: 50ms
```

### 4. Read Replicas

```typescript
// Write to primary
async function createStudent(data: StudentData) {
  return await primaryDB.Student.create(data);
}

// Read from replica
async function getStudent(id: number) {
  return await replicaDB.Student.findByPk(id);
}

// Read from replica for heavy queries
async function searchStudents(query: string) {
  return await replicaDB.Student.findAll({
    where: {
      name: { [Op.like]: `%${query}%` }
    }
  });
}
```

---

## Network Performance

### 1. Compression

```typescript
import compression from 'compression';

// Enable gzip compression
app.use(compression());

// Response size:
// Without compression: 100 KB
// With compression: 10 KB (10x smaller)
```

### 2. HTTP/2

```typescript
import http2 from 'http2';
import fs from 'fs';

// HTTP/2 allows multiplexing (multiple requests over single connection)
const server = http2.createSecureServer({
  key: fs.readFileSync('private-key.pem'),
  cert: fs.readFileSync('certificate.pem')
});

// Benefits:
// ✅ Header compression
// ✅ Multiplexing (parallel requests)
// ✅ Server push
```

### 3. CDN (Content Delivery Network)

```
Without CDN:
User (Japan) → Server (USA) → 200ms latency

With CDN:
User (Japan) → CDN Edge (Tokyo) → 10ms latency
                       ↓ (cache miss)
                 Server (USA) → 200ms (first request only)

90%+ cache hit rate → 10ms average latency
```

### 4. Reduce Requests (Bundling)

```html
<!-- ❌ 10 separate requests -->
<script src="/js/file1.js"></script>
<script src="/js/file2.js"></script>
...
<script src="/js/file10.js"></script>

<!-- ✅ 1 bundled request -->
<script src="/js/bundle.min.js"></script>
```

---

## Monitoring & Observability

### The 3 Pillars

#### 1. Metrics

**Time-series numeric data**

```typescript
// Counter (always increases)
httpRequestsTotal.inc();

// Gauge (can go up or down)
activeUsers.set(150);

// Histogram (distribution)
httpRequestDuration.observe(0.250);  // 250ms

// Summary (quantiles)
apiLatency.observe(0.100);
```

**Example: Prometheus + Grafana**

```typescript
import { Counter, Histogram, register } from 'prom-client';

const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status']
});

const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'route']
});

// Middleware to track metrics
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;

    httpRequestsTotal.inc({
      method: req.method,
      route: req.route?.path || req.path,
      status: res.statusCode
    });

    httpRequestDuration.observe({
      method: req.method,
      route: req.route?.path || req.path
    }, duration);
  });

  next();
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

#### 2. Logs

**Structured logging**

```typescript
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Structured log
logger.info('Student enrolled', {
  studentId: 123,
  classId: 456,
  timestamp: new Date(),
  userId: req.user.id
});

// ✅ Can query logs:
// "Find all enrollments for student 123 in last hour"
```

#### 3. Traces

**Distributed tracing (track request across services)**

```typescript
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('school-admin-service');

app.get('/student/:id', async (req, res) => {
  const span = tracer.startSpan('get_student');

  try {
    // Trace database query
    const dbSpan = tracer.startSpan('db_query', { parent: span });
    const student = await Student.findByPk(req.params.id);
    dbSpan.end();

    // Trace external API call
    const apiSpan = tracer.startSpan('external_api', { parent: span });
    const grades = await fetchGrades(student.id);
    apiSpan.end();

    res.json({ student, grades });
  } finally {
    span.end();
  }
});

// Trace shows:
// get_student (200ms)
// ├─ db_query (50ms)
// └─ external_api (150ms)  ← Bottleneck identified!
```

### Alerting Rules

```yaml
# Prometheus alert rules
groups:
  - name: api_alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        annotations:
          summary: "High error rate (> 5%)"

      # High latency
      - alert: HighLatency
        expr: histogram_quantile(0.95, http_request_duration_seconds) > 1
        for: 5m
        annotations:
          summary: "API latency p95 > 1s"

      # Low availability
      - alert: ServiceDown
        expr: up == 0
        for: 1m
        annotations:
          summary: "Service is down"
```

---

## Performance Testing

### Types of Performance Tests

#### 1. Load Testing

**Test system under expected load**

```bash
# Apache Bench
ab -n 10000 -c 100 http://localhost:3000/api/students
# 10,000 requests, 100 concurrent

# k6 (modern tool)
k6 run load-test.js
```

```javascript
// load-test.js (k6)
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up to 100 users
    { duration: '5m', target: 100 },   // Stay at 100 users
    { duration: '2m', target: 0 },     // Ramp down to 0
  ],
  thresholds: {
    http_req_duration: ['p(95)<200'],  // 95% of requests < 200ms
    http_req_failed: ['rate<0.01'],    // < 1% errors
  },
};

export default function () {
  const res = http.get('http://localhost:3000/api/students');

  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200,
  });

  sleep(1);
}
```

#### 2. Stress Testing

**Test system beyond normal load (find breaking point)**

```javascript
export const options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '5m', target: 500 },   // Push to 500 users
    { duration: '5m', target: 1000 },  // Push to 1000 users
    { duration: '5m', target: 2000 },  // Find breaking point
  ],
};
```

#### 3. Spike Testing

**Test sudden traffic spikes**

```javascript
export const options = {
  stages: [
    { duration: '10s', target: 100 },
    { duration: '1m', target: 5000 },  // Sudden spike!
    { duration: '10s', target: 100 },
  ],
};
```

#### 4. Soak Testing (Endurance)

**Test system over extended period (find memory leaks)**

```javascript
export const options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '24h', target: 100 },  // Run for 24 hours
  ],
};
```

---

## Common Scalability Patterns

### 1. Database Scaling Patterns

#### Master-Replica

```
┌─────────┐
│ Primary │ ← Writes
└────┬────┘
     │ Replication
  ┌──┴──┬──────┐
  ▼     ▼      ▼
┌───┐ ┌───┐  ┌───┐
│R1 │ │R2 │  │R3 │ ← Reads
└───┘ └───┘  └───┘
```

#### Sharding

```
Users A-F    Users G-M    Users N-Z
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Shard 1 │  │ Shard 2 │  │ Shard 3 │
└─────────┘  └─────────┘  └─────────┘
```

### 2. Caching Patterns

#### Cache-Aside

```
1. Check cache
2. If miss → Query DB → Store in cache
3. Return data
```

#### Write-Through

```
1. Write to cache
2. Write to DB (sync)
3. Return success
```

### 3. Async Processing

```
┌────────┐      ┌───────┐      ┌────────┐
│ Client │─────→│ Queue │─────→│ Worker │
└────────┘      └───────┘      └────────┘
  ↑                                 │
  └─────── Response (async) ────────┘
```

### 4. Circuit Breaker

```typescript
class CircuitBreaker {
  private failures = 0;
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  private threshold = 5;
  private timeout = 60000;  // 1 minute

  async execute(fn: Function) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailure > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }

  private onFailure() {
    this.failures++;
    this.lastFailure = Date.now();

    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}

// Usage
const breaker = new CircuitBreaker();

app.get('/external-api', async (req, res) => {
  try {
    const data = await breaker.execute(() =>
      fetch('https://slow-api.com/data')
    );
    res.json(data);
  } catch (error) {
    res.status(503).json({ error: 'Service temporarily unavailable' });
  }
});
```

---

## Interview Questions

### Q1: How would you scale a system from 1K to 1M users?

**Answer:**

"I would approach scaling incrementally:

**Phase 1: 0-10K users (Single server)**
- Single server with web app + database
- Cost: ~$100/month
- Vertical scaling as needed

**Phase 2: 10K-100K users (Separate tiers)**
- Separate web and database servers
- Add Redis cache for frequently accessed data
- Use CDN for static assets
- Read replica for database
- Cost: ~$500/month

**Phase 3: 100K-500K users (Horizontal scaling)**
- Multiple web servers behind load balancer
- Database sharding if needed
- Message queue for async processing
- Auto-scaling based on load
- Cost: ~$2,000/month

**Phase 4: 500K-1M users (Distributed system)**
- Microservices architecture (if needed)
- Multiple database shards
- Redis cluster for distributed caching
- CDN for global distribution
- Multi-region deployment
- Cost: ~$10,000+/month

**Key principles:**
1. Start simple, scale incrementally
2. Measure before optimizing (data-driven decisions)
3. Horizontal scaling for stateless components
4. Cache aggressively (80-20 rule)
5. Async processing for heavy operations
6. Monitor everything (metrics, logs, traces)"

### Q2: Your API is slow. How do you diagnose and fix it?

**Answer:**

"I would follow this systematic approach:

**1. Identify the bottleneck:**
- Check metrics: p95 latency, error rate, throughput
- Use APM tools (New Relic, Datadog) to identify slow endpoints
- Check resource utilization: CPU, memory, disk I/O, network

**2. Profile the slow endpoint:**
- Add logging to measure each step
- Use profiling tools (Node.js: clinic, 0x)
- Check database query performance (EXPLAIN)

**3. Common causes and fixes:**

**Database bottleneck:**
- ❌ Problem: Slow queries (missing indexes, N+1 queries)
- ✅ Solution: Add indexes, optimize queries, use eager loading

**External API bottleneck:**
- ❌ Problem: Waiting for 3rd party APIs serially
- ✅ Solution: Parallel requests, caching, circuit breaker

**CPU bottleneck:**
- ❌ Problem: Heavy computation blocking event loop
- ✅ Solution: Offload to worker queue, optimize algorithm

**Memory bottleneck:**
- ❌ Problem: Loading large datasets into memory
- ✅ Solution: Pagination, streaming, increase memory

**4. Verify fix:**
- Load test before and after
- Monitor in production
- Ensure latency improved without introducing errors

**Example from my school admin system:**
If `/api/students` was slow, I would:
1. Check EXPLAIN on the query
2. Add index on frequently filtered columns (email, name)
3. Implement caching for student list (Redis, 5 min TTL)
4. Paginate results (limit 100 per page)
5. Measure: 500ms → 50ms (10x improvement)"

### Q3: Explain how you would design a rate limiter

**Answer:**

"I would use the **Token Bucket Algorithm** with Redis:

**Algorithm:**
- Each user gets a bucket with N tokens
- Each request consumes 1 token
- Tokens refill at rate R per second
- If bucket empty, reject request (429 Too Many Requests)

**Implementation:**

```typescript
import Redis from 'ioredis';
const redis = new Redis();

async function rateLimit(userId: string): Promise<boolean> {
  const key = `rate_limit:${userId}`;
  const maxTokens = 100;       // Bucket size
  const refillRate = 10;       // Tokens per second
  const now = Date.now() / 1000;

  // Get current tokens and last refill time
  const data = await redis.get(key);
  let tokens = maxTokens;
  let lastRefill = now;

  if (data) {
    const [savedTokens, savedTime] = data.split(':').map(Number);
    const elapsed = now - savedTime;
    tokens = Math.min(maxTokens, savedTokens + elapsed * refillRate);
    lastRefill = savedTime;
  }

  // Try to consume 1 token
  if (tokens >= 1) {
    tokens -= 1;
    await redis.setex(key, 3600, `${tokens}:${now}`);
    return true;  // ✅ Allow request
  } else {
    return false; // ❌ Rate limit exceeded
  }
}

// Middleware
app.use(async (req, res, next) => {
  const allowed = await rateLimit(req.user.id);

  if (!allowed) {
    return res.status(429).json({
      error: 'Too many requests',
      retryAfter: 10  // seconds
    });
  }

  next();
});
```

**Alternative approaches:**

1. **Fixed Window:** Simple counter per time window
   - Pros: Simple
   - Cons: Burst at window boundaries

2. **Sliding Window Log:** Track timestamp of each request
   - Pros: Accurate
   - Cons: High memory usage

3. **Leaky Bucket:** Queue requests, process at constant rate
   - Pros: Smooth traffic
   - Cons: Queueing delays

I would choose **Token Bucket** for most use cases due to simplicity and efficiency."

---

## Summary: Scalability Checklist

```
☐ Identify bottlenecks
  ☐ Profile application (CPU, memory, I/O)
  ☐ Monitor database queries
  ☐ Check external API latency

☐ Optimize application
  ☐ Improve algorithms (reduce time complexity)
  ☐ Add caching (Redis)
  ☐ Async processing (queues)
  ☐ Connection pooling

☐ Optimize database
  ☐ Add indexes
  ☐ Optimize queries (avoid N+1)
  ☐ Read replicas
  ☐ Sharding (if needed)

☐ Scale horizontally
  ☐ Load balancer
  ☐ Stateless servers (enable auto-scaling)
  ☐ Distributed cache

☐ Network optimization
  ☐ CDN for static assets
  ☐ Compression (gzip)
  ☐ HTTP/2

☐ Monitor & alert
  ☐ Metrics (Prometheus)
  ☐ Logs (structured logging)
  ☐ Traces (distributed tracing)
  ☐ Alerts (error rate, latency)

☐ Test
  ☐ Load testing
  ☐ Stress testing
  ☐ Soak testing
```

**Golden Rule:** Measure first, optimize second. Don't optimize prematurely!
