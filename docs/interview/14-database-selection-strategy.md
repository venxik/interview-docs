# Database Selection & Strategy Guide

## Table of Contents
1. [Database Types Overview](#database-types-overview)
2. [SQL vs NoSQL Decision Framework](#sql-vs-nosql-decision-framework)
3. [Relational Databases (SQL)](#relational-databases-sql)
4. [NoSQL Database Types](#nosql-database-types)
5. [Database Selection Decision Tree](#database-selection-decision-tree)
6. [Database Indexing Strategies](#database-indexing-strategies)
7. [Database Normalization](#database-normalization)
8. [Query Optimization](#query-optimization)
9. [ACID vs BASE](#acid-vs-base)
10. [Transactions & Isolation Levels](#transactions--isolation-levels)
11. [Common Database Interview Questions](#common-database-interview-questions)

---

## Database Types Overview

```
Databases
│
├── Relational (SQL)
│   ├── MySQL
│   ├── PostgreSQL
│   ├── Oracle
│   └── SQL Server
│
└── NoSQL
    ├── Document Store
    │   ├── MongoDB
    │   ├── CouchDB
    │   └── Firebase
    │
    ├── Key-Value Store
    │   ├── Redis
    │   ├── DynamoDB
    │   └── Memcached
    │
    ├── Column-Family Store
    │   ├── Cassandra
    │   ├── HBase
    │   └── ScyllaDB
    │
    ├── Graph Database
    │   ├── Neo4j
    │   ├── Amazon Neptune
    │   └── ArangoDB
    │
    ├── Time-Series Database
    │   ├── InfluxDB
    │   ├── TimescaleDB
    │   └── Prometheus
    │
    └── Search Engine
        ├── Elasticsearch
        ├── Solr
        └── Algolia
```

---

## SQL vs NoSQL Decision Framework

### Quick Decision Matrix

| Factor | SQL | NoSQL |
|--------|-----|-------|
| **Data Structure** | Structured, fixed schema | Flexible, schema-less |
| **Relationships** | Complex joins, foreign keys | Denormalized, embedded docs |
| **Transactions** | Strong ACID guarantees | Eventual consistency (mostly) |
| **Scalability** | Vertical (scale up) | Horizontal (scale out) |
| **Query Language** | Standard SQL | Database-specific |
| **Use Cases** | Banking, ERP, CRM | Social media, IoT, real-time |
| **Consistency** | Strong consistency | Eventual consistency |
| **Schema Changes** | Migrations required | Flexible, no migrations |

### When to Use SQL

✅ **Use SQL (Relational) when:**

1. **ACID compliance is critical**
   - Financial transactions
   - Inventory management
   - Order processing
   - Healthcare records

2. **Complex queries with joins**
   - Reporting and analytics
   - Data warehousing
   - Business intelligence

3. **Data is highly structured**
   - Well-defined schema
   - Relationships between entities
   - Data integrity is crucial

4. **Data consistency over availability**
   - Bank account balances
   - Stock trading
   - Booking systems

5. **Your data fits on a single powerful server**
   - < 1TB of data
   - Vertical scaling is acceptable

**Example Use Cases:**
- E-commerce order management
- **Your school administration system** ✅
- Accounting systems
- HR management systems
- Traditional web applications

### When to Use NoSQL

✅ **Use NoSQL when:**

1. **Massive scale (horizontal scaling needed)**
   - Billions of records
   - Petabytes of data
   - Distributed across regions

2. **Schema flexibility required**
   - Rapidly changing requirements
   - Varied data structures
   - Prototype/MVP development

3. **High write throughput**
   - Logging systems
   - IoT sensor data
   - Real-time analytics

4. **Geographic distribution**
   - Multi-region deployment
   - Low latency globally
   - Eventual consistency acceptable

5. **Specific data patterns**
   - Key-value lookups (Redis)
   - Document storage (MongoDB)
   - Time-series data (InfluxDB)
   - Graph relationships (Neo4j)

**Example Use Cases:**
- Social media feeds (Cassandra)
- Real-time analytics (InfluxDB)
- Session storage (Redis)
- Product catalogs (MongoDB)
- Recommendation engines (Neo4j)

---

## Relational Databases (SQL)

### Popular SQL Databases Comparison

#### 1. MySQL

```
Strengths:
✓ Most popular open-source database
✓ Great for web applications
✓ Fast read operations
✓ Easy to learn
✓ Large community

Weaknesses:
✗ Limited advanced features
✗ Less robust for complex queries
✗ MyISAM engine doesn't support transactions

Best For:
- Web applications
- LAMP stack
- E-commerce (small to medium)
- Content management systems

Used By: Facebook, Twitter, YouTube
```

**Your School Admin System uses MySQL!**

#### 2. PostgreSQL

```
Strengths:
✓ Most advanced open-source database
✓ Full ACID compliance
✓ Rich data types (JSON, arrays, hstore)
✓ Advanced features (CTEs, window functions)
✓ Better for complex queries
✓ Strong data integrity

Weaknesses:
✗ Slower for simple read operations
✗ More complex to configure
✗ Smaller community than MySQL

Best For:
- Complex applications
- Data integrity critical
- Advanced analytics
- Geospatial data (PostGIS)

Used By: Instagram, Spotify, Reddit
```

#### 3. SQL Server (Microsoft)

```
Strengths:
✓ Excellent Windows integration
✓ Great tooling (SSMS)
✓ Enterprise features
✓ Business intelligence built-in

Weaknesses:
✗ Expensive licensing
✗ Windows-centric
✗ Less portable

Best For:
- Enterprise applications
- .NET stack
- Windows environments

Used By: Large enterprises
```

#### 4. Oracle Database

```
Strengths:
✓ Most feature-rich
✓ Enterprise-grade
✓ Excellent performance at scale
✓ Advanced security

Weaknesses:
✗ Very expensive
✗ Complex
✗ Vendor lock-in

Best For:
- Large enterprises
- Mission-critical systems
- Heavy transactional workloads

Used By: Banks, government, large corporations
```

### SQL Database Schema Example

**Your School Admin System Schema:**

```sql
-- Well-normalized relational design

CREATE TABLE students (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_email (email),
    INDEX idx_name (name)
);

CREATE TABLE classes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    class_code VARCHAR(50) UNIQUE NOT NULL,
    class_name VARCHAR(255) NOT NULL,
    INDEX idx_class_code (class_code)
);

CREATE TABLE subjects (
    id INT PRIMARY KEY AUTO_INCREMENT,
    subject_code VARCHAR(50) UNIQUE NOT NULL,
    subject_name VARCHAR(255) NOT NULL,
    INDEX idx_subject_code (subject_code)
);

CREATE TABLE teachers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    INDEX idx_email (email)
);

-- Junction tables for many-to-many relationships

CREATE TABLE class_students (
    id INT PRIMARY KEY AUTO_INCREMENT,
    class_id INT NOT NULL,
    student_id INT NOT NULL,
    enrollment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (class_id) REFERENCES classes(id) ON DELETE CASCADE,
    FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
    UNIQUE KEY unique_enrollment (class_id, student_id),
    INDEX idx_class_id (class_id),
    INDEX idx_student_id (student_id)
);

CREATE TABLE teacher_class_subjects (
    id INT PRIMARY KEY AUTO_INCREMENT,
    teacher_id INT NOT NULL,
    class_id INT NOT NULL,
    subject_id INT NOT NULL,
    FOREIGN KEY (teacher_id) REFERENCES teachers(id) ON DELETE CASCADE,
    FOREIGN KEY (class_id) REFERENCES classes(id) ON DELETE CASCADE,
    FOREIGN KEY (subject_id) REFERENCES subjects(id) ON DELETE CASCADE,
    UNIQUE KEY unique_assignment (teacher_id, class_id, subject_id),
    INDEX idx_teacher_id (teacher_id),
    INDEX idx_class_subject (class_id, subject_id)
);
```

**Why SQL is perfect for this system:**
- ✅ Clear relationships (students ↔ classes ↔ teachers)
- ✅ Data integrity (foreign keys prevent orphaned records)
- ✅ Complex queries (find all students in a teacher's classes)
- ✅ ACID transactions (enroll student in multiple classes atomically)
- ✅ Moderate scale (thousands of students, not billions)

---

## NoSQL Database Types

### 1. Document Store (MongoDB, CouchDB)

**Data Model:** JSON-like documents

```javascript
// MongoDB Example: Product Catalog

{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "name": "Laptop",
  "price": 999.99,
  "category": "Electronics",
  "specs": {
    "cpu": "Intel i7",
    "ram": "16GB",
    "storage": "512GB SSD"
  },
  "reviews": [
    {
      "user": "john_doe",
      "rating": 5,
      "comment": "Great laptop!",
      "date": ISODate("2024-01-15")
    },
    {
      "user": "jane_smith",
      "rating": 4,
      "comment": "Good value",
      "date": ISODate("2024-01-20")
    }
  ],
  "tags": ["laptop", "portable", "business"],
  "inStock": true,
  "lastModified": ISODate("2024-01-20")
}
```

**Characteristics:**
- ✅ Flexible schema (each document can have different fields)
- ✅ Embedded documents (reviews inside product)
- ✅ Easy to scale horizontally
- ✅ Fast reads/writes for single documents
- ❌ Limited join support
- ❌ Data duplication (denormalized)

**When to Use:**
- Product catalogs (varied attributes)
- Content management (blog posts, articles)
- User profiles (different user types)
- Mobile applications (offline sync)

**Querying:**

```javascript
// Find laptops over $500 with good reviews
db.products.find({
  category: "Electronics",
  price: { $gte: 500 },
  "reviews.rating": { $gte: 4 }
})

// Update embedded document
db.products.updateOne(
  { _id: ObjectId("...") },
  { $push: { reviews: newReview } }
)
```

**Comparison with SQL:**

```
SQL Approach (Normalized):
┌──────────┐    ┌─────────┐
│ Products │───<│ Reviews │
└──────────┘    └─────────┘
3 tables, JOIN required

MongoDB Approach (Denormalized):
┌────────────────────┐
│ Product Document   │
│  - Product fields  │
│  - Reviews[]       │
└────────────────────┘
1 collection, no JOIN
```

### 2. Key-Value Store (Redis, DynamoDB, Memcached)

**Data Model:** Simple key → value pairs

```javascript
// Redis Examples

// String value
SET user:1000:name "John Doe"
GET user:1000:name
→ "John Doe"

// Hash (object)
HSET user:1000 name "John Doe" email "john@example.com" age 30
HGETALL user:1000
→ { name: "John Doe", email: "john@example.com", age: 30 }

// List (ordered)
LPUSH recent_searches:user:1000 "laptop" "phone" "tablet"
LRANGE recent_searches:user:1000 0 4
→ ["laptop", "phone", "tablet"]

// Set (unique values)
SADD tags:product:500 "electronics" "portable" "business"
SMEMBERS tags:product:500
→ ["electronics", "portable", "business"]

// Sorted Set (with scores)
ZADD leaderboard 100 "player1" 200 "player2" 150 "player3"
ZREVRANGE leaderboard 0 2 WITHSCORES
→ ["player2", 200, "player3", 150, "player1", 100]

// TTL (automatic expiration)
SETEX session:abc123 3600 "user_data"
// Expires in 1 hour
```

**Characteristics:**
- ✅ Extremely fast (in-memory)
- ✅ Simple operations
- ✅ Atomic operations
- ✅ Built-in TTL (expiration)
- ❌ No complex queries
- ❌ Limited by memory (Redis)

**When to Use:**
- **Caching** (most common use case)
- Session storage
- Real-time leaderboards
- Rate limiting
- Pub/Sub messaging
- Temporary data

**Use Cases:**

```typescript
// 1. Caching database queries
async function getStudent(id: number) {
  const cacheKey = `student:${id}`;

  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // Cache miss - query database
  const student = await db.Student.findByPk(id);

  // Store in cache (1 hour)
  await redis.setex(cacheKey, 3600, JSON.stringify(student));

  return student;
}

// 2. Rate limiting
async function checkRateLimit(userId: string): Promise<boolean> {
  const key = `rate_limit:${userId}`;
  const current = await redis.incr(key);

  if (current === 1) {
    // First request, set expiry
    await redis.expire(key, 60); // 60 seconds
  }

  return current <= 100; // Max 100 requests per minute
}

// 3. Session storage
async function storeSession(sessionId: string, userData: object) {
  await redis.setex(
    `session:${sessionId}`,
    86400, // 24 hours
    JSON.stringify(userData)
  );
}

// 4. Real-time leaderboard
async function updateScore(playerId: string, score: number) {
  await redis.zadd('game:leaderboard', score, playerId);
}

async function getTopPlayers(count: number) {
  return await redis.zrevrange('game:leaderboard', 0, count - 1, 'WITHSCORES');
}
```

### 3. Column-Family Store (Cassandra, HBase)

**Data Model:** Wide column store

```
Row Key: user_123
┌───────────────┬────────────────┬─────────────────┬──────────────┐
│ Column Family │   personal     │   activity      │   settings   │
├───────────────┼────────────────┼─────────────────┼──────────────┤
│ Columns       │ name: "John"   │ last_login: ... │ theme: "dark"│
│               │ email: "..."   │ login_count: 50 │ lang: "en"   │
│               │ age: 30        │ last_post: ...  │ tz: "PST"    │
└───────────────┴────────────────┴─────────────────┴──────────────┘
```

**Real Example: Cassandra for Time-Series Data**

```cql
-- Schema for sensor data (IoT)
CREATE TABLE sensor_data (
    sensor_id UUID,
    year INT,
    month INT,
    timestamp TIMESTAMP,
    temperature DECIMAL,
    humidity DECIMAL,
    PRIMARY KEY ((sensor_id, year, month), timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);

-- Data is partitioned by sensor_id, year, month
-- Within partition, sorted by timestamp

-- Efficient query (within partition)
SELECT * FROM sensor_data
WHERE sensor_id = ?
  AND year = 2024
  AND month = 1
  AND timestamp > '2024-01-01'
  AND timestamp < '2024-01-31';
```

**Characteristics:**
- ✅ Extremely scalable (petabytes)
- ✅ Fast writes (append-only)
- ✅ No single point of failure
- ✅ Tunable consistency
- ❌ No joins
- ❌ Limited query flexibility
- ❌ Denormalized data

**When to Use:**
- Time-series data (IoT sensors, logs)
- Write-heavy workloads
- Massive scale (petabytes)
- Geographic distribution
- High availability requirements

**Example: Instagram's Cassandra Usage**

```
Store user photos metadata:
Partition Key: user_id
Clustering Key: photo_timestamp

Quick queries:
- Get all photos for user (single partition read)
- Get recent photos (sorted by timestamp)
- Distribute users across cluster (partition key)
```

### 4. Graph Database (Neo4j, Amazon Neptune)

**Data Model:** Nodes and edges (relationships)

```
Social Network Example:

(Alice)-[:FRIENDS_WITH]->(Bob)
   │
   └─[:WORKS_AT]─>(Company A)
   │
   └─[:LIKES]─>(Post 1)
        │
        └─[:TAGGED_IN]─>(Bob)

(Bob)-[:FRIENDS_WITH]->(Charlie)
  │
  └─[:WORKS_AT]─>(Company B)
  │
  └─[:LIKES]─>(Post 1)
```

**Cypher Query (Neo4j):**

```cypher
// Find friends of friends who work at the same company
MATCH (me:Person {name: "Alice"})-[:FRIENDS_WITH]-(friend)-[:FRIENDS_WITH]-(foaf)
WHERE (me)-[:WORKS_AT]->(:Company)<-[:WORKS_AT]-(foaf)
  AND NOT (me)-[:FRIENDS_WITH]-(foaf)
RETURN foaf.name, COUNT(*) as mutual_friends
ORDER BY mutual_friends DESC
LIMIT 10;

// Recommendation: "People you may know"
```

**Comparison with SQL:**

```sql
-- SQL equivalent (much slower, complex joins)
SELECT foaf.name, COUNT(*) as mutual_friends
FROM persons me
JOIN friendships f1 ON me.id = f1.person1_id
JOIN persons friend ON f1.person2_id = friend.id
JOIN friendships f2 ON friend.id = f2.person1_id
JOIN persons foaf ON f2.person2_id = foaf.id
JOIN employments e1 ON me.id = e1.person_id
JOIN employments e2 ON foaf.id = e2.person_id
WHERE me.name = 'Alice'
  AND e1.company_id = e2.company_id
  AND NOT EXISTS (
    SELECT 1 FROM friendships f3
    WHERE f3.person1_id = me.id AND f3.person2_id = foaf.id
  )
GROUP BY foaf.name
ORDER BY mutual_friends DESC
LIMIT 10;
```

**Characteristics:**
- ✅ Fast relationship queries (no joins needed)
- ✅ Natural modeling of connected data
- ✅ Path finding algorithms built-in
- ❌ Limited scalability vs document stores
- ❌ Learning curve (Cypher query language)

**When to Use:**
- Social networks (friends, followers)
- Recommendation engines
- Fraud detection (transaction patterns)
- Knowledge graphs
- Network and IT operations

**Use Cases:**

```
LinkedIn: "People you may know"
- 2nd degree connections
- Mutual connections
- Similar job titles

Amazon: "Customers who bought this also bought"
- Product relationships
- Purchase patterns

Fraud Detection:
- Money flow patterns
- Suspicious transaction chains
- Connected accounts
```

### 5. Time-Series Database (InfluxDB, TimescaleDB)

**Data Model:** Optimized for time-stamped data

```
Measurement: cpu_usage
┌────────────────────┬──────┬──────────┬───────┐
│ time               │ host │ region   │ value │
├────────────────────┼──────┼──────────┼───────┤
│ 2024-01-20 10:00:00│ srv1 │ us-east  │ 45.2  │
│ 2024-01-20 10:00:01│ srv1 │ us-east  │ 47.1  │
│ 2024-01-20 10:00:02│ srv1 │ us-east  │ 43.8  │
│ 2024-01-20 10:00:00│ srv2 │ us-west  │ 62.3  │
└────────────────────┴──────┴──────────┴───────┘
```

**InfluxDB Query (Flux):**

```flux
// Average CPU usage per host in last hour
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu_usage")
  |> group(columns: ["host"])
  |> aggregateWindow(every: 5m, fn: mean)
  |> yield(name: "average_cpu")

// Detect spikes (> 90% usage)
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu_usage" and r._value > 90)
  |> group(columns: ["host"])
  |> count()
```

**Characteristics:**
- ✅ Optimized for time-series data
- ✅ Fast writes (millions per second)
- ✅ Automatic downsampling
- ✅ Built-in analytics functions
- ✅ Data retention policies
- ❌ Not for general-purpose use

**When to Use:**
- **Monitoring & metrics** (server metrics, application performance)
- IoT sensor data
- Financial market data
- Real-time analytics
- DevOps observability

**Example: Application Monitoring**

```typescript
// Write metrics to InfluxDB
import { InfluxDB, Point } from '@influxdata/influxdb-client';

const influx = new InfluxDB({ url, token });
const writeApi = influx.getWriteApi(org, bucket);

// Record API response time
function recordResponseTime(endpoint: string, duration: number) {
  const point = new Point('api_response_time')
    .tag('endpoint', endpoint)
    .tag('method', 'GET')
    .floatField('duration_ms', duration)
    .timestamp(new Date());

  writeApi.writePoint(point);
}

// Query average response times
const queryApi = influx.getQueryApi(org);
const query = `
  from(bucket: "metrics")
    |> range(start: -1h)
    |> filter(fn: (r) => r._measurement == "api_response_time")
    |> group(columns: ["endpoint"])
    |> aggregateWindow(every: 5m, fn: mean)
`;

const results = await queryApi.collectRows(query);
```

### 6. Search Engine (Elasticsearch, Solr)

**Data Model:** Inverted index for full-text search

```json
// Document in Elasticsearch
{
  "id": 123,
  "title": "Introduction to System Design",
  "content": "System design interviews require understanding of distributed systems...",
  "author": "John Doe",
  "tags": ["system-design", "interviews", "distributed-systems"],
  "published_date": "2024-01-20",
  "views": 1500
}
```

**Elasticsearch Query:**

```json
// Full-text search with filters
POST /articles/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "distributed systems",
            "fields": ["title^2", "content"]
          }
        }
      ],
      "filter": [
        { "range": { "published_date": { "gte": "2024-01-01" } } },
        { "term": { "tags": "system-design" } }
      ]
    }
  },
  "sort": [
    { "_score": "desc" },
    { "views": "desc" }
  ],
  "size": 10
}
```

**Characteristics:**
- ✅ Powerful full-text search
- ✅ Fuzzy matching, typo tolerance
- ✅ Aggregations and analytics
- ✅ Real-time indexing
- ✅ Relevance scoring
- ❌ Not ACID compliant
- ❌ Not a primary data store

**When to Use:**
- Search functionality (products, articles)
- Log analysis (ELK stack)
- Real-time analytics
- Autocomplete / typeahead

**Example: E-commerce Product Search**

```typescript
// Search products with autocomplete
async function searchProducts(query: string) {
  const result = await elasticClient.search({
    index: 'products',
    body: {
      query: {
        bool: {
          should: [
            // Exact match (highest priority)
            { match_phrase: { name: { query, boost: 3 } } },
            // Fuzzy match (handle typos)
            { match: { name: { query, fuzziness: 'AUTO' } } },
            // Description match
            { match: { description: { query, boost: 0.5 } } }
          ]
        }
      },
      // Autocomplete suggestions
      suggest: {
        product_suggest: {
          prefix: query,
          completion: { field: 'name_suggest' }
        }
      }
    }
  });

  return result.hits.hits;
}
```

---

## Database Selection Decision Tree

```
START: What is your primary use case?

├─ Structured data with relationships?
│  └─ YES → Need ACID transactions?
│     ├─ YES → Complex queries/reporting?
│     │  ├─ YES → PostgreSQL (advanced features)
│     │  └─ NO  → MySQL (simple, fast)
│     └─ NO  → Can you scale vertically?
│        ├─ YES → MySQL/PostgreSQL
│        └─ NO  → Cassandra (eventually consistent)
│
├─ Flexible schema / rapid changes?
│  └─ YES → MongoDB (document store)
│
├─ Simple key-value lookups?
│  └─ YES → Need persistence?
│     ├─ YES → DynamoDB
│     └─ NO  → Redis (cache)
│
├─ Graph relationships / social network?
│  └─ YES → Neo4j
│
├─ Time-series / metrics?
│  └─ YES → InfluxDB / TimescaleDB
│
├─ Full-text search?
│  └─ YES → Elasticsearch
│
└─ Massive scale (petabytes)?
   └─ YES → Cassandra / HBase
```

### Real-World Examples

```
Netflix:
- Cassandra: Viewing history, user preferences (scalability)
- MySQL: Billing, subscriptions (ACID)
- Elasticsearch: Search titles
- Redis: Session management, caching

Uber:
- PostgreSQL: Transactional data (trips, payments)
- Redis: Real-time locations, caching
- Cassandra: Trip history (massive scale)
- Elasticsearch: Search drivers, locations

Facebook:
- MySQL: Social graph (customized)
- Cassandra: Inbox, messages
- Redis: Caching
- Graph database: Friend recommendations

Your School Admin System:
- MySQL: Perfect choice! ✅
  - Structured data (students, classes, teachers)
  - Relationships (many-to-many)
  - ACID transactions (enrollment)
  - Moderate scale (thousands, not billions)
  - Complex queries (reports)
```

---

## Database Indexing Strategies

### What is an Index?

**Index = Book's Index**

```
Without Index (Full Table Scan):
SELECT * FROM students WHERE email = 'john@school.com';
→ Scan all 10,000 rows (slow)

With Index on email:
→ B-Tree lookup: log₂(10,000) ≈ 14 comparisons (fast)
```

### Index Types

#### 1. Primary Key Index (Clustered Index)

```sql
CREATE TABLE students (
    id INT PRIMARY KEY AUTO_INCREMENT,  -- Automatically indexed
    name VARCHAR(255),
    email VARCHAR(255)
);

-- Data is physically sorted by id
-- Fastest lookups by id
```

#### 2. Unique Index

```sql
-- Enforce uniqueness + fast lookups
CREATE UNIQUE INDEX idx_email ON students(email);

-- Prevents duplicates
INSERT INTO students (email) VALUES ('john@school.com');  -- OK
INSERT INTO students (email) VALUES ('john@school.com');  -- ❌ Error
```

#### 3. Composite Index (Multi-Column)

```sql
-- Index on multiple columns
CREATE INDEX idx_name_grade ON students(name, grade);

-- Efficient queries:
SELECT * FROM students WHERE name = 'John' AND grade = 10;  -- ✅ Uses index
SELECT * FROM students WHERE name = 'John';                 -- ✅ Uses index (leftmost prefix)
SELECT * FROM students WHERE grade = 10;                    -- ❌ Doesn't use index

-- Leftmost Prefix Rule:
-- Index (A, B, C) can be used for:
-- ✅ WHERE A = ?
-- ✅ WHERE A = ? AND B = ?
-- ✅ WHERE A = ? AND B = ? AND C = ?
-- ❌ WHERE B = ?
-- ❌ WHERE C = ?
```

#### 4. Covering Index

```sql
-- Index includes all columns needed by query
CREATE INDEX idx_student_details ON students(id, name, email);

-- Query only uses index (no table lookup)
SELECT id, name, email FROM students WHERE id = 123;
-- ✅ Index-only scan (faster)
```

#### 5. Partial Index (PostgreSQL)

```sql
-- Index only active students
CREATE INDEX idx_active_students ON students(email) WHERE active = true;

-- Smaller index, faster queries for active students
SELECT * FROM students WHERE email = 'john@school.com' AND active = true;
```

#### 6. Full-Text Index

```sql
-- MySQL full-text search
CREATE FULLTEXT INDEX idx_article_content ON articles(title, body);

-- Search query
SELECT * FROM articles
WHERE MATCH(title, body) AGAINST('system design' IN NATURAL LANGUAGE MODE);
```

### Indexing Best Practices

#### ✅ DO Index:

1. **Primary Keys** (automatic)
2. **Foreign Keys** (for joins)
3. **Frequently queried columns** (WHERE, JOIN, ORDER BY)
4. **Unique constraints** (prevent duplicates)

```sql
-- Your school admin system
CREATE INDEX idx_student_email ON students(email);          -- Frequent lookups
CREATE INDEX idx_class_code ON classes(class_code);         -- Unique identifier
CREATE INDEX idx_class_student_class ON class_students(class_id);  -- Foreign key
CREATE INDEX idx_class_student_student ON class_students(student_id); -- Foreign key
```

#### ❌ DON'T Index:

1. **Small tables** (< 1000 rows - full scan is fast enough)
2. **Columns with low cardinality** (few distinct values)
   ```sql
   -- Bad: Only 2 values (male/female)
   CREATE INDEX idx_gender ON students(gender);  -- ❌
   ```
3. **Columns rarely queried**
4. **Tables with heavy writes** (indexes slow down INSERT/UPDATE)

### Index Trade-offs

```
Pros:
✅ Faster SELECT queries (10-100x speedup)
✅ Faster JOIN operations
✅ Faster sorting (ORDER BY)
✅ Enforce uniqueness

Cons:
❌ Slower INSERT/UPDATE/DELETE (must update index)
❌ Additional storage space (indexes can be larger than table)
❌ Too many indexes can confuse query optimizer
```

### Index Performance Example

```sql
-- Table: 1 million students

-- Without index
SELECT * FROM students WHERE email = 'john@school.com';
-- 🐌 Full table scan: 1,000,000 rows examined
-- Time: ~500ms

-- With index on email
CREATE INDEX idx_email ON students(email);
SELECT * FROM students WHERE email = 'john@school.com';
-- 🚀 Index lookup: ~10 rows examined
-- Time: ~5ms

-- 100x faster!
```

### Analyzing Index Usage

```sql
-- Check if index is used (MySQL)
EXPLAIN SELECT * FROM students WHERE email = 'john@school.com';

-- Output:
-- type: ref (index used) ✅
-- key: idx_email
-- rows: 1

-- No index used:
-- type: ALL (full table scan) ❌
-- rows: 1000000
```

---

## Database Normalization

### What is Normalization?

**Normalization:** Organizing data to reduce redundancy and improve integrity

### Normal Forms

#### Unnormalized Data (Bad)

```
Order Table:
┌────────┬──────────────┬────────────────────────────────────┐
│OrderID │ CustomerInfo │ Products                           │
├────────┼──────────────┼────────────────────────────────────┤
│ 1001   │ John, NYC    │ Laptop:$999, Mouse:$25, Keyboard:$50│
│ 1002   │ Jane, LA     │ Phone:$699, Case:$15               │
└────────┴──────────────┴────────────────────────────────────┘

Problems:
❌ Data duplication (customer info repeated)
❌ Update anomaly (change customer city in multiple places)
❌ Deletion anomaly (delete order → lose customer info)
❌ Hard to query (products in comma-separated string)
```

#### 1st Normal Form (1NF): Atomic Values

```
Orders:
┌────────┬──────────────┬─────────────┬──────────────┐
│OrderID │ CustomerName │ City        │ ProductName  │ Price │
├────────┼──────────────┼─────────────┼──────────────┤───────┤
│ 1001   │ John         │ NYC         │ Laptop       │ 999   │
│ 1001   │ John         │ NYC         │ Mouse        │ 25    │
│ 1001   │ John         │ NYC         │ Keyboard     │ 50    │
│ 1002   │ Jane         │ LA          │ Phone        │ 699   │
│ 1002   │ Jane         │ LA          │ Case         │ 15    │
└────────┴──────────────┴─────────────┴──────────────┴───────┘

✅ No repeating groups
✅ Each cell contains single value
❌ Still has redundancy (customer info repeated)
```

#### 2nd Normal Form (2NF): No Partial Dependencies

```
Customers:                    Orders:
┌────────────┬──────┐         ┌────────┬────────────┐
│ CustomerID │ City │         │OrderID │ CustomerID │
├────────────┼──────┤         ├────────┼────────────┤
│ C001       │ NYC  │         │ 1001   │ C001       │
│ C002       │ LA   │         │ 1002   │ C002       │
└────────────┴──────┘         └────────┴────────────┘

Order_Items:
┌────────┬────────────┬──────────────┬───────┐
│OrderID │ ProductID  │ ProductName  │ Price │
├────────┼────────────┼──────────────┼───────┤
│ 1001   │ P001       │ Laptop       │ 999   │
│ 1001   │ P002       │ Mouse        │ 25    │
│ 1001   │ P003       │ Keyboard     │ 50    │
└────────┴────────────┴──────────────┴───────┘

✅ Non-key attributes depend on entire primary key
❌ ProductName/Price depend on ProductID, not OrderID
```

#### 3rd Normal Form (3NF): No Transitive Dependencies

```
Customers:                    Orders:
┌────────────┬──────────┐     ┌────────┬────────────┬────────────┐
│ CustomerID │ CityID   │     │OrderID │ CustomerID │ OrderDate  │
├────────────┼──────────┤     ├────────┼────────────┼────────────┤
│ C001       │ CT001    │     │ 1001   │ C001       │ 2024-01-20 │
│ C002       │ CT002    │     │ 1002   │ C002       │ 2024-01-21 │
└────────────┴──────────┘     └────────┴────────────┴────────────┘

Cities:                       Products:
┌────────┬──────────┐         ┌───────────┬──────────────┬───────┐
│ CityID │ CityName │         │ ProductID │ ProductName  │ Price │
├────────┼──────────┤         ├───────────┼──────────────┼───────┤
│ CT001  │ NYC      │         │ P001      │ Laptop       │ 999   │
│ CT002  │ LA       │         │ P002      │ Mouse        │ 25    │
└────────┴──────────┘         └───────────┴──────────────┴───────┘

Order_Items:
┌────────┬───────────┬──────────┐
│OrderID │ ProductID │ Quantity │
├────────┼───────────┼──────────┤
│ 1001   │ P001      │ 1        │
│ 1001   │ P002      │ 2        │
└────────┴───────────┴──────────┘

✅ All attributes depend only on primary key
✅ No redundancy
✅ Data integrity
```

### Denormalization (When to Break the Rules)

**Sometimes denormalization improves performance:**

```sql
-- Normalized (3NF)
SELECT
    o.order_id,
    c.customer_name,
    c.city,
    p.product_name,
    oi.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id;
-- 4 table joins (slow for millions of rows)

-- Denormalized (add redundant columns)
SELECT
    order_id,
    customer_name,  -- ← Redundant (also in customers table)
    city,           -- ← Redundant
    product_name,   -- ← Redundant
    quantity
FROM order_details;
-- No joins (fast, but data duplication)
```

**When to Denormalize:**
- ✅ Read-heavy workloads (analytics, reporting)
- ✅ Performance critical (avoid complex joins)
- ✅ Data rarely changes (no update anomalies)

**Example:** Your school system could denormalize student name in class_students for faster queries:

```sql
-- Normalized (current)
SELECT s.name, c.class_name
FROM class_students cs
JOIN students s ON cs.student_id = s.id
JOIN classes c ON cs.class_id = c.id;

-- Denormalized (faster but redundant)
SELECT student_name, class_name
FROM class_students_denormalized;
```

---

## Query Optimization

### Slow Query Example

```sql
-- ❌ Slow query (1000ms)
SELECT *
FROM students s
JOIN class_students cs ON s.id = cs.student_id
JOIN classes c ON cs.class_id = c.id
WHERE s.name LIKE '%John%'
  AND c.class_code = 'MATH-101'
ORDER BY s.created_at DESC
LIMIT 10;
```

### Optimization Techniques

#### 1. Add Indexes

```sql
-- ✅ Index foreign keys
CREATE INDEX idx_class_student_student ON class_students(student_id);
CREATE INDEX idx_class_student_class ON class_students(class_id);

-- ✅ Index WHERE columns
CREATE INDEX idx_class_code ON classes(class_code);
CREATE INDEX idx_student_name ON students(name);

-- ✅ Index ORDER BY columns
CREATE INDEX idx_student_created ON students(created_at DESC);

-- Now query is 100x faster (10ms)
```

#### 2. Avoid SELECT *

```sql
-- ❌ Fetches all columns (wasteful)
SELECT * FROM students;

-- ✅ Select only needed columns
SELECT id, name, email FROM students;
```

#### 3. Avoid LIKE with Leading Wildcard

```sql
-- ❌ Can't use index
SELECT * FROM students WHERE name LIKE '%John%';

-- ✅ Can use index
SELECT * FROM students WHERE name LIKE 'John%';

-- ✅ Use full-text search for partial matches
SELECT * FROM students WHERE MATCH(name) AGAINST('John');
```

#### 4. Use LIMIT

```sql
-- ❌ Returns all rows
SELECT * FROM students;

-- ✅ Limit results
SELECT * FROM students LIMIT 10;
```

#### 5. Optimize Joins

```sql
-- ❌ Join on unindexed column
SELECT * FROM students s
JOIN classes c ON s.grade = c.grade_level;

-- ✅ Join on indexed foreign keys
SELECT * FROM students s
JOIN class_students cs ON s.id = cs.student_id;
```

#### 6. Use EXPLAIN

```sql
EXPLAIN SELECT * FROM students WHERE email = 'john@school.com';

-- Check:
-- ✅ type: const/ref (uses index)
-- ❌ type: ALL (full table scan)
-- ✅ rows: 1 (few rows examined)
-- ❌ rows: 1000000 (many rows examined)
```

#### 7. Batch Operations

```sql
-- ❌ Slow (N queries)
for (let student of students) {
  await db.query('INSERT INTO students VALUES (?)', [student]);
}

-- ✅ Fast (1 query)
await db.query('INSERT INTO students VALUES ?', [students]);
```

---

## ACID vs BASE

### ACID (SQL Databases)

**Atomicity:** All or nothing

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Deduct $100
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Add $100

-- If any fails, both rollback
COMMIT;  -- Or ROLLBACK
```

**Consistency:** Data integrity maintained

```sql
-- Constraint: balance >= 0
UPDATE accounts SET balance = -50 WHERE id = 1;  -- ❌ Violates constraint, rejected
```

**Isolation:** Concurrent transactions don't interfere

```
Transaction A: Read balance = $100
Transaction B: Read balance = $100
Transaction A: Write balance = $50  (deduct $50)
Transaction B: Write balance = $75  (deduct $25)
Result: $75 (wrong! Should be $50)

✅ With Isolation: Transactions execute serially
```

**Durability:** Committed data survives crashes

```
COMMIT;  -- Data written to disk, survives power failure
```

### BASE (NoSQL Databases)

**Basically Available:** System always responds (may be stale)

**Soft state:** State may change over time (eventual consistency)

**Eventual consistency:** Will become consistent eventually

```javascript
// DynamoDB example
await dynamodb.put({ userId: 123, name: "John Updated" });
// ✅ Write successful

// Read from replica immediately
const user = await dynamodb.get({ userId: 123 });
// ❌ May still return "John" (old value)

// Read again after 100ms
setTimeout(async () => {
  const user = await dynamodb.get({ userId: 123 });
  // ✅ Returns "John Updated" (replicas caught up)
}, 100);
```

**ACID vs BASE Comparison:**

```
ACID (SQL):                    BASE (NoSQL):
✅ Strong consistency          ✅ High availability
✅ Data integrity              ✅ Partition tolerance
✅ Simple reasoning            ✅ Better performance
❌ Lower availability          ❌ Eventual consistency
❌ Harder to scale             ❌ Complex conflict resolution

Use ACID for:                  Use BASE for:
- Banking                      - Social media feeds
- E-commerce orders            - Caching
- Inventory                    - Logging
- Anything financial           - Analytics
```

---

## Transactions & Isolation Levels

### What is a Transaction?

```sql
-- Transfer $100 from Account A to Account B
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE id = 'B';

COMMIT;  -- Both succeed or both fail (atomicity)
```

### Isolation Levels (Weakest → Strongest)

#### 1. Read Uncommitted (Lowest)

```
Transaction A:              Transaction B:
BEGIN;
UPDATE accounts
SET balance = 100
WHERE id = 1;
                           BEGIN;
                           SELECT balance FROM accounts WHERE id = 1;
                           -- Reads: 100 (uncommitted!)
ROLLBACK;                  -- Dirty Read!
                           -- Transaction A rolled back, but B read uncommitted value
```

**Problems:** Dirty reads
**Use:** Never (data integrity issues)

#### 2. Read Committed (Default in most DBs)

```
Transaction A:              Transaction B:
BEGIN;
UPDATE accounts
SET balance = 100
WHERE id = 1;
                           BEGIN;
                           SELECT balance FROM accounts WHERE id = 1;
                           -- Reads: 50 (original, committed value)
COMMIT;
                           SELECT balance FROM accounts WHERE id = 1;
                           -- Reads: 100 (new committed value)
                           -- Non-Repeatable Read! (value changed between reads)
```

**Problems:** Non-repeatable reads
**Use:** Most applications (good balance)

#### 3. Repeatable Read

```
Transaction A:              Transaction B:
BEGIN;
SELECT balance
FROM accounts
WHERE id = 1;
-- Reads: 50
                           BEGIN;
                           UPDATE accounts SET balance = 100 WHERE id = 1;
                           COMMIT;
SELECT balance
FROM accounts
WHERE id = 1;
-- Still reads: 50 (repeatable read, sees snapshot)
COMMIT;
```

**Problems:** Phantom reads (new rows may appear)
**Use:** Reports, analytics

#### 4. Serializable (Highest)

```
Transactions execute as if serial (one after another)
No dirty reads, no non-repeatable reads, no phantom reads
```

**Trade-off:** Lowest concurrency, slowest
**Use:** Financial transactions, critical operations

### Setting Isolation Level

```sql
-- MySQL
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- PostgreSQL (default is READ COMMITTED)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

## Common Database Interview Questions

### Q1: How do you choose between SQL and NoSQL?

**Answer:**

"I would consider several factors:

**Choose SQL if:**
1. Data has clear relationships (students, classes, teachers)
2. ACID transactions are critical (financial data)
3. Complex queries with joins are needed
4. Schema is well-defined and stable
5. Data integrity is paramount

**Choose NoSQL if:**
1. Need massive horizontal scaling (billions of records)
2. Schema is flexible or rapidly changing
3. Specific access patterns (key-value, document, graph)
4. Eventual consistency is acceptable
5. High write throughput required

In my school admin project, I chose MySQL because:
- Clear relationships (many-to-many between students and classes)
- Need ACID (enrolling student in multiple classes atomically)
- Moderate scale (thousands, not billions)
- Complex queries for reports (which students in which classes)
- Data integrity is critical (no orphaned records)

For a different use case like user session storage, I'd use Redis (key-value NoSQL) for fast access and automatic expiration."

### Q2: Explain the N+1 query problem and how to solve it

**Answer:**

"N+1 problem occurs when you fetch a list of N items, then make N additional queries to fetch related data.

**Example:**

```typescript
// ❌ N+1 Problem (1 + N queries)
const classes = await Class.findAll();  // 1 query (fetches 10 classes)

for (const cls of classes) {
  const students = await cls.getStudents();  // N queries (10 more queries!)
  console.log(cls.name, students.length);
}
// Total: 11 queries
```

**Solution 1: Eager Loading**

```typescript
// ✅ Eager load with JOIN (1 query)
const classes = await Class.findAll({
  include: [{ model: Student }]  // JOIN in single query
});

for (const cls of classes) {
  console.log(cls.name, cls.students.length);
}
// Total: 1 query
```

**Solution 2: Batch Loading (DataLoader)**

```typescript
const DataLoader = require('dataloader');

const studentLoader = new DataLoader(async (classIds) => {
  // Fetch all students for all class IDs in one query
  const students = await ClassStudent.findAll({
    where: { classId: classIds }
  });

  // Group by classId
  return classIds.map(id =>
    students.filter(s => s.classId === id)
  );
});

// Usage
for (const cls of classes) {
  const students = await studentLoader.load(cls.id);  // Batched!
}
```

In my project, I use Sequelize's `include` option to avoid N+1 queries when loading related data."

### Q3: How would you design a database schema for a social media application?

**Answer:**

```sql
-- Users table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    bio TEXT,
    profile_image_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
);

-- Posts table
CREATE TABLE posts (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    content TEXT NOT NULL,
    image_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at DESC)
);

-- Followers (self-referencing many-to-many)
CREATE TABLE followers (
    follower_id BIGINT NOT NULL,    -- User doing the following
    followee_id BIGINT NOT NULL,    -- User being followed
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (follower_id, followee_id),
    FOREIGN KEY (follower_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (followee_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_follower (follower_id),
    INDEX idx_followee (followee_id)
);

-- Likes
CREATE TABLE likes (
    user_id BIGINT NOT NULL,
    post_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, post_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    INDEX idx_post (post_id)  -- For counting likes per post
);

-- Comments
CREATE TABLE comments (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    post_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_post_id (post_id),
    INDEX idx_created_at (created_at)
);
```

**Scaling Considerations:**

For large scale (millions of users), I would:

1. **Denormalize for feeds:**
   - Pre-compute user feeds (fanout on write)
   - Store in Redis/Cassandra for fast access

2. **Shard posts by user_id or time:**
   - Distribute data across multiple databases

3. **Use CDN for images:**
   - Store URLs only in database

4. **Cache heavily:**
   - User profiles in Redis
   - Popular posts in cache

5. **Consider NoSQL for specific needs:**
   - Cassandra for timeline/feed (write-heavy)
   - Redis for real-time counters (likes count)
   - Elasticsearch for user search"

---

## Summary: Database Decision Checklist

```
☐ Define Requirements
  ☐ Data structure (structured vs flexible)
  ☐ Relationships (complex vs simple)
  ☐ Consistency needs (strong vs eventual)
  ☐ Scale (thousands vs billions)

☐ Consider Access Patterns
  ☐ Read-heavy or write-heavy?
  ☐ Simple lookups or complex queries?
  ☐ Real-time requirements?

☐ Evaluate Trade-offs
  ☐ ACID vs availability
  ☐ Consistency vs partition tolerance
  ☐ Vertical vs horizontal scaling

☐ Choose Database Type
  ☐ SQL: Structured, ACID, relationships
  ☐ Document: Flexible schema, nested data
  ☐ Key-Value: Simple lookups, caching
  ☐ Column-Family: Time-series, massive scale
  ☐ Graph: Relationships, social networks
  ☐ Search: Full-text search

☐ Design Schema
  ☐ Normalize (reduce redundancy)
  ☐ Add indexes (foreign keys, WHERE columns)
  ☐ Plan for growth (sharding strategy)

☐ Optimize Performance
  ☐ Indexes on query columns
  ☐ Avoid N+1 queries
  ☐ Use caching (Redis)
  ☐ Monitor slow queries
```

**Remember:** There's no perfect database, only the right tool for the job!
