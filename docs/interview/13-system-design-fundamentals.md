# System Design Fundamentals

## Table of Contents
1. [System Design Interview Framework](#system-design-interview-framework)
2. [Requirements Gathering](#requirements-gathering)
3. [Capacity Estimation & Back-of-Envelope Calculations](#capacity-estimation--back-of-envelope-calculations)
4. [High-Level Architecture Patterns](#high-level-architecture-patterns)
5. [Component Design Patterns](#component-design-patterns)
6. [API Design](#api-design)
7. [Data Flow & Communication Patterns](#data-flow--communication-patterns)
8. [Caching Strategies](#caching-strategies)
9. [Load Balancing](#load-balancing)
10. [Microservices vs Monolith](#microservices-vs-monolith)
11. [CAP Theorem](#cap-theorem)
12. [Consistency Patterns](#consistency-patterns)
13. [Replication & Partitioning](#replication--partitioning)
14. [Common Interview Questions](#common-interview-questions)

---

## System Design Interview Framework

### The 7-Step Approach

```
1. Clarify Requirements (5 mins)
   ├── Functional Requirements
   └── Non-Functional Requirements

2. Capacity Estimation (5 mins)
   ├── Traffic estimates
   ├── Storage estimates
   └── Bandwidth estimates

3. High-Level Design (10 mins)
   ├── Draw major components
   ├── Show data flow
   └── Identify bottlenecks

4. Deep Dive (15 mins)
   ├── Database schema
   ├── API design
   └── Critical components

5. Identify Bottlenecks (5 mins)
   ├── Single points of failure
   ├── Scalability issues
   └── Performance bottlenecks

6. Scaling & Optimization (10 mins)
   ├── Horizontal vs Vertical scaling
   ├── Caching strategies
   └── Database optimization

7. Trade-offs & Alternatives (5 mins)
   ├── Discuss alternatives
   ├── Justify decisions
   └── Address edge cases
```

### Interview Tips

**DO:**
- ✅ Ask clarifying questions
- ✅ State assumptions explicitly
- ✅ Think out loud
- ✅ Start simple, then scale
- ✅ Draw diagrams
- ✅ Discuss trade-offs
- ✅ Consider failure scenarios

**DON'T:**
- ❌ Jump into implementation details
- ❌ Assume requirements
- ❌ Design for unlimited scale immediately
- ❌ Ignore interviewer hints
- ❌ Forget about non-functional requirements

---

## Requirements Gathering

### Functional Requirements

**Questions to Ask:**

1. **Users & Scale**
   - How many users? (DAU - Daily Active Users)
   - Expected growth rate?
   - Geographic distribution?

2. **Core Features**
   - What are the main use cases?
   - Read-heavy or write-heavy?
   - Real-time requirements?

3. **Data**
   - What data needs to be stored?
   - How long should data be retained?
   - Any compliance requirements? (GDPR, HIPAA)

### Non-Functional Requirements

**The "ilities":**

1. **Scalability**
   - Handle 1M users → 100M users
   - Horizontal scaling capability

2. **Availability**
   - 99.9% (8.76 hours downtime/year)
   - 99.99% (52.56 minutes downtime/year)
   - 99.999% (5.26 minutes downtime/year)

3. **Reliability**
   - No data loss
   - Consistent behavior

4. **Performance**
   - Latency requirements (< 200ms API response)
   - Throughput (requests per second)

5. **Consistency**
   - Strong consistency vs Eventual consistency
   - ACID vs BASE

6. **Security**
   - Authentication & Authorization
   - Encryption (at rest, in transit)

### Example: URL Shortener Requirements

**Functional:**
- Shorten long URLs to short codes (7 characters)
- Redirect short URLs to original URLs
- Custom aliases (optional)
- Analytics (click count, referrer)
- URL expiration

**Non-Functional:**
- Read-heavy (100:1 read-to-write ratio)
- Low latency (< 100ms redirect)
- High availability (99.99%)
- 100M URLs created per month
- 10B redirects per month

---

## Capacity Estimation & Back-of-Envelope Calculations

### Key Numbers to Remember

```
Latency Comparison Numbers
--------------------------
L1 cache reference                 0.5 ns
L2 cache reference                   7 ns
Main memory reference              100 ns
Read 1 MB sequentially from memory 250 µs
Disk seek                           10 ms
Read 1 MB sequentially from SSD      1 ms
Read 1 MB sequentially from disk     5 ms
Send 1KB over 1 Gbps network        10 µs
Round trip within same datacenter  500 µs
Round trip CA to Netherlands       150 ms

Storage Units
-------------
1 KB = 1,000 bytes = 10^3
1 MB = 1,000 KB = 10^6
1 GB = 1,000 MB = 10^9
1 TB = 1,000 GB = 10^12
1 PB = 1,000 TB = 10^15

Time Units
----------
1 day = 86,400 seconds ≈ 100,000 seconds (10^5)
1 month ≈ 2.5M seconds (2.5 × 10^6)
1 year ≈ 30M seconds (3 × 10^7)

Common Assumptions
------------------
1 million users
100 requests per user per day
= 100M requests/day
= 100M / 100,000 seconds
≈ 1,000 requests/second (1K RPS)

Peak traffic = Average × 3
= 3,000 requests/second (3K RPS)
```

### Estimation Example: Twitter-like System

**Given:**
- 300M monthly active users (MAU)
- 50% daily active (150M DAU)
- Each user posts 2 tweets per day
- Each user views 100 tweets per day

**Traffic Estimates:**

```
Writes (Posts):
150M users × 2 tweets/day = 300M tweets/day
300M / 100,000 seconds = 3,000 tweets/second (average)
Peak: 3,000 × 3 = 9,000 tweets/second

Reads (Views):
150M users × 100 tweets/day = 15B tweet views/day
15B / 100,000 = 150,000 views/second (average)
Peak: 150,000 × 3 = 450,000 views/second

Read-to-Write Ratio: 150,000 / 3,000 = 50:1 (read-heavy)
```

**Storage Estimates:**

```
Per Tweet:
- Tweet ID: 8 bytes
- User ID: 8 bytes
- Text: 140 characters × 2 bytes = 280 bytes
- Timestamp: 8 bytes
- Metadata: 100 bytes
Total: ~400 bytes per tweet

Daily Storage:
300M tweets/day × 400 bytes = 120 GB/day

Yearly Storage:
120 GB/day × 365 days = 43.8 TB/year

5-Year Storage:
43.8 TB × 5 = 219 TB ≈ 220 TB

With replication (3x):
220 TB × 3 = 660 TB
```

**Bandwidth Estimates:**

```
Incoming (Write):
3,000 tweets/second × 400 bytes = 1.2 MB/s

Outgoing (Read):
Assume average tweet display = 1 KB (with user info, images metadata)
150,000 views/second × 1 KB = 150 MB/s

Total Bandwidth: ~151 MB/s ≈ 1.2 Gbps
```

**Memory/Cache Estimates:**

```
Using 80-20 rule (20% of tweets generate 80% of reads)

Daily tweets to cache:
300M tweets × 0.2 = 60M tweets
60M × 400 bytes = 24 GB

With metadata and user info:
24 GB × 2 = 48 GB cache size
```

---

## High-Level Architecture Patterns

### 1. Layered Architecture (N-Tier)

```
┌──────────────────────────────┐
│   Presentation Layer (UI)    │ ← Handles user interface
├──────────────────────────────┤
│   Application Layer (API)    │ ← Business logic, orchestration
├──────────────────────────────┤
│   Domain Layer (Services)    │ ← Core business rules
├──────────────────────────────┤
│   Data Access Layer (Repo)   │ ← Database operations
├──────────────────────────────┤
│   Database Layer             │ ← Data storage
└──────────────────────────────┘
```

**Pros:**
- Clear separation of concerns
- Easy to understand and maintain
- Good for traditional web applications

**Cons:**
- Can become monolithic
- Tight coupling between layers
- Difficult to scale independently

**Use When:**
- Building CRUD applications
- Team is familiar with pattern
- Moderate complexity

**Your School Admin System Uses This!**

### 2. Microservices Architecture

```
┌─────────┐   ┌─────────┐   ┌─────────┐
│  User   │   │ Order   │   │ Payment │
│ Service │   │ Service │   │ Service │
└────┬────┘   └────┬────┘   └────┬────┘
     │             │             │
     └─────────────┼─────────────┘
                   │
            ┌──────┴──────┐
            │  API Gateway │
            └──────┬──────┘
                   │
            ┌──────┴──────┐
            │   Clients   │
            └─────────────┘

Each service has its own database:
User DB    Order DB    Payment DB
```

**Pros:**
- Independent scaling
- Technology diversity
- Fault isolation
- Faster deployments

**Cons:**
- Increased complexity
- Network latency
- Data consistency challenges
- More operational overhead

**Use When:**
- Large, complex applications
- Need independent scaling
- Multiple teams working in parallel
- Different technology requirements

### 3. Event-Driven Architecture

```
┌─────────┐        ┌──────────────┐
│ Service │──event→│ Message Queue│
│    A    │        │  (Kafka/SQS) │
└─────────┘        └──────┬───────┘
                          │
                    ┌─────┴─────┐
                    │           │
              ┌─────┴────┐ ┌────┴─────┐
              │ Service  │ │ Service  │
              │    B     │ │    C     │
              └──────────┘ └──────────┘
```

**Pros:**
- Loose coupling
- Scalability
- Resilience (async processing)
- Flexibility

**Cons:**
- Harder to debug
- Eventual consistency
- Message ordering challenges

**Use When:**
- Asynchronous processing needed
- Need to decouple services
- Event sourcing requirements
- Real-time data processing

### 4. Client-Server Architecture

```
┌──────────┐  HTTP/HTTPS   ┌──────────┐
│  Client  │ ────────────→ │  Server  │
│ (Browser)│ ←──────────── │   API    │
└──────────┘   JSON/HTML   └────┬─────┘
                                │
                           ┌────┴─────┐
                           │ Database │
                           └──────────┘
```

**Most common for web applications - your project uses this!**

### 5. Peer-to-Peer (P2P)

```
┌─────┐     ┌─────┐
│ Peer│←───→│ Peer│
│  1  │     │  2  │
└──┬──┘     └──┬──┘
   │           │
   └─────┬─────┘
         │
      ┌──┴──┐
      │Peer │
      │  3  │
      └─────┘
```

**Use When:**
- File sharing (BitTorrent)
- Blockchain systems
- Video conferencing (WebRTC)

---

## Component Design Patterns

### 1. Load Balancer

**Purpose:** Distribute traffic across multiple servers

```
                 ┌────────┐
    Request ────→│  Load  │
                 │Balancer│
                 └───┬────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
    ┌───┴───┐   ┌────┴───┐   ┌───┴───┐
    │Server │   │Server  │   │Server │
    │   1   │   │   2    │   │   3   │
    └───────┘   └────────┘   └───────┘
```

**Algorithms:**
- **Round Robin**: Distribute requests sequentially
- **Least Connections**: Send to server with fewest active connections
- **IP Hash**: Route based on client IP (sticky sessions)
- **Weighted Round Robin**: Servers with more capacity get more requests

**Technologies:**
- Nginx
- HAProxy
- AWS ELB/ALB
- Google Cloud Load Balancer

### 2. Reverse Proxy

**Purpose:** Single entry point, SSL termination, caching

```
Internet ─→ Reverse Proxy ─→ Application Servers
              (Nginx)
```

**Benefits:**
- SSL/TLS termination
- Caching static content
- Compression
- DDoS protection

### 3. API Gateway

**Purpose:** Single entry point for microservices

```
Mobile ─┐
Web ────┼→ API Gateway ─→ Authentication
Desktop─┘        │         Authorization
                 │         Rate Limiting
                 │         Routing
                 │
        ┌────────┼────────┐
        ↓        ↓        ↓
    Service  Service  Service
       A        B        C
```

**Responsibilities:**
- Request routing
- Authentication
- Rate limiting
- Request/response transformation
- Monitoring

**Technologies:**
- Kong
- AWS API Gateway
- Azure API Management
- Express Gateway

### 4. Message Queue

**Purpose:** Asynchronous communication, decoupling

```
Producer ─→ [Queue] ─→ Consumer
              │
              ├─→ Consumer 2
              └─→ Consumer 3
```

**Use Cases:**
- Email sending
- Image processing
- Order processing
- Log aggregation

**Technologies:**
- RabbitMQ (AMQP)
- Apache Kafka (high throughput)
- AWS SQS
- Redis Pub/Sub

**Message Queue vs Pub/Sub:**

```
Queue (Point-to-Point):
Producer → [Queue] → One Consumer gets message

Pub/Sub (Broadcast):
Publisher → [Topic] → All Subscribers get message
```

### 5. CDN (Content Delivery Network)

**Purpose:** Deliver static content from edge locations

```
┌─────────┐
│ Client  │
│  (USA)  │
└────┬────┘
     │ Request image
     ↓
┌─────────────┐
│  CDN Edge   │ ← If cached, return immediately
│   (USA)     │
└──────┬──────┘
       │ If not cached
       ↓
┌──────────────┐
│ Origin Server│
│   (Europe)   │
└──────────────┘
```

**Benefits:**
- Reduced latency (geographic proximity)
- Reduced load on origin server
- DDoS protection
- Better availability

**Technologies:**
- Cloudflare
- AWS CloudFront
- Akamai
- Fastly

---

## API Design

### REST API Design

**Principles:**
1. Use nouns, not verbs in endpoints
2. Use plural nouns for collections
3. Use HTTP methods correctly
4. Use proper status codes

**Example: Student Management**

```
Good REST API Design:
---------------------
GET    /api/students           # List all students
GET    /api/students/123       # Get student by ID
POST   /api/students           # Create new student
PUT    /api/students/123       # Update entire student
PATCH  /api/students/123       # Partial update
DELETE /api/students/123       # Delete student

GET    /api/students/123/classes  # Get student's classes

Bad Design:
-----------
GET    /api/getStudent         # ❌ Verb in URL
POST   /api/student/create     # ❌ Unnecessary action
GET    /api/students-list      # ❌ Redundant suffix
```

**HTTP Status Codes:**

```
Success:
200 OK              - Request succeeded
201 Created         - Resource created
204 No Content      - Success, no body to return

Client Errors:
400 Bad Request     - Invalid request
401 Unauthorized    - Authentication required
403 Forbidden       - No permission
404 Not Found       - Resource doesn't exist
409 Conflict        - Resource conflict (duplicate)
422 Unprocessable   - Validation failed

Server Errors:
500 Internal Error  - Server error
503 Service Unavailable - Server overloaded/down
```

**Pagination:**

```
GET /api/students?page=2&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "currentPage": 2,
    "totalPages": 10,
    "totalItems": 200,
    "itemsPerPage": 20,
    "hasNext": true,
    "hasPrevious": true
  }
}
```

**Filtering & Sorting:**

```
GET /api/students?grade=10&sort=name&order=asc

GET /api/students?classId=5&active=true
```

**Versioning:**

```
Option 1: URL path
GET /api/v1/students
GET /api/v2/students

Option 2: Header
GET /api/students
Header: Accept-Version: v1

Option 3: Query parameter (not recommended)
GET /api/students?version=1
```

### GraphQL vs REST

```
REST:
GET /api/students/123
{
  "id": 123,
  "name": "John",
  "email": "john@school.com",
  "grade": 10,
  "classes": [...], # May not need this
  "teachers": [...] # Or this
}
# ❌ Over-fetching: Getting data you don't need

GraphQL:
query {
  student(id: 123) {
    name
    email
  }
}
# ✅ Only request what you need

{
  "data": {
    "student": {
      "name": "John",
      "email": "john@school.com"
    }
  }
}
```

**When to use GraphQL:**
- Mobile apps (reduce bandwidth)
- Need flexible queries
- Rapidly changing requirements

**When to use REST:**
- Simple CRUD operations
- Caching is important
- Team familiar with REST

---

## Data Flow & Communication Patterns

### 1. Synchronous Communication

```
Client ──Request──→ Server
       ←─Response─┘
       (waits)
```

**Pros:**
- Simple to understand
- Immediate response
- Easy error handling

**Cons:**
- Tight coupling
- Blocking (client waits)
- Cascading failures

**Use When:**
- Need immediate response
- User is waiting

### 2. Asynchronous Communication

```
Client ──Request──→ Queue ──→ Worker
       ←─ACK─────┘
       (continues)

Later: Worker ──Result──→ Notification Service ──→ Client
```

**Pros:**
- Non-blocking
- Better scalability
- Fault tolerance

**Cons:**
- More complex
- Eventual consistency
- Harder to debug

**Use When:**
- Long-running tasks (video encoding)
- Batch processing
- Don't need immediate response

### 3. Request-Response Pattern

**HTTP Example:**
```typescript
// Client
const response = await fetch('/api/students');
const data = await response.json();
```

**Use:** REST APIs, typical web requests

### 4. Publisher-Subscriber Pattern

```
┌───────────┐
│ Publisher │ (Order Service creates order)
└─────┬─────┘
      │ Publishes "OrderCreated" event
      ↓
┌─────────────┐
│    Topic    │
└──────┬──────┘
       │
   ┌───┼────┐
   ↓   ↓    ↓
 Sub1 Sub2 Sub3
(Email) (Inventory) (Analytics)
```

**Use:** Event-driven systems, notifications

### 5. Request-Reply with Queue

```
Client → [Request Queue] → Server
         ↓
Client ← [Reply Queue] ← Server
```

**Use:** Microservices communication

---

## Caching Strategies

### Why Cache?

```
Without Cache:
Client → API → Database (100ms)
Total: 100ms

With Cache:
Client → API → Cache (1ms)
Total: 1ms

100x faster!
```

### Where to Cache?

```
┌─────────┐
│ Browser │ ← 1. Client-side cache
└────┬────┘
     ↓
┌─────────┐
│   CDN   │ ← 2. CDN cache (static files)
└────┬────┘
     ↓
┌─────────┐
│   API   │
│ Server  │ ← 3. Application cache (Redis)
└────┬────┘
     ↓
┌─────────┐
│Database │ ← 4. Database cache (query cache)
└─────────┘
```

### Caching Patterns

#### 1. Cache-Aside (Lazy Loading)

```
1. Client requests data
2. API checks cache
3. If HIT: Return from cache
4. If MISS:
   → Query database
   → Store in cache
   → Return to client

Code:
async function getStudent(id) {
  // Check cache
  let student = await cache.get(`student:${id}`);

  if (student) {
    return student; // Cache HIT
  }

  // Cache MISS - query database
  student = await db.query('SELECT * FROM students WHERE id = ?', [id]);

  // Store in cache (expire in 1 hour)
  await cache.set(`student:${id}`, student, 3600);

  return student;
}
```

**Pros:**
- Only cache what's requested
- Resilient (cache failure → read from DB)

**Cons:**
- Cache miss penalty (3 round trips)
- Stale data possible

**Use When:**
- Read-heavy workloads
- Data requested unpredictably

#### 2. Write-Through Cache

```
1. Client writes data
2. API writes to cache
3. API writes to database (synchronously)
4. Return success

Code:
async function updateStudent(id, data) {
  // Write to database
  await db.query('UPDATE students SET ? WHERE id = ?', [data, id]);

  // Update cache
  await cache.set(`student:${id}`, data, 3600);

  return data;
}
```

**Pros:**
- Cache always in sync
- No stale data

**Cons:**
- Write latency (2 writes)
- Unused data may be cached

**Use When:**
- Need consistent reads
- Data frequently read after write

#### 3. Write-Behind (Write-Back) Cache

```
1. Client writes data
2. API writes to cache
3. Return success (immediately)
4. Asynchronously write to database (later)

Code:
async function updateStudent(id, data) {
  // Write to cache immediately
  await cache.set(`student:${id}`, data, 3600);

  // Queue database write for later
  await writeQueue.add({ id, data });

  return data; // Fast response
}
```

**Pros:**
- Fastest writes
- Batch writes to DB

**Cons:**
- Data loss risk (cache fails before DB write)
- Complex implementation

**Use When:**
- Write-heavy workloads
- Can tolerate potential data loss

#### 4. Refresh-Ahead Cache

```
1. Before cache expires, proactively refresh
2. Keep cache warm

Code:
async function getPopularStudent(id) {
  const TTL = 3600; // 1 hour
  const REFRESH_THRESHOLD = 300; // 5 minutes

  const cached = await cache.get(`student:${id}`);
  const ttl = await cache.ttl(`student:${id}`);

  // If close to expiration, refresh in background
  if (ttl < REFRESH_THRESHOLD) {
    refreshInBackground(id);
  }

  return cached;
}
```

**Use When:**
- Predictable access patterns
- Expensive computations

### Cache Eviction Policies

```
LRU (Least Recently Used):
- Evict oldest accessed item
- Most common
- Redis default

LFU (Least Frequently Used):
- Evict least accessed item
- Better for different access patterns

FIFO (First In First Out):
- Evict oldest item (by insertion time)
- Simple but less effective

TTL (Time To Live):
- Auto-expire after time
- Prevents stale data
```

### Cache Invalidation

**"There are only two hard things in Computer Science: cache invalidation and naming things."**

#### Strategies:

**1. TTL-based (Time-based expiration)**
```typescript
cache.set('student:123', data, 3600); // Expire in 1 hour
```

**2. Event-based (Invalidate on write)**
```typescript
async function updateStudent(id, data) {
  await db.update(id, data);
  await cache.del(`student:${id}`); // Invalidate cache
}
```

**3. Tag-based (Group invalidation)**
```typescript
// Tag cache entries
cache.set('student:123', data, { tags: ['students', 'class:10'] });

// Invalidate all grade 10 students
cache.invalidateTag('class:10');
```

### Redis Caching Example

```typescript
import Redis from 'ioredis';

const redis = new Redis();

class StudentCache {
  // Cache-aside pattern
  async getStudent(id: number): Promise<Student> {
    const cacheKey = `student:${id}`;

    // Try cache first
    const cached = await redis.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }

    // Cache miss - query DB
    const student = await db.Student.findByPk(id);

    // Store in cache (1 hour TTL)
    await redis.setex(cacheKey, 3600, JSON.stringify(student));

    return student;
  }

  // Invalidate on update
  async updateStudent(id: number, data: Partial<Student>): Promise<Student> {
    const student = await db.Student.update(data, { where: { id } });

    // Invalidate cache
    await redis.del(`student:${id}`);

    return student;
  }

  // Cache list with pagination
  async getStudents(page: number, limit: number): Promise<Student[]> {
    const cacheKey = `students:page:${page}:limit:${limit}`;

    const cached = await redis.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }

    const students = await db.Student.findAll({
      offset: (page - 1) * limit,
      limit,
    });

    // Cache for 5 minutes (lists change more frequently)
    await redis.setex(cacheKey, 300, JSON.stringify(students));

    return students;
  }
}
```

---

## Load Balancing

### Types of Load Balancers

#### 1. Layer 4 (Transport Layer) - TCP/UDP

```
Client ──TCP SYN──→ Load Balancer ──→ Server
                    (Checks IP:Port)
```

**Characteristics:**
- Routes based on IP and port
- Fast (no packet inspection)
- Can't read HTTP headers
- Connection-based

**Use:** High throughput, simple routing

#### 2. Layer 7 (Application Layer) - HTTP

```
Client ──HTTP GET /api/students──→ Load Balancer
                                   (Reads URL, headers)
                                   ├─→ API Server 1 (if /api/*)
                                   └─→ Static Server (if /static/*)
```

**Characteristics:**
- Routes based on content (URL, headers, cookies)
- Can terminate SSL
- More flexible
- Slower than L4

**Use:** Microservices, content-based routing

### Load Balancing Algorithms

#### 1. Round Robin

```
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1 (cycle repeats)
```

**Pros:** Simple, fair distribution
**Cons:** Doesn't consider server load
**Use:** Servers have similar capacity

#### 2. Weighted Round Robin

```
Server 1 (weight: 3) gets 3 requests
Server 2 (weight: 1) gets 1 request
```

**Use:** Servers have different capacities

#### 3. Least Connections

```
Server 1: 5 connections  ┐
Server 2: 3 connections  ├─ Send to Server 2 (fewest)
Server 3: 7 connections  ┘
```

**Pros:** Adapts to load
**Cons:** More complex
**Use:** Long-lived connections, varied request times

#### 4. IP Hash

```
hash(client_ip) % num_servers = server_index

Client 1 (IP: 192.168.1.1) → hash → Server 2 (always)
Client 2 (IP: 192.168.1.2) → hash → Server 1 (always)
```

**Pros:** Sticky sessions (same client → same server)
**Cons:** Uneven distribution if few clients
**Use:** Session affinity needed, WebSockets

#### 5. Least Response Time

```
Server 1: avg 100ms
Server 2: avg 50ms  ← Choose this
Server 3: avg 200ms
```

**Pros:** Optimal performance
**Cons:** Requires health monitoring
**Use:** Performance-critical applications

### Health Checks

```
Load Balancer ──ping──→ Server
              ←─pong──┘

If no response → Mark unhealthy → Remove from rotation
```

**Types:**
- **Passive:** Monitor actual traffic (mark unhealthy on errors)
- **Active:** Periodic health check requests

**Example:**
```
GET /health
Response: 200 OK { "status": "healthy" }

Check every 10 seconds
Fail threshold: 3 consecutive failures
Success threshold: 2 consecutive successes
```

---

## Microservices vs Monolith

### Monolithic Architecture

```
┌─────────────────────────────────┐
│         Single Application      │
│  ┌────────────────────────────┐ │
│  │    User Management         │ │
│  ├────────────────────────────┤ │
│  │    Order Processing        │ │
│  ├────────────────────────────┤ │
│  │    Payment                 │ │
│  ├────────────────────────────┤ │
│  │    Inventory               │ │
│  └────────────────────────────┘ │
│              ↓                  │
│       Single Database           │
└─────────────────────────────────┘
```

**Pros:**
- ✅ Simple to develop
- ✅ Easy to test
- ✅ Easy to deploy (single unit)
- ✅ Less operational overhead
- ✅ Better performance (in-process calls)

**Cons:**
- ❌ Hard to scale (must scale entire app)
- ❌ Technology lock-in
- ❌ Long-term maintenance difficulty
- ❌ Large codebase becomes unwieldy
- ❌ Single point of failure

**Use When:**
- Small to medium applications
- Small team
- Rapid prototyping
- Simple domain

**Example:** Your school administration system is a monolith (good choice for the scale!)

### Microservices Architecture

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│   User   │  │  Order   │  │ Payment  │  │Inventory │
│ Service  │  │ Service  │  │ Service  │  │ Service  │
└────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │              │             │
   User DB      Order DB      Payment DB    Inventory DB
```

**Pros:**
- ✅ Independent scaling
- ✅ Technology flexibility
- ✅ Fault isolation
- ✅ Faster deployments
- ✅ Team autonomy

**Cons:**
- ❌ Increased complexity
- ❌ Network latency
- ❌ Data consistency challenges
- ❌ Testing complexity
- ❌ Deployment coordination

**Use When:**
- Large, complex applications
- Multiple teams
- Need independent scaling
- Mature DevOps practices

### Decision Framework

```
Start with Monolith if:
✓ Small team (< 10 developers)
✓ Unclear requirements
✓ Need to move fast
✓ Limited ops resources

Consider Microservices if:
✓ Large team (> 20 developers)
✓ Well-defined domain boundaries
✓ Different scaling needs per feature
✓ Strong DevOps culture
✓ Proven product-market fit
```

### Migration Path: Monolith → Microservices

```
Phase 1: Monolith with modules
┌─────────────────────────┐
│  ┌─────┐ ┌─────┐ ┌────┐ │
│  │Mod A│ │Mod B│ │Mod │ │
│  └─────┘ └─────┘ └────┘ │
└─────────────────────────┘

Phase 2: Extract first service
┌───────────────────┐   ┌─────────┐
│    ┌─────┐ ┌────┐ │   │ Service │
│    │Mod B│ │Mod │ │◄─►│   A     │
│    └─────┘ └────┘ │   └─────────┘
└───────────────────┘

Phase 3: Continue extraction
┌─────────────┐  ┌─────────┐  ┌─────────┐
│   ┌────┐    │  │Service  │  │Service  │
│   │Mod │    │◄─┤   A     │  │   B     │
│   └────┘    │  └─────────┘  └─────────┘
└─────────────┘
```

**Strangler Fig Pattern:** Gradually replace monolith pieces

---

## CAP Theorem

### The CAP Theorem

**"You can only guarantee 2 out of 3:"**

```
       Consistency
           ▲
          / \
         /   \
        /     \
       /  CA   \
      /____ ____\
     /  CP   AP  \
    /             \
Partition ────────→ Availability
Tolerance
```

#### Definitions:

**Consistency (C):**
- All nodes see the same data at the same time
- Every read receives the most recent write
- Example: Bank account balance must be accurate across all servers

**Availability (A):**
- Every request receives a response (success or failure)
- No guarantee it's the most recent data
- Example: System responds even if one server is down

**Partition Tolerance (P):**
- System continues to operate despite network failures
- Network can drop/delay messages between nodes
- **In distributed systems, this is mandatory** (networks always fail)

### CAP Combinations

#### CP (Consistency + Partition Tolerance)

```
┌────────┐         ┌────────┐
│ Node 1 │   X     │ Node 2 │  Network partition!
└────────┘         └────────┘

User → Node 2 (write)
Node 2: "Cannot guarantee consistency, rejecting write"
❌ Sacrifice Availability for Consistency
```

**Databases:** MongoDB, HBase, Redis (some modes)
**Use When:** Strong consistency required (financial systems)

#### AP (Availability + Partition Tolerance)

```
┌────────┐         ┌────────┐
│ Node 1 │   X     │ Node 2 │  Network partition!
│balance:│         │balance:│
│  $100  │         │  $50   │  ← Different values!
└────────┘         └────────┘

User → Node 2 (read)
Node 2: "Here's data: $50" (might be stale)
✅ Always responds, but might be inconsistent
```

**Databases:** Cassandra, DynamoDB, CouchDB
**Use When:** High availability critical (social media, caching)

#### CA (Consistency + Availability)

**Only works in single-node systems or perfect networks (doesn't exist in distributed systems)**

**Databases:** Traditional RDBMS (MySQL, PostgreSQL) in single-node setup

### Real-World Examples

#### Example 1: Bank Transaction (Choose CP)

```
User A: Transfer $100 from Account X to Account Y

┌────────┐  sync   ┌────────┐
│ Node 1 │◄───────►│ Node 2 │
│Account │         │Account │
│  X:    │         │  Y:    │
└────────┘         └────────┘

If network partition occurs:
→ Reject transaction (sacrifice availability)
→ Maintain consistency (no money created/lost)
```

#### Example 2: Social Media Feed (Choose AP)

```
User posts status: "Hello World"

┌────────┐   X    ┌────────┐
│ Node 1 │        │ Node 2 │
│Has new │        │No new  │
│ post   │        │ post   │
└────────┘        └────────┘

If network partition occurs:
→ Both nodes still serve reads (availability)
→ Node 2 shows stale feed (eventual consistency)
→ When partition heals, sync up
```

### PACELC Theorem (Extended CAP)

**"If Partition, choose A or C, Else (no partition), choose Latency or Consistency"**

```
PAC/ELC:
- If Partition: Availability vs Consistency
- Else (normal): Latency vs Consistency

Examples:
- DynamoDB: PA/EL (Availability + Low Latency)
- MongoDB: PC/EC (Consistency always)
- Cassandra: PA/EL (Availability + Low Latency)
```

---

## Consistency Patterns

### 1. Strong Consistency

```
Write(X=1) ──┐
             ▼
         [Database]
             │
Read(X) ────►│──► Returns: 1 (guaranteed latest)
```

**Guarantee:** Read always returns latest write

**Implementation:**
- Synchronous replication
- All replicas must acknowledge write

**Trade-off:** Higher latency, lower availability

**Use:** Financial systems, inventory management

### 2. Eventual Consistency

```
Write(X=1) ──► [Primary]
               │
               ├──async──► [Replica 1] (not yet updated)
               └──async──► [Replica 2] (not yet updated)

Read(X) from Replica 1 → Returns: 0 (stale!)

... time passes ...

Read(X) from Replica 1 → Returns: 1 (caught up!)
```

**Guarantee:** All replicas will eventually have same data (no guarantee when)

**Trade-off:** Low latency, high availability, temporary inconsistency

**Use:** Social media, caching, product catalogs

### 3. Read-Your-Own-Writes Consistency

```
User A writes X=1
User A reads X → Always sees 1 (their own write)

User B reads X → Might see 0 or 1 (eventual)
```

**Implementation:**
- Read from same replica you wrote to
- Or, track write timestamp and read from sufficiently up-to-date replica

**Use:** User profile updates, comments

### 4. Monotonic Reads Consistency

```
User reads X → Gets value 5
User reads X again → Gets value 5 or newer (never 4)
```

**Guarantee:** Reads never go backwards in time

**Implementation:** Sticky sessions (always read from same replica)

### 5. Causal Consistency

```
User A posts: "I got engaged!"
User A posts: "Thanks for all the congratulations!"

All users see posts in correct order (cause → effect)
```

**Guarantee:** Causally related writes seen in order

---

## Replication & Partitioning

### Replication (Vertical Scaling of Data)

**Purpose:** Store same data on multiple machines

**Benefits:**
- High availability (failover)
- Better read performance (distribute reads)
- Disaster recovery

#### 1. Primary-Replica (Master-Slave)

```
        ┌─────────┐
        │ Primary │ ← All writes go here
        │ (Write) │
        └────┬────┘
             │
    ┌────────┼────────┐
    │        │        │
    ▼        ▼        ▼
┌────────┐ ┌────────┐ ┌────────┐
│Replica │ │Replica │ │Replica │ ← Reads distributed
│   1    │ │   2    │ │   3    │
└────────┘ └────────┘ └────────┘
```

**Replication:**
- Synchronous: Primary waits for replica ACK (strong consistency, slow)
- Asynchronous: Primary doesn't wait (eventual consistency, fast)

**Pros:**
- Simple
- Good for read-heavy workloads

**Cons:**
- Write bottleneck (single primary)
- Replication lag (async)

**Use:** MySQL, PostgreSQL, MongoDB

#### 2. Multi-Primary (Multi-Master)

```
┌─────────┐       ┌─────────┐
│Primary 1│◄─────►│Primary 2│
│ (Write) │       │ (Write) │
└─────────┘       └─────────┘
Both can accept writes
```

**Pros:**
- No single write bottleneck
- Better availability

**Cons:**
- Write conflicts (need resolution)
- More complex

**Use:** CouchDB, Cassandra (all nodes equal)

**Conflict Resolution:**
```
Primary 1: Sets X=1 at time T1
Primary 2: Sets X=2 at time T2 (T2 > T1)

Strategy 1: Last Write Wins (LWW)
→ X=2 (based on timestamp)

Strategy 2: Application-level resolution
→ Keep both versions, let app decide
```

#### 3. Leaderless Replication

```
┌──────┐  ┌──────┐  ┌──────┐
│Node 1│  │Node 2│  │Node 3│
└──────┘  └──────┘  └──────┘
   ▲         ▲         ▲
   └─────────┼─────────┘
        Client writes to all

Quorum: Write to W nodes, Read from R nodes
Where: W + R > N (total nodes)
```

**Example:** N=3, W=2, R=2

```
Write(X=1):
✓ Node 1: success
✓ Node 2: success  ← 2 ACKs, write complete
✗ Node 3: failed

Read(X):
Query Node 1, Node 2 → Both return 1
(Majority have latest value)
```

**Use:** Cassandra, DynamoDB, Riak

### Partitioning (Sharding) - Horizontal Scaling

**Purpose:** Split data across multiple machines

**Why:** Single machine can't hold all data

#### 1. Horizontal Partitioning (Sharding)

```
Users Table (10M records)

Shard by ID:
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Shard 1  │  │ Shard 2  │  │ Shard 3  │
│ ID 1-3M  │  │ID 3M-6M  │  │ID 6M-10M │
└──────────┘  └──────────┘  └──────────┘
```

**Sharding Strategies:**

##### a. Range-based Sharding

```
Shard 1: A-F
Shard 2: G-M
Shard 3: N-Z

User "Alice" → Shard 1
User "John" → Shard 2
```

**Pros:** Simple, range queries easy
**Cons:** Uneven distribution (hotspots)

##### b. Hash-based Sharding

```
hash(userId) % num_shards = shard_id

hash(user_123) % 3 = 1 → Shard 1
hash(user_456) % 3 = 2 → Shard 2
```

**Pros:** Even distribution
**Cons:** Range queries difficult, resharding hard

##### c. Consistent Hashing

```
Hash Ring (0-360°):
        0°
        │
   Shard 3
       │
  270°─┼─90°
       │    Shard 1
   Shard 2
      180°

hash(user_123) = 45° → Goes to Shard 1 (next clockwise)
hash(user_456) = 200° → Goes to Shard 2
```

**Pros:** Adding/removing shards affects minimal data
**Cons:** Complex implementation

**Use:** DynamoDB, Cassandra, Memcached

#### 2. Vertical Partitioning

```
Users Table:
ID | Name | Email | Address | Phone | ...

Split to:
┌─────────────────┐  ┌─────────────────┐
│ ID | Name | Email│  │ ID | Address | Phone│
└─────────────────┘  └─────────────────┘
Frequently accessed   Less frequently accessed
```

**Use:** Optimize storage, separate hot/cold data

### Replication + Partitioning

```
Shard 1 (ID: 1-3M)                  Shard 2 (ID: 3M-6M)
┌─────────┐                         ┌─────────┐
│ Primary │                         │ Primary │
└────┬────┘                         └────┬────┘
     │                                   │
 ┌───┴───┐                           ┌───┴───┐
 ▼       ▼                           ▼       ▼
┌───┐   ┌───┐                       ┌───┐   ┌───┐
│R1 │   │R2 │                       │R1 │   │R2 │
└───┘   └───┘                       └───┘   └───┘

Each shard is replicated for availability!
```

---

## Common Interview Questions

### Q1: Design a URL shortener (bit.ly)

**Answer Framework:**

```
1. Requirements:
   - Shorten URL: POST /api/shorten { "url": "..." }
   - Redirect: GET /{shortCode}
   - 100M URLs/month
   - 100:1 read-to-write ratio

2. Capacity:
   - 100M writes/month = ~40 writes/second
   - 10B reads/month = ~4000 reads/second
   - Storage: 100M × 500 bytes = 50 GB/year

3. High-level design:
   Client → Load Balancer → API Servers → Database
                                    ↓
                                  Cache (Redis)

4. Short code generation:
   Option 1: Hash (MD5) + Base62 encode → Take first 7 chars
   Option 2: Auto-increment ID → Base62 encode

5. Database:
   Schema:
   - short_code (PK, indexed)
   - original_url
   - created_at
   - expires_at
   - click_count

6. Caching:
   - Cache popular URLs (80-20 rule)
   - TTL: 24 hours
   - Cache size: 20% of 100M = 20M × 500 bytes = 10 GB

7. Scaling:
   - Database sharding by short_code
   - CDN for static assets
   - Rate limiting per IP
```

### Q2: How would you design a system to handle 1 million requests per second?

**Answer:**

```
1. Load Balancing:
   - Multiple load balancers (HA)
   - Layer 7 routing
   - DNS round-robin

2. Application Tier:
   - 100+ servers (assuming 10K RPS per server)
   - Autoscaling based on CPU/memory
   - Stateless servers

3. Caching:
   - Redis cluster (sharded)
   - CDN for static content
   - 90%+ cache hit rate

4. Database:
   - Read replicas (10+)
   - Write to primary, read from replicas
   - Connection pooling
   - Sharding for very high write load

5. Async Processing:
   - Message queues for non-critical work
   - Separate worker pools

6. Monitoring:
   - Real-time metrics (Prometheus + Grafana)
   - Alerts for anomalies
   - Distributed tracing

Key: Start with ballpark numbers
100K RPS → ~100 servers (1K RPS each)
1M RPS → ~1000 servers
```

### Q3: Explain database replication lag and how to handle it

**Answer:**

```
Replication Lag: Time delay between primary write and replica update

┌─────────┐  Write X=1    ┌─────────┐
│ Primary │  (t=0)        │ Replica │
│  X=1    │──────────────►│  X=0    │ (t=0)
└─────────┘               │  X=1    │ (t=100ms)
                          └─────────┘
                          ↑ 100ms lag

Causes:
- Network latency
- High write volume
- Replica slower hardware
- Long-running queries on replica

Problems:
1. User writes, then reads from replica → sees old data
2. Inconsistent reads across replicas

Solutions:

1. Read-your-own-writes:
   - Read from primary after write (for that user)
   - Or track write timestamp, read from up-to-date replica

2. Sticky sessions:
   - Always read from same replica
   - Guarantees monotonic reads

3. Synchronous replication:
   - Wait for replica ACK
   - Trade-off: Higher latency

4. Monitor lag:
   - Alert if lag > threshold (e.g., 5 seconds)
   - Stop routing reads to lagging replicas

5. Application-level:
   - Show "processing" indicator
   - Eventual consistency UI ("updates may take a moment")
```

### Q4: How do you handle database hotspots?

**Answer:**

```
Hotspot: Disproportionate load on one shard/partition

Example:
Celebrity posts tweet → Millions read from same shard

Solutions:

1. Identify hotspots:
   - Monitor query patterns
   - Track shard load

2. Replication:
   - Replicate hot data to multiple nodes
   - Distribute reads across replicas

3. Caching:
   - Cache hot items aggressively
   - Reduce database hits

4. Re-sharding:
   - Hash sharding for even distribution
   - Consistent hashing

5. Application-level:
   - Denormalize hot data
   - Pre-compute popular aggregations

6. Database-level:
   - Partition by different key
   - Use composite keys

Example: Twitter celebrity tweets
- Cache celebrity profiles/tweets heavily
- Replicate to multiple data centers
- Use CDN for media
- Fan-out on write (pre-compute timelines)
```

---

## Summary: System Design Checklist

When designing any system, cover these areas:

```
☐ Clarify Requirements
  ☐ Functional requirements
  ☐ Non-functional (scale, latency, availability)
  ☐ Constraints

☐ Capacity Estimation
  ☐ Traffic (QPS)
  ☐ Storage
  ☐ Bandwidth

☐ High-Level Design
  ☐ Client → Load Balancer → API → Database
  ☐ Identify components
  ☐ Data flow

☐ Database Design
  ☐ Schema
  ☐ Indexes
  ☐ SQL vs NoSQL
  ☐ Replication strategy
  ☐ Partitioning strategy

☐ Caching
  ☐ What to cache
  ☐ Cache invalidation
  ☐ TTL

☐ API Design
  ☐ REST endpoints
  ☐ Request/response format
  ☐ Versioning

☐ Scalability
  ☐ Horizontal vs vertical
  ☐ Stateless servers
  ☐ Database sharding
  ☐ CDN

☐ Reliability
  ☐ Single points of failure
  ☐ Replication
  ☐ Health checks
  ☐ Failover

☐ Consistency
  ☐ Strong vs eventual
  ☐ CAP trade-offs

☐ Security
  ☐ Authentication
  ☐ Authorization
  ☐ Rate limiting
  ☐ Encryption

☐ Monitoring
  ☐ Metrics
  ☐ Logging
  ☐ Alerts
```

Remember: **No perfect design, only trade-offs!**
