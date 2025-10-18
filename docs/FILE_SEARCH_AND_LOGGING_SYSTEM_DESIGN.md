# File Search & Logging System Design

This guide focuses on **system design interview preparation** for two common scenarios:
1. **File Search System** - Design a scalable file searching service
2. **Centralized Logging System** - Design logging infrastructure for microservices

Each design includes architecture diagrams, detailed flows, scaling strategies, and trade-off analysis.

---

## Table of Contents

1. [File Search System Design](#1-file-search-system-design)
2. [Centralized Logging System Design](#2-centralized-logging-system-design)

---

## 1. File Search System Design

### Problem Statement

**Design a file search system like Google Drive or Dropbox search that allows users to:**
- Search files by name (prefix, suffix, contains, exact match)
- Search by file metadata (type, size, modified date, owner)
- Search file content (for text files)
- Get results quickly (< 100ms)
- Support millions of files and thousands of concurrent searches

---

### Requirements Clarification

#### Functional Requirements
- Search files by name with multiple match types
- Filter by extension, size, date range
- Content search for text files
- Pagination for large result sets
- Real-time updates (new files appear in search immediately)

#### Non-Functional Requirements
- **Latency:** < 100ms for search queries
- **Scale:** Support 100M files, 10K concurrent users
- **Availability:** 99.9% uptime
- **Consistency:** Eventually consistent (acceptable for search)

#### Capacity Estimates
```
Users: 10M total, 100K daily active
Files per user: 1000 average
Total files: 10B (10 billion)
Searches per day: 10M
QPS: 10M / 86400 = ~115 queries/second (peak: ~500 QPS)

Storage:
- Average file metadata: 1KB
- Total metadata: 10B × 1KB = 10TB
- With replication (3x): 30TB

Bandwidth:
- Search query size: 500 bytes
- Response size: 50KB (50 results × 1KB each)
- Bandwidth: 500 QPS × 50KB = 25 MB/s
```

---

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Layer                            │
│  (Web App, Mobile App, Desktop App)                             │
└────────────────────────┬────────────────────────────────────────┘
                         │ HTTPS
                         v
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway / Load Balancer                │
│  - Rate limiting (100 requests/min per user)                    │
│  - Authentication (JWT tokens)                                  │
│  - Request routing                                              │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┼────────────────┐
         v               v                v
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Search     │  │   Indexing   │  │   Metadata   │
│   Service    │  │   Service    │  │   Service    │
│              │  │              │  │              │
│ - Query      │  │ - File watch │  │ - CRUD ops   │
│   parsing    │  │ - Index      │  │ - File info  │
│ - Result     │  │   building   │  │              │
│   ranking    │  │ - Updates    │  │              │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       v                 v                 v
┌─────────────────────────────────────────────────────────────────┐
│                      Data Storage Layer                         │
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│  │Elasticsearch │    │    Redis     │    │  PostgreSQL  │     │
│  │  (Search     │    │   (Cache)    │    │  (Metadata)  │     │
│  │   Index)     │    │              │    │              │     │
│  │              │    │ - Hot data   │    │ - File info  │     │
│  │ - Inverted   │    │ - User prefs │    │ - User data  │     │
│  │   index      │    │ - Recent     │    │ - Permissions│     │
│  │ - Full-text  │    │   searches   │    │              │     │
│  └──────────────┘    └──────────────┘    └──────────────┘     │
└─────────────────────────────────────────────────────────────────┘
       │
       v
┌─────────────────────────────────────────────────────────────────┐
│                  Message Queue (Kafka)                          │
│  Topics: file.created, file.updated, file.deleted              │
└─────────────────────────────────────────────────────────────────┘
```

---

### Core Components

#### 1. Search Service
**Responsibilities:**
- Receive and parse search queries
- Query Elasticsearch index
- Rank and filter results
- Apply user permissions
- Return paginated results

**Query Flow:**
```
User Query: "report 2023" extension:pdf size:>1MB

Parsed Query:
{
  "text": "report 2023",
  "filters": {
    "extension": "pdf",
    "size": { "gte": 1048576 }
  }
}

Elasticsearch Query:
{
  "query": {
    "bool": {
      "must": [
        { "match": { "filename": "report 2023" } }
      ],
      "filter": [
        { "term": { "extension": "pdf" } },
        { "range": { "size": { "gte": 1048576 } } }
      ]
    }
  },
  "from": 0,
  "size": 20
}
```

#### 2. Indexing Service
**Responsibilities:**
- Monitor file system changes (file watcher)
- Extract metadata (name, size, type, modified date)
- Build search index
- Handle index updates

**Key Design Decision:** Push vs Pull Indexing

**Option A: Push-Based (Event-Driven)**
```
File Storage → Event (file.created) → Kafka → Indexing Service → Elasticsearch
```
- ✅ Real-time updates
- ✅ Decoupled architecture
- ❌ Eventual consistency

**Option B: Pull-Based (Periodic Scan)**
```
Indexing Service → Periodic Scan → File Storage → Build Index → Elasticsearch
```
- ✅ Simpler implementation
- ✅ Strong consistency
- ❌ Higher latency
- ❌ Resource intensive

**Recommendation:** Use push-based for production systems

#### 3. Metadata Service
**Responsibilities:**
- Store file metadata in PostgreSQL
- Handle CRUD operations
- Manage file permissions
- Serve as source of truth

**Database Schema:**
```
files:
- file_id (PK)
- user_id (FK)
- name
- extension
- size
- path
- parent_folder_id
- created_at
- modified_at
- is_deleted

Indexes:
- (user_id, name)
- (user_id, modified_at)
- (extension)
```

---

### Detailed Search Flow

```
┌─────────┐
│  User   │
└────┬────┘
     │ 1. Search Query: "budget report"
     │    extension:xlsx, modified:last-7-days
     v
┌─────────────────┐
│  API Gateway    │
└────┬────────────┘
     │ 2. Authenticate & Rate Limit
     │    User ID: user_123
     v
┌─────────────────┐
│ Search Service  │
└────┬────────────┘
     │ 3. Check Cache (Redis)
     │    Key: search:user_123:query_hash
     │    → Cache MISS
     v
┌─────────────────┐
│ Search Service  │
│ Query Parser    │
└────┬────────────┘
     │ 4. Parse Query
     │    text: "budget report"
     │    filters: {extension: xlsx, date: last-7-days}
     v
┌─────────────────┐
│ Elasticsearch   │
└────┬────────────┘
     │ 5. Execute Search
     │    - Full-text search on filename
     │    - Apply filters
     │    - Score and rank results
     │    → Found 234 matches
     v
┌─────────────────┐
│ Search Service  │
│ Post-Processing │
└────┬────────────┘
     │ 6. Apply Permissions
     │    - Filter out files user can't access
     │    - Check sharing settings
     │    → 187 accessible files
     │
     │ 7. Rank Results
     │    - Exact matches first
     │    - Recently modified boost
     │    - Frequently accessed boost
     │
     │ 8. Paginate
     │    - Return first 20 results
     │    - Include total count
     │    - Provide cursor for next page
     v
┌─────────────────┐
│ Redis Cache     │
└────┬────────────┘
     │ 9. Cache Results
     │    TTL: 5 minutes
     v
┌─────────┐
│  User   │
│ (JSON)  │
└─────────┘
{
  "results": [...],
  "total": 187,
  "page": 1,
  "hasMore": true,
  "searchTime": "23ms"
}
```

---

### Indexing Flow (Real-Time Updates)

```
┌─────────────────┐
│  File Storage   │
│   (S3, HDFS)    │
└────┬────────────┘
     │ Event: File Created/Modified/Deleted
     v
┌─────────────────┐
│  Event Stream   │
│    (Kafka)      │
└────┬────────────┘
     │ Topic: file.events
     │ Message: {
     │   event_type: "created",
     │   file_id: "f123",
     │   path: "/docs/report.pdf",
     │   size: 2048576,
     │   timestamp: "2025-10-14T10:30:00Z"
     │ }
     v
┌─────────────────┐
│ Indexing Service│
│   (Consumer)    │
└────┬────────────┘
     │ 1. Extract Metadata
     │    - Filename: "report.pdf"
     │    - Extension: "pdf"
     │    - Path segments: ["docs", "report.pdf"]
     │
     │ 2. Content Extraction (if text file)
     │    - Extract first 10,000 characters
     │    - Remove special characters
     │    - Generate text preview
     │
     │ 3. Generate Search Document
     │    {
     │      "file_id": "f123",
     │      "filename": "report",
     │      "extension": "pdf",
     │      "size": 2048576,
     │      "path": "/docs/report.pdf",
     │      "user_id": "user_123",
     │      "modified_at": "2025-10-14T10:30:00Z",
     │      "content_preview": "..."
     │    }
     v
┌─────────────────┐
│ Elasticsearch   │
└────┬────────────┘
     │ 4. Index Document
     │    - Create inverted index
     │    - Update term frequencies
     │    - Store document
     │    → Indexed successfully
     v
┌─────────────────┐
│  PostgreSQL     │
└─────────────────┘
     5. Update Metadata DB
        - Mark as indexed
        - Update timestamps
```

---

### Search Index Design (Elasticsearch)

#### Index Structure
```
Index: files-{user_id}
Sharding: By user_id (each user's files on specific shards)
Replicas: 2 (for high availability)

Document Structure:
{
  "file_id": "f123",
  "filename": "Q4 Budget Report 2023",
  "filename_normalized": "q4 budget report 2023",
  "extension": "xlsx",
  "size": 2048576,
  "path": "/documents/finance/2023/Q4 Budget Report 2023.xlsx",
  "path_segments": ["documents", "finance", "2023"],
  "user_id": "user_123",
  "owner_name": "John Doe",
  "created_at": "2023-10-01T09:00:00Z",
  "modified_at": "2023-12-15T14:30:00Z",
  "content_preview": "Summary of Q4 expenses...",
  "tags": ["finance", "budget", "2023"],
  "is_shared": false,
  "access_count": 45,
  "last_accessed": "2025-10-13T08:00:00Z"
}

Mappings:
{
  "properties": {
    "filename": { "type": "text", "analyzer": "standard" },
    "filename_normalized": { "type": "text", "analyzer": "lowercase" },
    "extension": { "type": "keyword" },
    "size": { "type": "long" },
    "path": { "type": "keyword" },
    "path_segments": { "type": "keyword" },
    "modified_at": { "type": "date" },
    "content_preview": { "type": "text" },
    "tags": { "type": "keyword" }
  }
}
```

#### Search Ranking Algorithm
```
Score = (Text Relevance Score) × (Boost Factors)

Text Relevance (TF-IDF):
- Term Frequency: How often search term appears in filename
- Inverse Document Frequency: How rare the term is across all files

Boost Factors:
1. Exact Match Boost: 3x
   - "report.pdf" exact match for query "report.pdf"

2. Prefix Match Boost: 2x
   - "report_2023.pdf" for query "report"

3. Recency Boost: 1.5x for files modified < 30 days
   boost = 1.5 / (1 + days_old / 30)

4. Access Frequency Boost: 1.2x for frequently accessed
   boost = 1 + log(access_count + 1) / 10

5. File Type Boost: User preference
   - If user frequently opens PDFs, boost PDF results

Final Score Example:
- Base TF-IDF Score: 5.2
- Exact Match: 5.2 × 3 = 15.6
- Recency (10 days old): 15.6 × 1.35 = 21.06
- Access Count (50 times): 21.06 × 1.2 = 25.27
```

---

### Scaling Strategies

#### 1. Horizontal Scaling

**Search Service:**
```
                  ┌─────────────┐
                  │Load Balancer│
                  └──────┬──────┘
         ┌────────────┼────────────┐
         v            v            v
    ┌────────┐  ┌────────┐  ┌────────┐
    │Search 1│  │Search 2│  │Search N│
    └────────┘  └────────┘  └────────┘
```
- Stateless services (easy to scale)
- Auto-scaling based on CPU/QPS
- Target: 100 QPS per instance

**Elasticsearch Cluster:**
```
Data Sharding:
- Shard 1: Users 0-999,999
- Shard 2: Users 1M-1.999M
- Shard N: Users NM-(N+1)M

Each shard has 2 replicas:
Shard 1: [Primary, Replica 1, Replica 2]
```

#### 2. Caching Strategy

**Layer 1: Browser Cache**
- Cache search results for 5 minutes
- Invalidate on file changes

**Layer 2: CDN (CloudFront)**
- Cache static assets
- Cache popular search results

**Layer 3: Application Cache (Redis)**
```
Cache Keys:
1. Search Results
   Key: search:{user_id}:{query_hash}
   TTL: 5 minutes
   Value: {results, total, timestamp}

2. User Preferences
   Key: user:{user_id}:prefs
   TTL: 1 hour
   Value: {file_type_prefs, sorting, filters}

3. File Metadata (Hot Data)
   Key: file:{file_id}
   TTL: 15 minutes
   Value: {name, size, modified_at, ...}

Cache Eviction: LRU (Least Recently Used)
Cache Size: 100GB per Redis instance
Hit Rate Target: > 80%
```

#### 3. Database Optimization

**PostgreSQL (Metadata Store):**
```
Partitioning:
- Partition by user_id (100K users per partition)
- Time-based partitioning for audit logs

Indexes:
- B-Tree: (user_id, filename)
- B-Tree: (user_id, modified_at DESC)
- GiST: (path) for prefix searches

Connection Pooling:
- Max connections: 1000
- Idle timeout: 60s
- Pool size per service: 20

Read Replicas:
- 1 master (writes)
- 5 replicas (reads)
- Read/Write split: 90% reads, 10% writes
```

**Elasticsearch Optimization:**
```
Index Lifecycle:
- Hot tier (0-7 days): SSD, all replicas
- Warm tier (7-90 days): SSD, fewer replicas
- Cold tier (90-365 days): HDD, 1 replica
- Delete tier (> 365 days): Archive to S3

Query Optimization:
- Use filters instead of queries (cacheable)
- Limit fields returned (_source filtering)
- Use pagination (scroll API for deep pagination)
- Disable scoring for filters (filter context)
```

#### 4. Content Search Optimization

**Problem:** Full-text search on large files is slow

**Solution: Chunking + Indexing**
```
Large File (100MB PDF)
  ↓
Extract Text (100K words)
  ↓
Split into Chunks (1000 words each)
  ↓
Index Each Chunk Separately
  ↓
100 searchable documents

Search Result:
- Return matching chunks
- Show preview with highlighted terms
- Link to specific page/section in file
```

---

### Trade-Offs and Alternatives

#### Approach Comparison

| Aspect | Simple DB Query | Elasticsearch | Trie/Prefix Tree |
|--------|----------------|---------------|------------------|
| **Search Speed** | O(n) - Slow | O(log n) - Fast | O(k) - Very Fast |
| **Relevance Ranking** | Basic | Advanced (TF-IDF) | None |
| **Full-Text Search** | Limited | Excellent | Poor |
| **Memory Usage** | Low | High | Very High |
| **Scalability** | Poor | Excellent | Limited |
| **Setup Complexity** | Low | Medium | High |
| **Best For** | < 10K files | Production | Autocomplete only |

#### Design Decisions

**1. Why Elasticsearch over PostgreSQL Full-Text Search?**

✅ **Elasticsearch:**
- Distributed by design (horizontal scaling)
- Advanced relevance scoring
- Real-time search (near instant indexing)
- Better performance for complex queries
- Built-in analytics

❌ **PostgreSQL FTS:**
- Limited scalability (vertical only)
- Basic relevance ranking
- Slower for large datasets
- Single point of failure (without setup)

**Trade-off:** Elasticsearch adds complexity and operational overhead, but necessary for scale.

---

**2. Why Eventually Consistent over Strong Consistency?**

**Eventually Consistent (Chosen):**
- ✅ Better performance (no distributed locks)
- ✅ Higher availability (AP in CAP theorem)
- ✅ Acceptable for search (slight delay OK)
- ❌ Search may not show very recent files (< 1 second delay)

**Strong Consistency:**
- ✅ Always shows latest state
- ❌ Slower writes (coordination overhead)
- ❌ Lower availability (must wait for all nodes)

**Decision:** Search latency of 1-2 seconds is acceptable for better performance.

---

**3. Why Push-Based Indexing over Pull-Based?**

**Push (Event-Driven):**
```
File Created → Event → Index Updated
Latency: < 1 second
```
- ✅ Real-time updates
- ✅ Lower resource usage (only index changes)
- ✅ Decoupled architecture
- ❌ Complex setup (Kafka, consumers)

**Pull (Periodic Scan):**
```
Cron Job → Scan All Files → Index Changes
Latency: 5-60 minutes
```
- ✅ Simple implementation
- ❌ Delayed updates
- ❌ Resource intensive (scans all files)
- ❌ Misses quick changes

**Decision:** Real-time search is critical for user experience.

---

### Advanced Features

#### 1. Autocomplete / Suggestions

**Architecture:**
```
┌──────────┐
│  User    │ Types: "rep"
└────┬─────┘
     │ After 3 characters
     v
┌──────────────┐
│ Redis Cache  │ Check: autocomplete:user_123:rep
└────┬─────────┘
     │ Cache MISS
     v
┌──────────────┐
│Elasticsearch │ Prefix Query: filename.prefix = "rep"
│              │ Aggregation: Top 10 matches
└────┬─────────┘
     │ Results: ["report.pdf", "repository.txt", ...]
     v
┌──────────────┐
│ Redis Cache  │ Store with TTL: 1 hour
└──────────────┘

Optimization:
- Use completion suggester in Elasticsearch
- Cache popular prefixes
- Update suggestions based on user history
```

#### 2. Personalized Search

**Ranking Personalization:**
```
User Profile:
{
  "user_id": "user_123",
  "file_type_preference": {
    "pdf": 0.4,
    "docx": 0.3,
    "xlsx": 0.2,
    "txt": 0.1
  },
  "frequent_folders": ["/work/projects", "/personal"],
  "recent_searches": ["budget", "report", "presentation"],
  "collaboration_network": ["user_456", "user_789"]
}

Personalized Boost:
- Boost preferred file types: +30%
- Boost files in frequent folders: +20%
- Boost files shared by colleagues: +15%
- Boost files matching recent searches: +10%
```

#### 3. Fuzzy Search (Typo Tolerance)

**Levenshtein Distance:**
```
Query: "reprot" (typo)
Fuzzy match: "report" (distance: 2)

Elasticsearch Fuzzy Query:
{
  "query": {
    "fuzzy": {
      "filename": {
        "value": "reprot",
        "fuzziness": 2,
        "prefix_length": 2
      }
    }
  }
}

Results:
- "report.pdf" (exact match after fuzzy)
- "report_2023.xlsx"
- "quarterly_report.docx"
```

---

### Monitoring & Observability

**Key Metrics:**
```
1. Search Performance:
   - P50, P95, P99 latency
   - QPS (queries per second)
   - Error rate
   - Cache hit rate

2. Index Health:
   - Indexing lag (time to index new file)
   - Index size
   - Shard distribution
   - Replication lag

3. Business Metrics:
   - Searches per user
   - Zero-result searches (need better indexing?)
   - Search abandonment rate
   - Click-through rate

Alerts:
- P99 latency > 500ms
- Error rate > 1%
- Indexing lag > 10 seconds
- Elasticsearch cluster health: Red/Yellow
```

---

## 2. Centralized Logging System Design

### Problem Statement

**Design a centralized logging system for a microservices architecture that:**
- Collects logs from 100+ services
- Ingests 1TB of logs per day
- Enables fast log search and analysis
- Supports real-time monitoring and alerting
- Retains logs for compliance (1 year)

---

### Requirements Clarification

#### Functional Requirements
- Collect logs from all microservices
- Support structured logging (JSON)
- Real-time log streaming
- Search logs by service, level, timestamp, correlation ID
- Aggregate and visualize metrics
- Alert on errors/anomalies
- Log retention with tiered storage

#### Non-Functional Requirements
- **Throughput:** 1M log events/second
- **Latency:** < 1 second from log generation to searchability
- **Availability:** 99.95% uptime
- **Scalability:** Support 1000+ services
- **Durability:** No log loss (at-least-once delivery)

#### Capacity Estimates
```
Services: 100 microservices
Logs per service: 10,000 events/sec
Total: 1M events/sec

Average log size: 1KB
Daily volume: 1M events/sec × 86,400 sec × 1KB = 86TB/day
With compression (5:1): 17TB/day
Monthly storage: 510TB
Yearly storage: 6PB (uncompressed), 1.2PB (compressed)

Network bandwidth:
- Peak: 1M events/sec × 1KB = 1GB/s
- Average: 500MB/s

Query load:
- 1000 engineers
- 10 queries/hour per engineer
- Total: 10,000 queries/day = 0.12 QPS (very low)
```

---

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Service Layer (100+ Services)                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │Service A │  │Service B │  │Service C │  │Service D │  │Service N │    │
│  │(Node.js) │  │(Python)  │  │(Java)    │  │(Go)      │  │(Rust)    │    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘    │
│       │ Log         │ Log         │ Log         │ Log         │ Log        │
│       │ Events      │ Events      │ Events      │ Events      │ Events     │
└───────┼─────────────┼─────────────┼─────────────┼─────────────┼────────────┘
        │             │             │             │             │
        └─────────────┴─────────────┴─────────────┴─────────────┘
                                    │
                                    v
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Log Collection Layer                                │
│                                                                              │
│  ┌──────────────────────┐       ┌──────────────────────┐                  │
│  │   Fluentd / Fluent   │       │      Logstash        │                  │
│  │   Bit (Lightweight)  │       │    (Heavier, more    │                  │
│  │                      │       │     features)        │                  │
│  │  - Log forwarding    │       │  - Log parsing       │                  │
│  │  - Buffering         │       │  - Enrichment        │                  │
│  │  - Protocol support  │       │  - Filtering         │                  │
│  └──────────┬───────────┘       └──────────┬───────────┘                  │
│             │                              │                               │
└─────────────┼──────────────────────────────┼───────────────────────────────┘
              │                              │
              v                              v
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Message Queue (Kafka)                               │
│                                                                              │
│  Topics:                                                                    │
│  - logs.raw          (all incoming logs)                                   │
│  - logs.parsed       (structured logs)                                     │
│  - logs.error        (error logs only)                                     │
│  - logs.audit        (audit logs)                                          │
│                                                                              │
│  Partitions: 100 (for parallelism)                                         │
│  Retention: 7 days (buffer)                                                │
└────────────────────────────┬────────────────────────────────────────────────┘
                             │
                             v
┌─────────────────────────────────────────────────────────────────────────────┐
│                      Log Processing Layer                                   │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────┐    │
│  │                   Stream Processor (Kafka Streams)                 │    │
│  │                                                                    │    │
│  │  1. Parse logs (JSON, syslog, custom formats)                     │    │
│  │  2. Extract metadata (timestamp, level, service, request_id)      │    │
│  │  3. Enrich with context (service metadata, environment)           │    │
│  │  4. Filter sensitive data (PII, credentials)                      │    │
│  │  5. Aggregate metrics (error rates, latencies)                    │    │
│  │  6. Detect anomalies (spike in errors)                            │    │
│  └───────────────────────────────────────────────────────────────────┘    │
└────────────────────────────┬────────────────────────────────────────────────┘
                             │
              ┌──────────────┼──────────────┬────────────────┐
              v              v              v                v
┌──────────────────┐  ┌──────────────┐  ┌────────────┐  ┌────────────┐
│  Elasticsearch   │  │   ClickHouse │  │     S3     │  │  Prometheus│
│  (Hot Storage)   │  │  (Analytics) │  │  (Archive) │  │  (Metrics) │
│                  │  │              │  │            │  │            │
│  - Last 7 days   │  │  - Aggregate │  │  - 1 year  │  │  - Error   │
│  - Real-time     │  │    queries   │  │    archive │  │    rates   │
│    search        │  │  - Fast      │  │  - Cold    │  │  - Latency │
│  - Full text     │  │    analytics │  │    storage │  │  - Volume  │
└────────┬─────────┘  └──────┬───────┘  └─────┬──────┘  └─────┬──────┘
         │                   │                │               │
         └───────────────────┴────────────────┴───────────────┘
                             │
                             v
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Visualization & Query Layer                              │
│                                                                              │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐            │
│  │    Kibana    │      │   Grafana    │      │    Custom    │            │
│  │              │      │              │      │   Dashboard  │            │
│  │  - Log search│      │  - Metrics   │      │              │            │
│  │  - Discovery │      │  - Dashboards│      │  - Business  │            │
│  │  - Dashboards│      │  - Alerting  │      │    metrics   │            │
│  └──────────────┘      └──────────────┘      └──────────────┘            │
└─────────────────────────────────────────────────────────────────────────────┘
                             │
                             v
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Alerting Layer                                     │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────┐    │
│  │                        Alert Manager                              │    │
│  │                                                                    │    │
│  │  Rules:                                                           │    │
│  │  - Error rate > 1% for 5 minutes → PagerDuty (P1)               │    │
│  │  - Service down → Slack, Email (P0)                              │    │
│  │  - High latency (P95 > 1s) → Slack (P2)                         │    │
│  │  - Disk usage > 80% → Email (P3)                                 │    │
│  │                                                                    │    │
│  │  Notification Channels:                                           │    │
│  │  - PagerDuty (critical)                                          │    │
│  │  - Slack (warnings)                                              │    │
│  │  - Email (info)                                                  │    │
│  └───────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Core Components

#### 1. Log Collection Agents

**Fluentd vs Fluent Bit vs Logstash:**

| Feature | Fluentd | Fluent Bit | Logstash |
|---------|---------|------------|----------|
| **Memory** | 40MB+ | 650KB | 200MB+ |
| **Performance** | 5K events/sec | 20K events/sec | 3K events/sec |
| **Language** | Ruby + C | C | Java (JRuby) |
| **Plugins** | 1000+ | 50+ | 200+ |
| **Use Case** | General | Embedded/Edge | Heavy processing |

**Recommendation:** Fluent Bit for log forwarding, Logstash for complex transformations

**Deployment Pattern:**
```
Kubernetes Pod:
┌─────────────────────────────┐
│  Application Container      │
│  - Writes logs to stdout    │
│  - Structured JSON format   │
└──────────┬──────────────────┘
           │
           v (sidecar)
┌─────────────────────────────┐
│  Fluent Bit Sidecar         │
│  - Reads container logs     │
│  - Adds pod metadata        │
│  - Forwards to Kafka        │
└─────────────────────────────┘
```

#### 2. Message Queue (Kafka)

**Why Kafka?**
- ✅ High throughput (millions of events/sec)
- ✅ Durable (replicated, persisted to disk)
- ✅ Decouples producers from consumers
- ✅ Replay capability (reprocess logs)
- ✅ Partitioning (parallel processing)

**Topic Configuration:**
```
Topic: logs.raw
Partitions: 100
Replication Factor: 3
Retention: 7 days (168 hours)
Compression: LZ4 (fast, good compression)

Partition Key: service_name
- Ensures logs from same service go to same partition
- Maintains ordering per service
```

**Producer Configuration:**
```
acks: 1 (leader acknowledgment)
- Balance between durability and performance
- acks=all is too slow for logs
- acks=0 risks losing logs

batch.size: 16KB
linger.ms: 10ms
- Batching improves throughput

compression.type: lz4
- 3-5x compression ratio
- Fast compression/decompression
```

#### 3. Stream Processing

**Processing Pipeline:**
```
1. Validation
   - Verify JSON structure
   - Check required fields (timestamp, level, message)
   - Discard malformed logs

2. Parsing
   - Extract structured fields
   - Parse timestamps (ISO 8601)
   - Parse log levels (INFO, WARN, ERROR)

3. Enrichment
   - Add service metadata (version, environment)
   - GeoIP lookup (if IP address present)
   - Add correlation IDs (if not present)

4. Filtering
   - Remove DEBUG logs in production
   - Filter out health check logs (reduce noise)
   - Redact sensitive fields (passwords, tokens, PII)

5. Routing
   - ERROR logs → logs.error topic
   - Audit logs → logs.audit topic
   - All logs → logs.parsed topic
```

**Example Log Transformation:**
```
Input (Raw):
{
  "timestamp": "2025-10-14T10:30:45.123Z",
  "level": "ERROR",
  "message": "Database connection failed",
  "service": "user-service",
  "error": {
    "message": "Connection timeout",
    "stack": "..."
  }
}

Output (Enriched):
{
  "timestamp": "2025-10-14T10:30:45.123Z",
  "level": "ERROR",
  "message": "Database connection failed",
  "service": {
    "name": "user-service",
    "version": "2.3.1",
    "environment": "production",
    "instance": "pod-user-service-7f9c6-abc123"
  },
  "error": {
    "message": "Connection timeout",
    "type": "DatabaseError",
    "stack": "..."
  },
  "metadata": {
    "request_id": "req_abc123",
    "user_id": "user_456",
    "ip": "192.168.1.100",
    "geo": {
      "country": "US",
      "city": "San Francisco"
    }
  },
  "indexed_at": "2025-10-14T10:30:45.500Z"
}
```

#### 4. Storage Layer

**Multi-Tier Storage Strategy:**

```
┌─────────────────────────────────────────────────────┐
│  Hot Tier (0-7 days) - Elasticsearch                │
│  - SSD storage                                      │
│  - Full indexing                                    │
│  - Real-time search (< 1 second)                   │
│  - Size: 10TB                                       │
│  - Cost: $$$                                        │
└─────────────────────────────────────────────────────┘
                      │
                      v (automated)
┌─────────────────────────────────────────────────────┐
│  Warm Tier (7-30 days) - Elasticsearch             │
│  - HDD storage                                      │
│  - Reduced replicas (1 instead of 2)               │
│  - Slower search (1-5 seconds)                     │
│  - Size: 30TB                                       │
│  - Cost: $$                                         │
└─────────────────────────────────────────────────────┘
                      │
                      v (automated)
┌─────────────────────────────────────────────────────┐
│  Cold Tier (30-365 days) - S3 Glacier              │
│  - Object storage                                   │
│  - Compressed (gzip)                                │
│  - No indexing                                      │
│  - Slow retrieval (minutes to hours)               │
│  - Size: 300TB                                      │
│  - Cost: $                                          │
└─────────────────────────────────────────────────────┘
                      │
                      v (policy)
┌─────────────────────────────────────────────────────┐
│  Deleted (> 365 days)                               │
│  - Compliance retention met                         │
│  - Permanently deleted                              │
└─────────────────────────────────────────────────────┘
```

**Elasticsearch Index Design:**
```
Index Naming: logs-{service}-{date}
Example: logs-user-service-2025-10-14

Benefits:
- Easy to delete old indices (rm logs-*-2024-*)
- Sharding per day (parallel searches)
- Rollover at midnight

Index Template:
{
  "index_patterns": ["logs-*"],
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1,
    "refresh_interval": "5s",
    "codec": "best_compression"
  },
  "mappings": {
    "properties": {
      "timestamp": { "type": "date" },
      "level": { "type": "keyword" },
      "message": { "type": "text" },
      "service": { "type": "keyword" },
      "request_id": { "type": "keyword" }
    }
  }
}
```

---

### Detailed Log Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    1. Log Generation                            │
│  Service: user-service                                          │
│  Code: logger.error("Database connection failed", {             │
│          request_id: "req_123",                                 │
│          user_id: "user_456"                                    │
│        });                                                      │
└──────────────────────────┬──────────────────────────────────────┘
                           │ stdout (container)
                           v
┌─────────────────────────────────────────────────────────────────┐
│                    2. Collection                                │
│  Fluent Bit Sidecar                                            │
│  - Reads from stdout                                           │
│  - Adds metadata:                                              │
│    * pod_name: "pod-user-service-7f9c6"                       │
│    * namespace: "production"                                   │
│    * node: "node-3"                                            │
│  - Timestamp: 2025-10-14T10:30:45.123Z                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTP POST
                           v
┌─────────────────────────────────────────────────────────────────┐
│                    3. Buffering                                 │
│  Kafka Topic: logs.raw                                         │
│  Partition: 42 (hash(user-service) % 100)                     │
│  - Durability: Replicated 3x                                   │
│  - Retention: 7 days                                           │
│  - Throughput: 1M events/sec                                   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           v
┌─────────────────────────────────────────────────────────────────┐
│                    4. Processing                                │
│  Kafka Streams Consumer                                         │
│  - Parse JSON                                                   │
│  - Validate required fields                                     │
│  - Enrich:                                                      │
│    * service_version: "2.3.1" (from service registry)         │
│    * geo_location: "US-CA" (from IP)                          │
│  - Filter:                                                      │
│    * Redact password field                                     │
│  - Route:                                                       │
│    * ERROR → logs.error (for alerting)                        │
│    * ALL → logs.parsed (for indexing)                         │
└──────────────────────────┬──────────────────────────────────────┘
                           │
              ┌────────────┴─────────────┐
              v                          v
┌──────────────────────┐      ┌──────────────────────┐
│  5a. Indexing        │      │  5b. Alerting        │
│  Elasticsearch       │      │  Alert Manager       │
│                      │      │                      │
│  Index:              │      │  Rule Match:         │
│  logs-user-service-  │      │  "error_rate > 1%"   │
│  2025-10-14          │      │                      │
│                      │      │  Action:             │
│  Document ID:        │      │  - Send PagerDuty    │
│  req_123_timestamp   │      │  - Post to Slack     │
│                      │      │                      │
│  Searchable in:      │      │  Notification sent!  │
│  < 1 second          │      │                      │
└──────────────────────┘      └──────────────────────┘
              │
              v
┌─────────────────────────────────────────────────────────────────┐
│                    6. Querying                                  │
│  Engineer searches in Kibana:                                   │
│  "level:ERROR AND service:user-service AND timestamp:[         │
│   2025-10-14T10:00:00 TO 2025-10-14T11:00:00]"                │
│                                                                 │
│  Results:                                                       │
│  - 47 matching logs                                            │
│  - Response time: 234ms                                        │
│  - Aggregations: Error types, Affected users                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### Scaling Strategies

#### 1. Horizontal Scaling

**Collection Layer:**
```
Fluent Bit per Pod:
- No central bottleneck
- Scales with application
- CPU: 0.1 core, Memory: 100MB per pod

Total:
- 1000 pods × 100MB = 100GB memory
- Marginal resource overhead
```

**Kafka Cluster:**
```
Brokers: 10 (for redundancy)
Partitions: 100 per topic
Target: 100K events/sec per broker
Total capacity: 1M events/sec

Scaling:
- Add brokers for more throughput
- Increase partitions for more parallelism
- Rebalance automatically
```

**Processing Layer:**
```
Kafka Streams Instances: 20
Each instance:
- Consumes from 5 partitions
- Processes 50K events/sec
- Auto-scaling based on lag

Scaling trigger:
- Consumer lag > 1 million messages
- Scale up by 5 instances
- Scale down if lag < 100K for 10 minutes
```

**Elasticsearch Cluster:**
```
Data Nodes:
- Hot tier: 20 nodes (SSD)
- Warm tier: 30 nodes (HDD)

Index per day:
- logs-*-2025-10-14: 1TB
- Sharded across 20 nodes (50GB per node)

Scaling:
- Add nodes as data grows
- Rebalance shards automatically
- Monitor heap usage (< 75%)
```

#### 2. Performance Optimization

**Batching:**
```
Collection:
- Batch 1000 logs before sending to Kafka
- Max latency: 1 second

Processing:
- Batch 500 logs before indexing to Elasticsearch
- Bulk API for efficiency

Benefit:
- 10x throughput improvement
- Reduced network overhead
```

**Compression:**
```
Kafka:
- LZ4 compression (4:1 ratio)
- 1TB/day → 250GB/day on disk

Elasticsearch:
- best_compression codec (5:1 ratio)
- 10TB → 2TB on disk

S3:
- gzip compression (10:1 ratio)
- 100TB → 10TB archived
```

**Sampling (for high-volume services):**
```
Debug Logs:
- Sample 10% in production
- 100% in dev/staging

Info Logs:
- Sample 50% for high-traffic endpoints
- 100% for critical paths

Error Logs:
- Always 100% (never sample errors)

Trade-off:
- Reduced volume (90% reduction possible)
- Slight loss of visibility (acceptable for DEBUG)
```

#### 3. Cost Optimization

**Storage Cost Breakdown:**
```
Elasticsearch Hot (7 days):
- 10TB × $0.10/GB/month = $1,000/month

Elasticsearch Warm (30 days):
- 30TB × $0.03/GB/month = $900/month

S3 Glacier (365 days):
- 300TB × $0.004/GB/month = $1,200/month

Total: $3,100/month for 1 year retention

Optimization:
- Compress aggressively: Save 50% → $1,550/month
- Sample DEBUG logs: Reduce by 40% → $930/month
- Shorter hot tier (3 days): Save 30% → $650/month

Optimized total: $650/month (80% savings)
```

---

### Trade-Offs and Alternatives

#### Architecture Decisions

**1. Why Kafka over Direct Elasticsearch Ingestion?**

**With Kafka (Chosen):**
```
Services → Kafka → Processors → Elasticsearch
```
- ✅ Buffering (handles spikes)
- ✅ Decoupling (Elasticsearch downtime OK)
- ✅ Replay capability (reprocess logs)
- ✅ Multiple consumers (ES + S3 + analytics)
- ❌ Added complexity
- ❌ Extra cost (Kafka infrastructure)

**Without Kafka:**
```
Services → Elasticsearch
```
- ✅ Simpler architecture
- ✅ Lower latency
- ❌ No buffering (lost logs during spikes)
- ❌ Elasticsearch becomes bottleneck
- ❌ No replay

**Decision:** Kafka provides reliability and flexibility worth the complexity.

---

**2. Why Elasticsearch over ClickHouse or Loki?**

| Feature | Elasticsearch | ClickHouse | Grafana Loki |
|---------|---------------|------------|--------------|
| **Full-Text Search** | Excellent | Limited | Limited |
| **Query Performance** | Fast | Very Fast | Fast |
| **Cost** | $$$ | $$ | $ |
| **Scalability** | Excellent | Excellent | Good |
| **Compression** | Good (5:1) | Excellent (10:1) | Excellent (10:1) |
| **Use Case** | General logs | Analytics | Metrics + logs |

**Decision:**
- Use Elasticsearch for log search (strong full-text search)
- Use ClickHouse for analytics (aggregations, metrics)
- Use Prometheus for metrics (time-series)

Hybrid approach leverages strengths of each system.

---

**3. Why Structured Logging (JSON) over Plain Text?**

**Structured (JSON):**
```json
{
  "timestamp": "2025-10-14T10:30:45.123Z",
  "level": "ERROR",
  "service": "user-service",
  "request_id": "req_123",
  "message": "Database connection failed"
}
```
- ✅ Easy to parse and query
- ✅ Searchable fields
- ✅ Consistent structure
- ❌ Slightly larger size (~20% overhead)

**Plain Text:**
```
[2025-10-14 10:30:45] ERROR user-service: Database connection failed (req_123)
```
- ✅ Human-readable
- ✅ Smaller size
- ❌ Hard to parse (regex needed)
- ❌ Inconsistent formats across services

**Decision:** Structured logging is essential for large-scale systems.

---

### Advanced Features

#### 1. Log Correlation (Distributed Tracing)

**Problem:** How to trace a request across 10+ microservices?

**Solution: Correlation IDs**

```
Request Flow:
┌─────────────┐ request_id: req_123
│   Client    ├─────────────────────────────────┐
└─────────────┘                                 v
                                    ┌─────────────────────┐
                                    │  API Gateway        │
                                    │  request_id: req_123│
                                    └──────────┬──────────┘
                                               │
                    ┌──────────────────────────┼──────────────────────────┐
                    v                          v                          v
        ┌─────────────────────┐   ┌─────────────────────┐   ┌─────────────────────┐
        │  Auth Service       │   │  User Service       │   │  Order Service      │
        │  request_id: req_123│   │  request_id: req_123│   │  request_id: req_123│
        └─────────────────────┘   └─────────────────────┘   └─────────────────────┘

All logs tagged with request_id: req_123

Query: Search all logs with request_id=req_123
Result: Complete trace across all services
```

**Log Example:**
```json
{
  "timestamp": "2025-10-14T10:30:45.123Z",
  "level": "INFO",
  "service": "auth-service",
  "request_id": "req_123",  // Propagated through all services
  "span_id": "span_456",     // Unique per service
  "parent_span_id": "span_000",
  "message": "User authenticated successfully"
}
```

#### 2. Anomaly Detection

**Statistical Anomaly Detection:**
```
Baseline (last 24 hours):
- Average error rate: 0.5%
- Std deviation: 0.1%

Current (5-minute window):
- Error rate: 3.2%

Calculation:
- Z-score = (3.2 - 0.5) / 0.1 = 27

Alert Threshold: Z-score > 3 (3 standard deviations)
→ Alert triggered! (27 >> 3)

Action:
- Send PagerDuty alert
- Create incident ticket
- Notify on-call engineer
```

**ML-Based Anomaly Detection:**
```
Training:
- Collect 30 days of normal logs
- Train model on patterns:
  * Error rates by hour
  * Request patterns
  * Latency distributions

Detection:
- Real-time scoring of log patterns
- Detect deviations from learned patterns
- Alert on anomalies

Benefits:
- Detects unknown issues
- Adapts to changing patterns
- Reduces false positives
```

#### 3. Log Sampling Strategies

**Smart Sampling:**
```
Rules:
1. Always log:
   - ERROR, FATAL levels
   - Audit events
   - Security events

2. Sample 10%:
   - DEBUG logs in production
   - Health check logs

3. Sample 50%:
   - INFO logs from high-traffic endpoints

4. Sample 100%:
   - New services (< 7 days old)
   - Critical services
   - Services with recent incidents

Implementation:
if (level === 'ERROR' || isAuditLog()) {
  log();  // Always log
} else if (level === 'DEBUG') {
  if (Math.random() < 0.1) log();  // 10% sampling
} else if (isHighTrafficEndpoint()) {
  if (Math.random() < 0.5) log();  // 50% sampling
} else {
  log();  // Default: log everything
}
```

---

### Monitoring & Alerting

**Key Metrics:**
```
1. Pipeline Health:
   - Log ingestion rate (events/sec)
   - Processing lag (seconds behind real-time)
   - Drop rate (lost logs)
   - Kafka consumer lag

2. Storage Health:
   - Elasticsearch cluster health (green/yellow/red)
   - Disk usage (%)
   - Index size growth rate
   - Query latency

3. Service Health:
   - Error rate by service
   - Top error messages
   - Services with increasing errors
   - Services not logging (silent failures)

4. Cost Metrics:
   - Storage cost per service
   - Most verbose services (GB/day)
   - Retention compliance
```

**Alert Rules:**
```
P0 (Critical - Page immediately):
- Logging pipeline down (no logs for 5 minutes)
- Elasticsearch cluster red

P1 (High - Page during business hours):
- Service error rate > 5%
- Processing lag > 10 minutes
- Disk usage > 90%

P2 (Medium - Slack notification):
- Service error rate > 1%
- Unusual error patterns detected
- Query latency > 5 seconds

P3 (Low - Email):
- New error type detected
- Service log volume increased 50%
```

---

### Interview Discussion Points

**Common Questions:**

**Q: How do you prevent log loss?**
- ✅ Kafka replication (3x)
- ✅ At-least-once delivery semantics
- ✅ Local buffering in log agents
- ✅ Dead letter queue for failed logs
- ✅ Monitor ingestion rate vs expected rate

**Q: How do you handle sensitive data in logs?**
- ✅ Automatic PII detection and redaction
- ✅ Allowlist of safe fields
- ✅ Encryption at rest and in transit
- ✅ Access controls (RBAC)
- ✅ Audit logging of log access

**Q: How do you handle log schema changes?**
- ✅ Backward-compatible changes only
- ✅ Version field in logs
- ✅ Dynamic field mapping in Elasticsearch
- ✅ Gradual rollout of changes
- ✅ Test in staging first

**Q: What if Kafka is full?**
- ✅ Increase retention period temporarily
- ✅ Add more disk to brokers
- ✅ Delete old topics (after archiving)
- ✅ Back-pressure to services (slow down logging)
- ✅ Emergency sampling (drop DEBUG logs)

**Q: How do you search across 1 year of logs?**
- ✅ Most searches are recent (7 days) - use hot tier
- ✅ For old logs: restore from S3 to warm tier
- ✅ Use date filters to limit search scope
- ✅ Pre-aggregate common queries
- ✅ Consider using ClickHouse for historical analytics

---

## Conclusion

Both file search and logging systems require careful consideration of:
- **Scalability:** Handle growth in data and users
- **Performance:** Fast queries and low latency
- **Cost:** Balance performance with cost
- **Reliability:** No data loss, high availability
- **Observability:** Monitor and alert on issues

**Key Takeaways:**
1. **Use the right tool for the job** - Elasticsearch for search, Kafka for buffering
2. **Plan for scale early** - Sharding, partitioning, tiered storage
3. **Monitor everything** - You can't fix what you can't measure
4. **Consider trade-offs** - Every design decision has pros and cons
5. **Start simple, evolve** - Don't over-engineer initially

These designs are battle-tested in production systems handling billions of logs and files daily. Use them as templates and adapt to your specific requirements!
