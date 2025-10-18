# System Design Interview Guide

This comprehensive guide covers common system design interview questions with detailed architectures, component explanations, trade-offs, and scaling strategies.

---

## Table of Contents

1. [Chat Application (WhatsApp/Slack)](#1-chat-application-whatsappslack)
2. [File Hosting Service (Dropbox/Google Drive)](#2-file-hosting-service-dropboxgoogle-drive)
3. [URL Shortener (bit.ly)](#3-url-shortener-bitly)
4. [Social Media Feed (Twitter/Instagram)](#4-social-media-feed-twitterinstagram)
5. [Video Streaming Platform (YouTube/Netflix)](#5-video-streaming-platform-youtubenetflix)
6. [E-commerce Platform (Amazon)](#6-e-commerce-platform-amazon)
7. [Notification System](#7-notification-system)
8. [Rate Limiter](#8-rate-limiter)
9. [Design Patterns Summary](#9-design-patterns-summary)
10. [Key Concepts & Glossary](#10-key-concepts--glossary)

---

## 1. Chat Application (WhatsApp/Slack)

### Overview
Design a real-time messaging system supporting one-on-one and group chats with features like online status, message delivery confirmation, and file sharing.

### Functional Requirements
- Send/receive messages in real-time
- One-on-one and group chats
- Online/offline status
- Message delivery status (sent, delivered, read)
- Message history
- File/image sharing
- Push notifications

### Non-Functional Requirements
- Low latency (< 200ms for message delivery)
- High availability (99.99%)
- Scalable to billions of messages/day
- End-to-end encryption (optional)

---

### Architecture Approach 1: WebSocket-Based (Recommended for Real-Time)

```
┌─────────────┐
│   Clients   │ (Mobile/Web)
│ (WebSocket) │
└──────┬──────┘
       │
       v
┌─────────────────────────────────────┐
│      Load Balancer (HAProxy)        │
│  (Sticky sessions for WebSockets)   │
└──────────────┬──────────────────────┘
               │
       ┌───────┴────────┐
       v                v
┌──────────────┐  ┌──────────────┐
│   Chat       │  │   Chat       │
│   Server 1   │  │   Server 2   │
│ (WebSocket)  │  │ (WebSocket)  │
└──────┬───────┘  └───────┬──────┘
       │                  │
       └──────────┬───────┘
                  v
         ┌─────────────────┐
         │  Message Queue  │
         │  (Kafka/RabbitMQ│
         └────────┬────────┘
                  │
       ┌──────────┼──────────┐
       v          v          v
┌─────────┐ ┌─────────┐ ┌──────────┐
│ Message │ │ Presence│ │ Push     │
│ Service │ │ Service │ │ Notif.   │
└────┬────┘ └────┬────┘ └────┬─────┘
     │           │           │
     v           v           v
┌──────────┐ ┌─────────┐ ┌─────────┐
│ Message  │ │  Redis  │ │  APNs   │
│ Database │ │ (Cache) │ │  FCM    │
│(Cassandra│ │         │ │         │
└──────────┘ └─────────┘ └─────────┘
```

#### Components Explained

**1. Chat Server (WebSocket Server)**
- Maintains persistent WebSocket connections with clients
- Routes messages between users
- Handles connection management (connect, disconnect, heartbeat)
- Technologies: Node.js + Socket.io, Go + Gorilla WebSocket

**2. Load Balancer with Sticky Sessions**
- Routes clients to specific chat servers
- Maintains sticky sessions (same user → same server)
- Health checks for server availability
- Technologies: HAProxy, Nginx, AWS ALB

**3. Message Queue (Kafka/RabbitMQ)**
- Decouples message ingestion from processing
- Guarantees message delivery
- Enables async processing
- Provides message ordering within partitions

**4. Message Service**
- Stores messages in database
- Handles message persistence
- Manages message history and search
- Implements message retention policies

**5. Presence Service**
- Tracks online/offline status
- Uses Redis for fast status checks
- Publishes status updates to subscribers
- Implements heartbeat mechanism (ping every 30s)

**6. Push Notification Service**
- Sends notifications to offline users
- Integrates with APNs (iOS) and FCM (Android)
- Batches notifications for efficiency
- Handles notification preferences

**7. Databases**
- **Message Database (Cassandra/MongoDB):**
  - Stores message history
  - Optimized for write-heavy workload
  - Distributed and horizontally scalable
  - Schema:
    ```sql
    messages (
      message_id UUID PRIMARY KEY,
      conversation_id UUID,
      sender_id UUID,
      content TEXT,
      timestamp TIMESTAMP,
      status ENUM('sent', 'delivered', 'read')
    )
    INDEX ON (conversation_id, timestamp)
    ```

- **User Database (PostgreSQL/MySQL):**
  - Stores user profiles
  - Manages contacts and relationships
  - Schema:
    ```sql
    users (
      user_id UUID PRIMARY KEY,
      username VARCHAR(50),
      phone_number VARCHAR(20),
      last_seen TIMESTAMP
    )
    ```

**8. Redis Cache**
- Caches user online status
- Stores recent messages for fast retrieval
- Manages chat server connections (userId → serverId mapping)
- Implements distributed locks

---

### Architecture Approach 2: HTTP Long Polling (Fallback)

```
┌─────────────┐
│   Clients   │
│ (HTTP Poll) │
└──────┬──────┘
       │ GET /messages?since=timestamp
       v
┌──────────────┐
│   API        │
│   Gateway    │
└──────┬───────┘
       │
       v
┌──────────────┐
│   Message    │
│   Service    │
└──────┬───────┘
       │
       v
┌──────────────┐
│   Database   │
└──────────────┘
```

#### How Long Polling Works
1. Client sends HTTP request: `GET /messages?userId=123&since=timestamp`
2. Server holds connection open (30-60 seconds)
3. If new message arrives, server responds immediately
4. If timeout, server responds with empty result
5. Client immediately sends new request

---

### Message Flow (Real-Time Messaging)

#### Scenario: User A sends message to User B

```
User A                Chat Server 1           Message Queue        Message Service       Chat Server 2        User B
  │                         │                       │                     │                     │               │
  │ 1. Send Message         │                       │                     │                     │               │
  ├────────────────────────>│                       │                     │                     │               │
  │                         │                       │                     │                     │               │
  │                         │ 2. Publish to Queue   │                     │                     │               │
  │                         ├──────────────────────>│                     │                     │               │
  │                         │                       │                     │                     │               │
  │ 3. Ack (Message Sent)   │                       │                     │                     │               │
  │<────────────────────────┤                       │                     │                     │               │
  │                         │                       │                     │                     │               │
  │                         │                       │ 4. Consume Message  │                     │               │
  │                         │                       ├────────────────────>│                     │               │
  │                         │                       │                     │                     │               │
  │                         │                       │                     │ 5. Save to DB       │               │
  │                         │                       │                     ├─────────────┐       │               │
  │                         │                       │                     │             │       │               │
  │                         │                       │                     │<────────────┘       │               │
  │                         │                       │                     │                     │               │
  │                         │                       │                     │ 6. Lookup User B    │               │
  │                         │                       │                     │    Server (Redis)   │               │
  │                         │                       │                     ├─────────────────────┤               │
  │                         │                       │                     │                     │               │
  │                         │                       │                     │ 7. Forward Message  │               │
  │                         │                       │                     ├────────────────────>│               │
  │                         │                       │                     │                     │               │
  │                         │                       │                     │                     │ 8. Deliver    │
  │                         │                       │                     │                     ├──────────────>│
  │                         │                       │                     │                     │               │
  │                         │                       │                     │                     │ 9. Read Receipt│
  │                         │                       │                     │                     │<───────────────│
  │                         │                       │                     │<────────────────────┤               │
  │ 10. Update Status       │                       │                     │                     │               │
  │     (Delivered)         │                       │                     │                     │               │
  │<────────────────────────┤                       │                     │                     │               │
```

---

### Data Models

#### Message Table (Cassandra)
```
CREATE TABLE messages (
    conversation_id UUID,
    message_id UUID,
    sender_id UUID,
    receiver_id UUID,
    content TEXT,
    media_url TEXT,
    created_at TIMESTAMP,
    status TEXT,  -- 'sent', 'delivered', 'read'
    PRIMARY KEY (conversation_id, created_at, message_id)
) WITH CLUSTERING ORDER BY (created_at DESC);

-- Index for fast lookups
CREATE INDEX ON messages (message_id);
```

#### Conversation Table
```
CREATE TABLE conversations (
    conversation_id UUID PRIMARY KEY,
    type TEXT,  -- 'one_on_one', 'group'
    participants SET<UUID>,
    created_at TIMESTAMP,
    last_message_at TIMESTAMP
);
```

#### User Presence (Redis)
```
Key: user:{user_id}:status
Value: {
  "online": true,
  "last_seen": "2025-10-13T10:30:00Z",
  "server_id": "chat-server-2"
}
TTL: 60 seconds (refreshed by heartbeat)
```

---

### Scaling Strategies

**1. Horizontal Scaling of Chat Servers**
- Add more WebSocket servers behind load balancer
- Use consistent hashing for user-to-server mapping
- Challenge: Users on different servers need message routing

**2. Database Sharding**
- **Shard by conversation_id:** All messages in a conversation on same shard
- **Shard by user_id:** User's data on specific shard
- **Hybrid:** Shard conversations, replicate user data

**3. Caching Strategy**
- Cache last 100 messages per conversation in Redis
- Cache user online status (TTL: 60s)
- Cache user profile data
- Invalidation: On new message or status change

**4. Message Queue Partitioning**
- Partition Kafka topics by conversation_id
- Guarantees message ordering within conversation
- Allows parallel processing across conversations

---

### Pros and Cons

#### WebSocket Approach

**Pros:**
- ✅ True real-time communication (< 50ms latency)
- ✅ Bi-directional communication
- ✅ Efficient for high-frequency updates
- ✅ Lower overhead than HTTP polling

**Cons:**
- ❌ Complex to scale (stateful connections)
- ❌ Requires load balancer with sticky sessions
- ❌ Connection management overhead
- ❌ Firewall/proxy compatibility issues

#### Long Polling Approach

**Pros:**
- ✅ Works everywhere (standard HTTP)
- ✅ Simpler to implement
- ✅ Stateless (easier to scale)
- ✅ No special load balancer needed

**Cons:**
- ❌ Higher latency (1-2 seconds)
- ❌ More server resources (held connections)
- ❌ More client-side complexity
- ❌ Not suitable for high-frequency updates

---

### Advanced Features

**1. End-to-End Encryption**
- Client-side encryption using public/private keys
- Server stores encrypted messages (can't read content)
- Key exchange using Signal Protocol or similar

**2. Message Search**
- Use Elasticsearch for full-text search
- Index messages asynchronously from Kafka
- Support filters: sender, date range, media type

**3. File Sharing**
- Upload files to object storage (S3/CloudFront)
- Generate pre-signed URLs for access
- Store only metadata in message database

**4. Group Chat Optimization**
- Fan-out on write: Send to all members immediately
- Fan-out on read: Store once, retrieve per user
- Trade-off: Write speed vs storage efficiency

---

## 2. File Hosting Service (Dropbox/Google Drive)

### Overview
Design a cloud storage system that allows users to upload, download, sync, and share files across multiple devices.

### Functional Requirements
- Upload/download files
- File synchronization across devices
- File sharing (public/private links)
- File versioning
- Folder structure
- Search files by name/content

### Non-Functional Requirements
- High availability (99.99%)
- High durability (99.999999999% - 11 nines)
- Support large files (up to 10GB)
- Fast sync (< 1 second for small changes)
- Scalable to billions of files

---

### Architecture Approach 1: Chunked Upload with Sync (Recommended)

```
┌──────────────────┐
│  Client App      │
│  (Desktop/Mobile)│
└────────┬─────────┘
         │
    ┌────┴────┐
    │         │
    v         v
┌─────────┐ ┌─────────┐
│  Block  │ │   Sync  │
│ Service │ │ Service │
└────┬────┘ └────┬────┘
     │           │
     v           v
┌──────────┐  ┌──────────┐
│  Object  │  │ Metadata │
│ Storage  │  │    DB    │
│  (S3)    │  │(Postgres)│
└──────────┘  └──────────┘
         │
         v
    ┌─────────┐
    │   CDN   │
    │(CloudFr)│
    └─────────┘
```

#### Components Explained

**1. Block Service**
- Chunks files into fixed-size blocks (4MB)
- Computes hash for each block (SHA-256)
- Uploads only changed blocks (deduplication)
- Handles compression
- Technologies: Go, Java

**2. Sync Service**
- Detects file changes on client
- Compares file versions
- Orchestrates block uploads/downloads
- Manages conflict resolution
- Maintains sync queue

**3. Metadata Database**
- Stores file metadata (name, size, owner, permissions)
- Maintains folder hierarchy
- Tracks file versions
- Manages sharing permissions
- Database: PostgreSQL with partitioning

**4. Object Storage (S3)**
- Stores actual file blocks
- High durability (11 nines)
- Automatic replication across regions
- Cost-effective for large data

**5. CDN (CloudFront)**
- Caches frequently accessed files
- Reduces latency for downloads
- Serves shared public files
- Handles geographic distribution

---

### Architecture Approach 2: Direct Upload (Simple)

```
┌──────────────┐
│   Client     │
└──────┬───────┘
       │
       v
┌──────────────┐
│   API        │
│   Gateway    │
└──────┬───────┘
       │
   ┌───┴────┐
   v        v
┌────────┐ ┌──────────┐
│ Upload │ │ Metadata │
│Service │ │  Service │
└───┬────┘ └─────┬────┘
    │            │
    v            v
┌────────┐   ┌────────┐
│   S3   │   │   DB   │
└────────┘   └────────┘
```

---

### File Upload Flow (Chunked)

```
Client              Block Service         Metadata DB          Object Storage
  │                      │                     │                      │
  │ 1. Initiate Upload   │                     │                      │
  ├─────────────────────>│                     │                      │
  │                      │                     │                      │
  │                      │ 2. Check Existing   │                      │
  │                      │    Blocks (Hash)    │                      │
  │                      ├────────────────────>│                      │
  │                      │                     │                      │
  │                      │ 3. Return Missing   │                      │
  │                      │    Block List       │                      │
  │                      │<────────────────────┤                      │
  │                      │                     │                      │
  │ 4. Upload Missing    │                     │                      │
  │    Blocks (Parallel) │                     │                      │
  ├─────────────────────>│                     │                      │
  │                      │                     │                      │
  │                      │ 5. Store Blocks     │                      │
  │                      ├──────────────────────────────────────────>│
  │                      │                     │                      │
  │                      │ 6. Update Metadata  │                      │
  │                      ├────────────────────>│                      │
  │                      │                     │                      │
  │ 7. Upload Complete   │                     │                      │
  │<─────────────────────┤                     │                      │
```

**Key Optimization: Block Deduplication**
- If a block already exists (same hash), skip upload
- Saves bandwidth and storage
- Example: Uploading a file with 1 changed byte only uploads 1 block (4MB) instead of entire 1GB file

---

### Sync Algorithm

**Problem:** How to keep files in sync across multiple devices?

**Solution: Change Detection + Merkle Trees**

#### 1. File Watcher (Client-Side)
```
Monitor file system events:
- File created
- File modified
- File deleted
- File moved
```

#### 2. Merkle Tree for Fast Comparison
```
Root Hash: ABC123
    │
    ├─── Folder: /Documents (Hash: DEF456)
    │       ├─── file1.txt (Hash: GHI789)
    │       └─── file2.pdf (Hash: JKL012)
    │
    └─── Folder: /Photos (Hash: MNO345)
            ├─── img1.jpg (Hash: PQR678)
            └─── img2.png (Hash: STU901)
```

**Sync Process:**
1. Client computes Merkle tree of local files
2. Sends root hash to server
3. Server compares with stored root hash
4. If different, traverse tree to find changed nodes
5. Only sync changed files/folders

---

### Data Models

#### Files Table
```sql
CREATE TABLE files (
    file_id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    parent_folder_id UUID,  -- NULL for root
    name VARCHAR(255) NOT NULL,
    size_bytes BIGINT,
    content_type VARCHAR(100),
    is_deleted BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    current_version INT DEFAULT 1,

    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (parent_folder_id) REFERENCES files(file_id)
);

CREATE INDEX idx_files_user_parent ON files(user_id, parent_folder_id);
CREATE INDEX idx_files_name ON files(name);
```

#### File Blocks Table
```sql
CREATE TABLE file_blocks (
    block_id UUID PRIMARY KEY,
    block_hash VARCHAR(64) UNIQUE NOT NULL,  -- SHA-256
    size_bytes INT,
    storage_path TEXT,  -- S3 key
    created_at TIMESTAMP
);

-- Junction table: which blocks belong to which file version
CREATE TABLE file_version_blocks (
    file_version_id UUID,
    block_id UUID,
    block_order INT,  -- Order of blocks in file
    PRIMARY KEY (file_version_id, block_order),
    FOREIGN KEY (block_id) REFERENCES file_blocks(block_id)
);
```

#### File Versions Table
```sql
CREATE TABLE file_versions (
    version_id UUID PRIMARY KEY,
    file_id UUID NOT NULL,
    version_number INT,
    size_bytes BIGINT,
    created_at TIMESTAMP,
    created_by UUID,  -- user who created this version

    FOREIGN KEY (file_id) REFERENCES files(file_id)
);

CREATE INDEX idx_versions_file ON file_versions(file_id, version_number);
```

#### Sharing Table
```sql
CREATE TABLE file_shares (
    share_id UUID PRIMARY KEY,
    file_id UUID NOT NULL,
    shared_by UUID,
    shared_with UUID,  -- NULL for public links
    permission ENUM('view', 'edit'),
    share_link VARCHAR(255) UNIQUE,  -- For public links
    expires_at TIMESTAMP,
    created_at TIMESTAMP,

    FOREIGN KEY (file_id) REFERENCES files(file_id)
);

CREATE INDEX idx_shares_file ON file_shares(file_id);
CREATE INDEX idx_shares_link ON file_shares(share_link);
```

---

### Scaling Strategies

**1. Block Storage Optimization**
- **Deduplication:** Store each unique block once (identified by hash)
- **Compression:** Compress blocks before storage (gzip, brotli)
- **Cold storage:** Move old versions to Glacier (99% cost reduction)

**2. Database Sharding**
- **Shard by user_id:** Each user's data on specific shard
- **Challenge:** Shared files span multiple shards
- **Solution:** Replicate sharing metadata across shards

**3. Caching Strategy**
- **Metadata Cache (Redis):**
  - Cache folder contents
  - Cache file metadata
  - TTL: 5 minutes
- **Block Cache (CDN):**
  - Cache frequently accessed blocks
  - Cache public shared files
  - TTL: 1 hour

**4. Read Optimization**
- Pre-signed URLs for direct S3 access
- Parallel block downloads (4-8 concurrent)
- Lazy loading (download on access, not full sync)

---

### Conflict Resolution

**Problem:** User edits same file on 2 devices while offline

**Solutions:**

#### 1. Last Write Wins (Simple)
- Use timestamp to determine winner
- Loses conflicting changes
- Simple but data loss possible

#### 2. Operational Transformation (OT)
- Used by Google Docs
- Transforms concurrent operations to be compatible
- Complex but no data loss
- Example:
  ```
  User A: Insert "X" at position 5
  User B: Insert "Y" at position 5

  Transform to:
  User A: Insert "X" at position 5
  User B: Insert "Y" at position 6  (adjusted)
  ```

#### 3. Conflict Files (Dropbox Approach)
- Keep both versions
- Rename conflicting file: `document (conflicted copy).txt`
- User manually resolves
- Simple and safe

---

### Pros and Cons

#### Chunked Upload Approach

**Pros:**
- ✅ Efficient sync (only changed blocks)
- ✅ Resume interrupted uploads
- ✅ Deduplication saves storage
- ✅ Fast for large files

**Cons:**
- ❌ Complex implementation
- ❌ Higher computational cost (hashing)
- ❌ More database operations
- ❌ Chunking overhead for small files

#### Direct Upload Approach

**Pros:**
- ✅ Simple to implement
- ✅ Fast for small files
- ✅ Less server computation
- ✅ Easier to debug

**Cons:**
- ❌ Inefficient for large files
- ❌ No deduplication
- ❌ Must re-upload entire file for changes
- ❌ Cannot resume uploads

---

### Advanced Features

**1. Real-Time Collaboration**
- Use WebSockets for real-time updates
- Operational Transformation for concurrent edits
- Presence awareness (who's viewing/editing)

**2. Full-Text Search**
- Extract text from files (OCR for images)
- Index in Elasticsearch
- Support filters: file type, size, date

**3. File Preview Generation**
- Generate thumbnails for images/videos
- Convert documents to PDF
- Store previews in CDN
- Async processing via queue

**4. Bandwidth Optimization**
- Delta sync (only send changed bytes)
- Compression
- P2P sync between user's devices (BitTorrent protocol)

---

## 3. URL Shortener (bit.ly)

### Overview
Design a service that creates short aliases for long URLs, tracks analytics, and handles billions of redirects per day.

### Functional Requirements
- Generate short URL from long URL
- Redirect short URL to original URL
- Custom aliases (optional)
- Expiration dates
- Analytics (click count, geography, devices)

### Non-Functional Requirements
- Low latency (< 50ms for redirect)
- High availability (99.99%)
- Scalable to billions of URLs
- Short URLs should be unpredictable

---

### Architecture

```
┌─────────────┐
│   Clients   │
└──────┬──────┘
       │
       v
┌──────────────┐
│ Load Balancer│
└──────┬───────┘
       │
   ┌───┴────┐
   v        v
┌────────┐ ┌────────┐
│  API   │ │ Redir  │
│Service │ │Service │
└───┬────┘ └────┬───┘
    │           │
    └─────┬─────┘
          v
    ┌──────────┐
    │  Redis   │
    │  (Cache) │
    └────┬─────┘
         │
         v
    ┌──────────┐
    │ Database │
    │(Postgres)│
    └────┬─────┘
         │
         v
    ┌──────────┐
    │Analytics │
    │  Queue   │
    │ (Kafka)  │
    └──────────┘
```

---

### Short URL Generation

**Problem:** Generate unique, short, unpredictable identifiers

#### Approach 1: Base62 Encoding (Recommended)

**Algorithm:**
```
1. Generate unique ID (e.g., auto-increment DB ID or UUID)
2. Encode ID to Base62 (0-9, a-z, A-Z)
3. Result: 7-character short URL

Example:
ID: 123456789
Base62: aZl8xV
Short URL: https://short.ly/aZl8xV
```

**Character Set:**
```
Base62 = [0-9a-zA-Z]
Length = 7 characters
Capacity = 62^7 = 3.5 trillion unique URLs
```

**Implementation:**
```javascript
const BASE62 = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';

function encodeBase62(num) {
  if (num === 0) return BASE62[0];

  let encoded = '';
  while (num > 0) {
    encoded = BASE62[num % 62] + encoded;
    num = Math.floor(num / 62);
  }
  return encoded;
}

function decodeBase62(str) {
  let decoded = 0;
  for (let i = 0; i < str.length; i++) {
    decoded = decoded * 62 + BASE62.indexOf(str[i]);
  }
  return decoded;
}

// Example
const id = 123456789;
const shortCode = encodeBase62(id);  // "aZl8xV"
const original = decodeBase62(shortCode);  // 123456789
```

**Pros:**
- ✅ Deterministic (same ID → same short URL)
- ✅ No collisions
- ✅ Fast encoding/decoding
- ✅ Reversible (decode to get ID)

**Cons:**
- ❌ Sequential (predictable if using auto-increment)
- ❌ Exposes total number of URLs

---

#### Approach 2: Hash-Based (MD5/SHA)

**Algorithm:**
```
1. Hash long URL using MD5
2. Take first 7 characters
3. Check for collision in database
4. If collision, add salt and rehash

Example:
URL: https://example.com/very/long/url
MD5: d131dd02c5e6eec4693d9a0698aff95c
Short: d131dd0
```

**Pros:**
- ✅ Same URL → same short URL (idempotent)
- ✅ Unpredictable
- ✅ No sequential pattern

**Cons:**
- ❌ Collision possible (birthday paradox)
- ❌ Requires collision handling
- ❌ More database checks

---

#### Approach 3: Key Generation Service (KGS)

**Architecture:**
```
┌──────────────┐
│  API Service │
└──────┬───────┘
       │
       v
┌──────────────┐         ┌──────────────┐
│     KGS      │────────>│  Pre-gen     │
│  (Service)   │         │  Keys DB     │
└──────────────┘         └──────────────┘
```

**How it works:**
1. Pre-generate millions of random keys
2. Store in database
3. Mark keys as "used" when assigned
4. API service requests key from KGS
5. KGS returns available key

**Pros:**
- ✅ Unpredictable
- ✅ No collision
- ✅ Very fast (pre-generated)

**Cons:**
- ❌ Requires separate service
- ❌ Keys can be wasted if not used
- ❌ Additional complexity

---

### Redirect Flow

```
User             Load Balancer      Redirect Service      Redis           Database
 │                     │                    │               │                │
 │ 1. GET /aZl8xV      │                    │               │                │
 ├────────────────────>│                    │               │                │
 │                     │                    │               │                │
 │                     │ 2. Route Request   │               │                │
 │                     ├───────────────────>│               │                │
 │                     │                    │               │                │
 │                     │                    │ 3. Check Cache│                │
 │                     │                    ├──────────────>│                │
 │                     │                    │               │                │
 │                     │                    │ 4. Cache Miss │                │
 │                     │                    │<──────────────┤                │
 │                     │                    │               │                │
 │                     │                    │ 5. Query DB   │                │
 │                     │                    ├────────────────────────────────>│
 │                     │                    │               │                │
 │                     │                    │ 6. Return URL │                │
 │                     │                    │<────────────────────────────────┤
 │                     │                    │               │                │
 │                     │                    │ 7. Cache URL  │                │
 │                     │                    ├──────────────>│                │
 │                     │                    │               │                │
 │                     │ 8. 301 Redirect    │               │                │
 │                     │<───────────────────┤               │                │
 │                     │                    │               │                │
 │ 9. 301 Redirect     │                    │               │                │
 │<────────────────────┤                    │               │                │
 │                     │                    │               │                │
 │                     │                    │ 10. Log Event │                │
 │                     │                    │    (Async)    │                │
 │                     │                    ├───────────────┘                │
```

---

### Data Models

#### URLs Table
```sql
CREATE TABLE urls (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(10) UNIQUE NOT NULL,
    original_url TEXT NOT NULL,
    user_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,
    click_count BIGINT DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE
);

CREATE INDEX idx_short_code ON urls(short_code);
CREATE INDEX idx_user_id ON urls(user_id);
CREATE INDEX idx_expires_at ON urls(expires_at) WHERE expires_at IS NOT NULL;
```

#### Analytics Table (Partitioned by Date)
```sql
CREATE TABLE url_analytics (
    id BIGSERIAL,
    short_code VARCHAR(10) NOT NULL,
    clicked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address INET,
    user_agent TEXT,
    referer TEXT,
    country VARCHAR(2),
    city VARCHAR(100),
    device_type VARCHAR(20),  -- 'mobile', 'desktop', 'tablet'

    PRIMARY KEY (short_code, clicked_at)
) PARTITION BY RANGE (clicked_at);

-- Create partitions
CREATE TABLE url_analytics_2025_10 PARTITION OF url_analytics
    FOR VALUES FROM ('2025-10-01') TO ('2025-11-01');
```

#### Redis Cache Schema
```
Key: short:{short_code}
Value: {
  "original_url": "https://example.com/long/url",
  "expires_at": "2026-10-13T00:00:00Z"
}
TTL: 3600 (1 hour)
```

---

### Scaling Strategies

**1. Caching**
- **Redis Cache:** Store 20% most popular URLs (80/20 rule)
- **Cache-Aside Pattern:** Check cache → if miss, query DB → update cache
- **TTL:** 1 hour (balance freshness vs hit rate)

**2. Database Optimization**
- **Read Replicas:** 10 read replicas for redirects
- **Write Master:** 1 master for URL creation
- **Connection Pooling:** Reuse database connections

**3. Analytics Processing**
- **Async Logging:** Don't block redirect for analytics
- **Kafka Queue:** Buffer analytics events
- **Batch Processing:** Aggregate clicks every 5 minutes
- **OLAP Database:** Use ClickHouse for analytics queries

**4. CDN for Static Content**
- Serve static pages from CDN
- Reduce load on origin servers

---

### Pros and Cons

#### 301 vs 302 Redirect

**301 Permanent Redirect:**
- Browser caches redirect
- ❌ Analytics miss subsequent visits
- ✅ SEO benefit

**302 Temporary Redirect (Recommended):**
- Browser doesn't cache
- ✅ Accurate analytics
- ❌ Slightly slower (no cache)

---

### Advanced Features

**1. Custom Aliases**
```
POST /api/shorten
{
  "url": "https://example.com/long/url",
  "custom_alias": "my-link"
}

Result: https://short.ly/my-link
```
**Challenge:** Check for conflicts, reserve keywords

**2. Rate Limiting**
- Prevent abuse (spam, DDoS)
- Limit: 10 URL creations per hour per user
- Implementation: Token bucket algorithm with Redis

**3. QR Code Generation**
- Generate QR code for short URL
- Cache QR code image in CDN

**4. Link Expiration**
- Set TTL on URLs
- Cron job to deactivate expired URLs
- Return 410 Gone for expired links

---

## 4. Social Media Feed (Twitter/Instagram)

### Overview
Design a news feed system that shows posts from followed users in reverse chronological order, with features like likes, comments, and real-time updates.

### Functional Requirements
- Post creation (text, images, videos)
- Follow/unfollow users
- News feed (posts from followed users)
- Like/comment on posts
- Real-time updates
- Search posts

### Non-Functional Requirements
- Low latency (< 300ms for feed load)
- High availability
- Scalable to 1 billion users
- Handle celebrity users (100M followers)

---

### Architecture

```
┌─────────────┐
│   Clients   │
└──────┬──────┘
       │
       v
┌──────────────┐
│ API Gateway  │
└──────┬───────┘
       │
   ┌───┴────────────────┬──────────────┐
   v                    v              v
┌────────┐         ┌────────┐    ┌──────────┐
│  Post  │         │  Feed  │    │ Timeline │
│Service │         │Service │    │  Service │
└───┬────┘         └────┬───┘    └────┬─────┘
    │                   │             │
    v                   v             v
┌────────┐         ┌─────────┐   ┌────────┐
│  Post  │         │  Redis  │   │ Fanout │
│   DB   │         │  Cache  │   │ Service│
│(Cassand│         │         │   └────┬───┘
└────────┘         └─────────┘        │
                                      v
                                 ┌─────────┐
                                 │  Queue  │
                                 │ (Kafka) │
                                 └─────────┘
```

---

### Feed Generation Approaches

#### Approach 1: Fan-Out on Write (Push Model)

**Concept:** When user posts, immediately push to all followers' feeds

```
User A posts                 Fanout Service              Timeline Storage
    │                              │                          │
    │ 1. Create Post               │                          │
    ├─────────────────────────────>│                          │
    │                              │                          │
    │                              │ 2. Get Followers         │
    │                              │    (User B, C, D...)     │
    │                              ├──────────────────┐       │
    │                              │                  │       │
    │                              │<─────────────────┘       │
    │                              │                          │
    │                              │ 3. Push to Each          │
    │                              │    Follower's Feed       │
    │                              ├─────────────────────────>│
    │                              │    INSERT post_id        │
    │                              │    INTO timeline_B       │
    │                              ├─────────────────────────>│
    │                              │    INSERT post_id        │
    │                              │    INTO timeline_C       │
    │                              │                          │
    │ 4. Post Created              │                          │
    │<─────────────────────────────┤                          │
```

**Timeline Storage (Redis):**
```
Key: timeline:{user_id}
Type: Sorted Set
Value: [post_id]
Score: timestamp

Example:
timeline:user_123 = [
  (score: 1697144400, value: post_999),
  (score: 1697144300, value: post_998),
  (score: 1697144200, value: post_997)
]
```

**Pros:**
- ✅ Fast feed retrieval (read from pre-computed feed)
- ✅ Simple read path
- ✅ Good for read-heavy workload

**Cons:**
- ❌ Slow for celebrities (fan-out to 100M followers)
- ❌ Wasted computation (inactive users)
- ❌ High write load

---

#### Approach 2: Fan-Out on Read (Pull Model)

**Concept:** When user requests feed, fetch posts from followed users at read time

```
User B requests feed     Feed Service          Post Database
       │                      │                      │
       │ 1. Get Feed          │                      │
       ├─────────────────────>│                      │
       │                      │                      │
       │                      │ 2. Get Following     │
       │                      │    List (A, C, E...) │
       │                      ├──────────────┐       │
       │                      │              │       │
       │                      │<─────────────┘       │
       │                      │                      │
       │                      │ 3. Query Posts       │
       │                      │    FROM A, C, E      │
       │                      ├─────────────────────>│
       │                      │    WHERE user_id IN  │
       │                      │    (A, C, E)         │
       │                      │    ORDER BY created  │
       │                      │    LIMIT 20          │
       │                      │                      │
       │                      │ 4. Return Posts      │
       │                      │<─────────────────────┤
       │                      │                      │
       │ 5. Display Feed      │                      │
       │<─────────────────────┤                      │
```

**Pros:**
- ✅ No fanout overhead
- ✅ No wasted work for inactive users
- ✅ Immediate consistency

**Cons:**
- ❌ Slow feed generation (query multiple users)
- ❌ High read load on database
- ❌ Difficult to scale

---

#### Approach 3: Hybrid (Recommended)

**Strategy:**
- **Regular users:** Fan-out on write
- **Celebrities (> 1M followers):** Fan-out on read
- **Inactive users:** Don't fanout, compute on demand

```javascript
function handleNewPost(post, author) {
  if (author.followerCount < 1_000_000) {
    // Fan-out on write for regular users
    fanoutService.pushToFollowers(post, author.followers);
  } else {
    // Celebrity: just save post, fan-out on read
    postService.save(post);
  }
}

function getUserFeed(userId) {
  // 1. Get pre-computed timeline (regular users)
  const timeline = redis.getSortedSet(`timeline:${userId}`);

  // 2. Fetch celebrity posts on-demand
  const celebPosts = postService.getRecentPosts(user.celebFollowing);

  // 3. Merge and sort
  const feed = merge(timeline, celebPosts).sortBy('timestamp').limit(20);

  return feed;
}
```

---

### Data Models

#### Posts Table (Cassandra)
```
CREATE TABLE posts (
    post_id UUID PRIMARY KEY,
    user_id UUID,
    content TEXT,
    media_urls LIST<TEXT>,
    created_at TIMESTAMP,
    like_count INT,
    comment_count INT
);

-- Index for user's posts
CREATE INDEX ON posts (user_id);

-- Materialized view for feed generation
CREATE MATERIALIZED VIEW posts_by_user AS
    SELECT * FROM posts
    WHERE user_id IS NOT NULL AND created_at IS NOT NULL
    PRIMARY KEY (user_id, created_at, post_id)
    WITH CLUSTERING ORDER BY (created_at DESC);
```

#### Follows Table (PostgreSQL)
```sql
CREATE TABLE follows (
    follower_id BIGINT NOT NULL,
    followee_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (follower_id, followee_id)
);

CREATE INDEX idx_follows_followee ON follows(followee_id);
```

#### Likes Table (Cassandra)
```
CREATE TABLE likes (
    post_id UUID,
    user_id UUID,
    created_at TIMESTAMP,
    PRIMARY KEY (post_id, user_id)
);

-- For "who liked this post"
CREATE INDEX ON likes (post_id);
```

#### Timeline Cache (Redis)
```
Key: timeline:{user_id}
Type: Sorted Set
Score: timestamp
Value: post_id

Commands:
ZADD timeline:123 1697144400 post_999
ZREVRANGE timeline:123 0 19  -- Get top 20 posts
```

---

### Feed Loading Flow (Hybrid Approach)

```
Client          API Gateway      Feed Service       Redis Cache      Database
  │                  │                 │                  │              │
  │ 1. GET /feed     │                 │                  │              │
  ├─────────────────>│                 │                  │              │
  │                  │                 │                  │              │
  │                  │ 2. Fetch Feed   │                  │              │
  │                  ├────────────────>│                  │              │
  │                  │                 │                  │              │
  │                  │                 │ 3. Get Timeline  │              │
  │                  │                 │    (Regular)     │              │
  │                  │                 ├─────────────────>│              │
  │                  │                 │                  │              │
  │                  │                 │ 4. Return IDs    │              │
  │                  │                 │<─────────────────┤              │
  │                  │                 │                  │              │
  │                  │                 │ 5. Query Celeb   │              │
  │                  │                 │    Posts         │              │
  │                  │                 ├──────────────────────────────────>│
  │                  │                 │                  │              │
  │                  │                 │ 6. Return Posts  │              │
  │                  │                 │<──────────────────────────────────┤
  │                  │                 │                  │              │
  │                  │                 │ 7. Merge & Sort  │              │
  │                  │                 ├─────────┐        │              │
  │                  │                 │         │        │              │
  │                  │                 │<────────┘        │              │
  │                  │                 │                  │              │
  │                  │ 8. Return Feed  │                  │              │
  │                  │<────────────────┤                  │              │
  │                  │                 │                  │              │
  │ 9. Display Feed  │                 │                  │              │
  │<─────────────────┤                 │                  │              │
```

---

### Scaling Strategies

**1. Timeline Caching**
- Cache last 1000 posts per user in Redis
- Pagination: Load older posts from database
- Cache invalidation: Update on new post from followed user

**2. Database Sharding**
- **Shard by user_id:** User's posts and timeline on same shard
- **Shard by post_id:** Distribute posts across shards
- **Challenge:** Cross-shard queries for feed generation

**3. Read Replicas**
- 10+ read replicas for post queries
- Master for writes only

**4. CDN for Media**
- Store images/videos in S3
- Serve via CloudFront CDN
- Thumbnail generation for fast loading

**5. Async Fanout**
- Use Kafka to queue fanout tasks
- Process in background (eventual consistency)
- Priority queue: Recent followers first

---

### Advanced Features

**1. Real-Time Updates (WebSocket)**
```
Client              WebSocket Server         Kafka
  │                        │                   │
  │ 1. Connect             │                   │
  ├───────────────────────>│                   │
  │                        │                   │
  │                        │ 2. Subscribe to   │
  │                        │    User's Topic   │
  │                        ├──────────────────>│
  │                        │                   │
  │                        │ 3. New Post Event │
  │                        │<──────────────────┤
  │                        │                   │
  │ 4. Push Update         │                   │
  │<───────────────────────┤                   │
```

**2. Trending Posts**
- Calculate engagement score: `score = likes + 2*comments + 3*shares`
- Decay over time: `final_score = score / (age_hours + 2)^1.5`
- Update trending list every 5 minutes
- Store in Redis sorted set

**3. Content Recommendation**
- ML model predicts user interest
- Features: past likes, follows, interactions
- Inject recommended posts into feed
- A/B test recommendations

**4. Feed Ranking Algorithm**
```
score = engagement_score * decay_factor * relevance_score

Where:
- engagement_score = likes + comments + shares
- decay_factor = 1 / (1 + age_hours)
- relevance_score = ML prediction (0-1)
```

---

### Pros and Cons

#### Fan-Out on Write
**Pros:**
- ✅ Fast reads
- ✅ Offline users get updates

**Cons:**
- ❌ Slow writes (celebrities)
- ❌ Wasted work (inactive users)

#### Fan-Out on Read
**Pros:**
- ✅ Fast writes
- ✅ No wasted work

**Cons:**
- ❌ Slow reads
- ❌ Database bottleneck

#### Hybrid
**Pros:**
- ✅ Balanced approach
- ✅ Scalable

**Cons:**
- ❌ Complex implementation
- ❌ Edge cases

---

## 5. Video Streaming Platform (YouTube/Netflix)

### Overview
Design a video hosting and streaming platform supporting uploads, transcoding, adaptive bitrate streaming, and recommendations.

### Functional Requirements
- Upload videos
- Stream videos (adaptive bitrate)
- Search videos
- Recommendations
- Comments/likes
- Subscriptions

### Non-Functional Requirements
- Low latency streaming (< 1 second buffering)
- High availability
- Scalable to billions of videos
- Support 4K streaming

---

### Architecture

```
┌──────────────┐
│   Clients    │
└──────┬───────┘
       │
   ┌───┴────┐
   v        v
┌────────┐ ┌────────┐
│ Upload │ │ Stream │
│  API   │ │   API  │
└───┬────┘ └────┬───┘
    │           │
    v           v
┌────────┐ ┌────────┐
│  S3    │ │  CDN   │
│Original│ │(CloudFr│
└───┬────┘ └────────┘
    │
    v
┌────────────┐
│  Transcode │
│  Service   │
│  (Elastic) │
└─────┬──────┘
      │
      v
┌────────────┐
│   S3       │
│ Transcoded │
│ (Multiple  │
│ Qualities) │
└────────────┘
```

---

### Video Upload Flow

```
User          API Gateway      Upload Service      S3              Transcode Queue
 │                 │                 │              │                    │
 │ 1. Upload Video │                 │              │                    │
 ├────────────────>│                 │              │                    │
 │                 │                 │              │                    │
 │                 │ 2. Generate     │              │                    │
 │                 │    Pre-signed   │              │                    │
 │                 │    URL          │              │                    │
 │                 ├────────────────>│              │                    │
 │                 │                 │              │                    │
 │                 │ 3. Return URL   │              │                    │
 │                 │<────────────────┤              │                    │
 │                 │                 │              │                    │
 │ 4. Pre-signed URL                 │              │                    │
 │<────────────────┤                 │              │                    │
 │                 │                 │              │                    │
 │ 5. Upload Directly to S3          │              │                    │
 ├───────────────────────────────────────────────────>│                   │
 │                 │                 │              │                    │
 │                 │                 │              │ 6. Upload Complete │
 │                 │                 │              │    (S3 Event)      │
 │                 │                 │<─────────────┤                    │
 │                 │                 │              │                    │
 │                 │                 │ 7. Enqueue Transcode               │
 │                 │                 ├────────────────────────────────────>│
 │                 │                 │              │                    │
 │ 8. Upload Success                 │              │                    │
 │<────────────────────────────────────              │                    │
```

**Why Pre-Signed URLs?**
- Direct upload to S3 (no API server bottleneck)
- Reduced server bandwidth
- Secure (temporary, limited access)

---

### Video Transcoding

**Purpose:** Convert uploaded video to multiple formats/qualities

**Output Formats:**
```
Original: 4K (3840x2160) @ 20 Mbps
├── 1080p (1920x1080) @ 8 Mbps
├── 720p (1280x720) @ 5 Mbps
├── 480p (854x480) @ 2.5 Mbps
└── 360p (640x360) @ 1 Mbps
```

**Transcoding Service (AWS Elastic Transcoder / Custom):**
```
┌─────────────┐
│   Queue     │
│  (SQS)      │
└──────┬──────┘
       │
       v
┌─────────────┐
│  Worker     │ (Auto-scaling)
│  (FFmpeg)   │
└──────┬──────┘
       │
       v
┌─────────────┐
│     S3      │
│  (Output)   │
└─────────────┘
```

**FFmpeg Command Example:**
```bash
# Transcode to 1080p H.264
ffmpeg -i input.mp4 \
  -vf scale=1920:1080 \
  -c:v libx264 \
  -crf 23 \
  -preset medium \
  -c:a aac \
  -b:a 128k \
  output_1080p.mp4
```

**Adaptive Bitrate Streaming (HLS):**
```
video.m3u8 (Master Playlist)
├── 1080p.m3u8
│   ├── segment0.ts
│   ├── segment1.ts
│   └── segment2.ts
├── 720p.m3u8
│   ├── segment0.ts
│   ├── segment1.ts
│   └── segment2.ts
└── 480p.m3u8
    ├── segment0.ts
    ├── segment1.ts
    └── segment2.ts
```

**How HLS Works:**
1. Client requests `video.m3u8`
2. Client selects quality based on bandwidth
3. Client downloads video segments (`.ts` files)
4. Client dynamically switches quality based on network conditions

---

### Video Streaming Flow

```
Client          CDN             Origin Server       S3
  │              │                    │             │
  │ 1. Request   │                    │             │
  │    video.m3u8│                    │             │
  ├─────────────>│                    │             │
  │              │                    │             │
  │              │ 2. Cache Miss      │             │
  │              ├───────────────────>│             │
  │              │                    │             │
  │              │                    │ 3. Fetch    │
  │              │                    ├────────────>│
  │              │                    │             │
  │              │                    │ 4. Return   │
  │              │                    │<────────────┤
  │              │                    │             │
  │              │ 5. Return Playlist │             │
  │              │<───────────────────┤             │
  │              │                    │             │
  │ 6. Return    │                    │             │
  │    Playlist  │                    │             │
  │<─────────────┤                    │             │
  │              │                    │             │
  │ 7. Request   │                    │             │
  │    segment0.ts (720p)             │             │
  ├─────────────>│                    │             │
  │              │                    │             │
  │              │ 8. Cache Hit       │             │
  │              │    (Return Cached) │             │
  │<─────────────┤                    │             │
```

---

### Data Models

#### Videos Table
```sql
CREATE TABLE videos (
    video_id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    title VARCHAR(200),
    description TEXT,
    thumbnail_url TEXT,
    duration_seconds INT,
    status ENUM('processing', 'ready', 'failed'),
    view_count BIGINT DEFAULT 0,
    like_count BIGINT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE INDEX idx_videos_user ON videos(user_id);
CREATE INDEX idx_videos_status ON videos(status);
```

#### Video Files Table
```sql
CREATE TABLE video_files (
    file_id UUID PRIMARY KEY,
    video_id UUID NOT NULL,
    quality VARCHAR(10),  -- '1080p', '720p', etc.
    format VARCHAR(10),   -- 'mp4', 'hls'
    file_url TEXT,
    file_size_bytes BIGINT,

    FOREIGN KEY (video_id) REFERENCES videos(video_id)
);

CREATE INDEX idx_files_video ON video_files(video_id);
```

#### Views Analytics (ClickHouse)
```sql
CREATE TABLE video_views (
    view_id UUID,
    video_id UUID,
    user_id UUID,
    watched_seconds INT,
    quality VARCHAR(10),
    device_type VARCHAR(20),
    country VARCHAR(2),
    timestamp DateTime
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (video_id, timestamp);
```

---

### Scaling Strategies

**1. CDN Distribution**
- Store videos in multiple edge locations
- Reduce latency (serve from nearest location)
- Cache popular videos (80/20 rule)
- Cost optimization (CDN cheaper than origin)

**2. Transcoding at Scale**
- Auto-scaling worker pool
- Priority queue (popular channels first)
- Parallel processing (multiple qualities simultaneously)
- Spot instances for cost savings

**3. Database Optimization**
- **Sharding:** Shard by user_id or video_id
- **Read Replicas:** 10+ replicas for video metadata
- **Caching:** Redis for video metadata (title, thumbnail)

**4. Storage Optimization**
- **Compression:** H.264/H.265 for videos
- **Thumbnail generation:** WebP format
- **Cold storage:** Move old videos to Glacier

---

### Advanced Features

**1. Live Streaming**
```
Broadcaster        Ingest Server      Transcode         CDN             Viewers
     │                  │                │              │                 │
     │ RTMP Stream      │                │              │                 │
     ├─────────────────>│                │              │                 │
     │                  │                │              │                 │
     │                  │ HLS Segments   │              │                 │
     │                  ├───────────────>│              │                 │
     │                  │                │              │                 │
     │                  │                │ Push to CDN  │                 │
     │                  │                ├─────────────>│                 │
     │                  │                │              │                 │
     │                  │                │              │ Pull Stream     │
     │                  │                │              │<────────────────┤
```

**2. Recommendation Engine**
- Collaborative filtering (users with similar watch history)
- Content-based filtering (video metadata, categories)
- Deep learning model (TensorFlow)
- Features: watch time, likes, search queries

**3. Content Moderation**
- AI-based content detection (nudity, violence)
- User reports
- Manual review queue
- Age restrictions

**4. Monetization**
- Pre-roll/mid-roll ads
- Subscription model (Netflix)
- Pay-per-view
- Super chat (live streaming)

---

### Pros and Cons

#### HLS (HTTP Live Streaming)

**Pros:**
- ✅ Adaptive bitrate (switches quality)
- ✅ Works over HTTP (firewall-friendly)
- ✅ Cacheable by CDN
- ✅ Wide device support

**Cons:**
- ❌ Higher latency (10-30 seconds)
- ❌ More files to manage (segments)

#### DASH (Dynamic Adaptive Streaming)

**Pros:**
- ✅ Codec agnostic
- ✅ Better compression (H.265)

**Cons:**
- ❌ Less browser support
- ❌ More complex

---

## 6. E-commerce Platform (Amazon)

### Overview
Design an e-commerce system with product catalog, shopping cart, order processing, and payment integration.

### Functional Requirements
- Browse products
- Search products
- Add to cart
- Checkout and payment
- Order tracking
- Inventory management
- Reviews and ratings

### Non-Functional Requirements
- High availability (99.99%)
- Strong consistency for inventory
- Support flash sales (10K orders/second)
- PCI-DSS compliance for payments

---

### Architecture

```
┌──────────────┐
│   Clients    │
└──────┬───────┘
       │
       v
┌──────────────┐
│  API Gateway │
└──────┬───────┘
       │
   ┌───┴──────────────┬──────────────┐
   v                  v              v
┌────────┐       ┌────────┐    ┌──────────┐
│Product │       │  Cart  │    │  Order   │
│Service │       │Service │    │  Service │
└───┬────┘       └────┬───┘    └────┬─────┘
    │                 │             │
    v                 v             v
┌────────┐       ┌─────────┐   ┌────────┐
│Product │       │  Redis  │   │ Order  │
│   DB   │       │         │   │   DB   │
└────────┘       └─────────┘   └────────┘
                                    │
                                    v
                               ┌──────────┐
                               │ Payment  │
                               │  Service │
                               └──────────┘
```

---

### Shopping Cart (Redis)

**Why Redis?**
- Fast read/write (< 1ms)
- Automatic expiration (abandoned carts)
- Supports complex data structures (hash, list)

**Data Structure:**
```
Key: cart:{user_id}
Type: Hash
Fields:
  product_id_1: quantity
  product_id_2: quantity
  product_id_3: quantity

TTL: 7 days (auto-expire abandoned carts)

Example:
HSET cart:user_123 product_456 2
HSET cart:user_123 product_789 1
EXPIRE cart:user_123 604800  # 7 days
```

**Operations:**
```
# Add to cart
HSET cart:{user_id} {product_id} {quantity}

# Remove from cart
HDEL cart:{user_id} {product_id}

# Get cart
HGETALL cart:{user_id}

# Clear cart
DEL cart:{user_id}
```

---

### Order Processing Flow

```
User       API Gateway    Cart Service    Order Service   Inventory    Payment     Email
 │              │              │               │              │           │           │
 │ 1. Checkout  │              │               │              │           │           │
 ├─────────────>│              │               │              │           │           │
 │              │              │               │              │           │           │
 │              │ 2. Get Cart  │               │              │           │           │
 │              ├─────────────>│               │              │           │           │
 │              │              │               │              │           │           │
 │              │ 3. Cart Data │               │              │           │           │
 │              │<─────────────┤               │              │           │           │
 │              │              │               │              │           │           │
 │              │              │ 4. Create Order              │           │           │
 │              │              │  (Status: PENDING)           │           │           │
 │              ├──────────────────────────────>│             │           │           │
 │              │              │               │              │           │           │
 │              │              │               │ 5. Reserve   │           │           │
 │              │              │               │    Inventory │           │           │
 │              │              │               ├─────────────>│           │           │
 │              │              │               │              │           │           │
 │              │              │               │ 6. Reserved  │           │           │
 │              │              │               │<─────────────┤           │           │
 │              │              │               │              │           │           │
 │              │              │               │ 7. Process   │           │           │
 │              │              │               │    Payment   │           │           │
 │              │              │               ├──────────────────────────>│           │
 │              │              │               │              │           │           │
 │              │              │               │ 8. Payment   │           │           │
 │              │              │               │    Success   │           │           │
 │              │              │               │<──────────────────────────┤           │
 │              │              │               │              │           │           │
 │              │              │               │ 9. Update Order           │           │
 │              │              │               │    (Status: CONFIRMED)    │           │
 │              │              │               ├─────────────────┐         │           │
 │              │              │               │                 │         │           │
 │              │              │               │<────────────────┘         │           │
 │              │              │               │              │           │           │
 │              │              │               │ 10. Deduct Inventory      │           │
 │              │              │               ├─────────────>│           │           │
 │              │              │               │              │           │           │
 │              │              │               │ 11. Send Confirmation     │           │
 │              │              │               ├───────────────────────────────────────>│
 │              │              │               │              │           │           │
 │              │ 12. Order Success            │              │           │           │
 │              │<─────────────────────────────┤              │           │           │
 │              │              │               │              │           │           │
 │ 13. Confirm  │              │               │              │           │           │
 │<─────────────┤              │               │              │           │           │
```

---

### Inventory Management (Pessimistic Locking)

**Problem:** Race condition - two users buy last item simultaneously

**Solution 1: Database Lock (Pessimistic)**
```sql
START TRANSACTION;

-- Lock the row
SELECT stock FROM products
WHERE product_id = 123
FOR UPDATE;  -- Pessimistic lock

-- Check availability
IF stock >= quantity THEN
  UPDATE products
  SET stock = stock - quantity
  WHERE product_id = 123;

  COMMIT;
ELSE
  ROLLBACK;
END IF;
```

**Pros:**
- ✅ Strong consistency
- ✅ No overselling

**Cons:**
- ❌ Performance bottleneck (locks)
- ❌ Deadlock possible

---

**Solution 2: Optimistic Locking**
```sql
-- Read current version
SELECT stock, version FROM products WHERE product_id = 123;
-- stock = 10, version = 5

-- Attempt update
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE product_id = 123 AND version = 5;

-- Check rows affected
IF rows_affected = 0 THEN
  -- Version changed (concurrent update), retry
  RETRY;
ELSE
  -- Success
  COMMIT;
END IF;
```

**Pros:**
- ✅ Better performance (no locks)
- ✅ No deadlocks

**Cons:**
- ❌ Retry logic needed
- ❌ More complex

---

**Solution 3: Redis Atomic Operations**
```
# Reserve inventory
DECR inventory:product_123

# Check result
IF result < 0 THEN
  -- Out of stock
  INCR inventory:product_123  # Rollback
  RETURN "Out of stock"
ELSE
  -- Success
  RETURN "Reserved"
END IF
```

**Pros:**
- ✅ Very fast
- ✅ Atomic operation

**Cons:**
- ❌ Redis must be reliable
- ❌ Sync with database needed

---

### Data Models

#### Products Table
```sql
CREATE TABLE products (
    product_id UUID PRIMARY KEY,
    name VARCHAR(200),
    description TEXT,
    price DECIMAL(10, 2),
    stock INT,
    category_id UUID,
    image_url TEXT,
    rating DECIMAL(2, 1),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_name ON products(name);  -- Full-text search
```

#### Orders Table
```sql
CREATE TABLE orders (
    order_id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    status ENUM('pending', 'confirmed', 'shipped', 'delivered', 'cancelled'),
    total_amount DECIMAL(10, 2),
    shipping_address TEXT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
```

#### Order Items Table
```sql
CREATE TABLE order_items (
    order_item_id UUID PRIMARY KEY,
    order_id UUID NOT NULL,
    product_id UUID NOT NULL,
    quantity INT,
    price DECIMAL(10, 2),  -- Price at time of order

    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
```

---

### Scaling Strategies

**1. Product Catalog**
- **Search:** Elasticsearch for full-text search
- **Cache:** Redis for popular products
- **CDN:** Serve product images from CDN

**2. Database Sharding**
- **Shard orders by user_id:** User's orders on same shard
- **Shard products by category:** Electronics, clothing, etc.

**3. Flash Sales (High Traffic)**
- **Queue System:** Kafka queue for orders
- **Rate Limiting:** Max 10 orders per user
- **Cache Inventory:** Redis for real-time stock count
- **Auto-scaling:** Scale order service horizontally

**4. Payment Processing**
- **Async:** Don't wait for payment confirmation
- **Retry Logic:** Retry failed payments
- **Idempotency:** Prevent duplicate charges

---

### Advanced Features

**1. Recommendation Engine**
- Collaborative filtering
- "Customers who bought X also bought Y"
- Personalized homepage

**2. Dynamic Pricing**
- Adjust prices based on demand
- Competitor price monitoring
- A/B testing

**3. Fraud Detection**
- ML model to detect fraudulent orders
- Risk score based on: order value, shipping address, payment method
- Manual review for high-risk orders

**4. Multi-Currency Support**
- Store prices in USD
- Convert at checkout using real-time exchange rates
- Display in user's local currency

---

## 7. Notification System

### Overview
Design a multi-channel notification system supporting email, SMS, push notifications, and in-app messages.

### Functional Requirements
- Send notifications via multiple channels
- User preferences (opt-in/opt-out)
- Notification templates
- Scheduling (send later)
- Priority levels
- Delivery status tracking

### Non-Functional Requirements
- High throughput (1M notifications/minute)
- Low latency (< 1 second)
- High availability
- Support retries for failures

---

### Architecture

```
┌──────────────┐
│  Services    │
│ (Trigger)    │
└──────┬───────┘
       │
       v
┌──────────────┐
│ Notification │
│   Service    │
└──────┬───────┘
       │
       v
┌──────────────┐
│    Queue     │
│   (Kafka)    │
└──────┬───────┘
       │
   ┌───┴────────┬──────────┐
   v            v          v
┌────────┐ ┌────────┐ ┌────────┐
│ Email  │ │  SMS   │ │  Push  │
│ Worker │ │ Worker │ │ Worker │
└───┬────┘ └───┬────┘ └───┬────┘
    │          │          │
    v          v          v
┌────────┐ ┌────────┐ ┌────────┐
│SendGrid│ │ Twilio │ │ APNs   │
│        │ │        │ │  FCM   │
└────────┘ └────────┘ └────────┘
```

---

### Notification Flow

```
Service        Notification       Queue          Worker         Provider
  │                │                │              │                │
  │ 1. Trigger     │                │              │                │
  │    Notification│                │              │                │
  ├───────────────>│                │              │                │
  │                │                │              │                │
  │                │ 2. Check       │              │                │
  │                │    Preferences │              │                │
  │                ├────────┐       │              │                │
  │                │        │       │              │                │
  │                │<───────┘       │              │                │
  │                │                │              │                │
  │                │ 3. Enqueue     │              │                │
  │                ├───────────────>│              │                │
  │                │                │              │                │
  │ 4. Ack         │                │              │                │
  │<───────────────┤                │              │                │
  │                │                │              │                │
  │                │                │ 5. Consume   │                │
  │                │                ├─────────────>│                │
  │                │                │              │                │
  │                │                │              │ 6. Send        │
  │                │                │              ├───────────────>│
  │                │                │              │                │
  │                │                │              │ 7. Status      │
  │                │                │              │<───────────────┤
  │                │                │              │                │
  │                │                │ 8. Update    │                │
  │                │                │    Status    │                │
  │                │<───────────────────────────────┤                │
```

---

### Data Models

#### Notifications Table
```sql
CREATE TABLE notifications (
    notification_id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    type VARCHAR(50),  -- 'order_confirmed', 'payment_failed', etc.
    channel ENUM('email', 'sms', 'push', 'in_app'),
    status ENUM('pending', 'sent', 'failed', 'read'),
    content JSONB,  -- Template + variables
    scheduled_at TIMESTAMP,
    sent_at TIMESTAMP,
    created_at TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE INDEX idx_notifications_user ON notifications(user_id);
CREATE INDEX idx_notifications_status ON notifications(status);
CREATE INDEX idx_notifications_scheduled ON notifications(scheduled_at)
    WHERE scheduled_at IS NOT NULL;
```

#### User Preferences Table
```sql
CREATE TABLE notification_preferences (
    user_id UUID PRIMARY KEY,
    email_enabled BOOLEAN DEFAULT TRUE,
    sms_enabled BOOLEAN DEFAULT TRUE,
    push_enabled BOOLEAN DEFAULT TRUE,
    in_app_enabled BOOLEAN DEFAULT TRUE,

    -- Fine-grained control
    marketing_email BOOLEAN DEFAULT FALSE,
    order_updates_email BOOLEAN DEFAULT TRUE,
    order_updates_sms BOOLEAN DEFAULT FALSE,

    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

#### Templates Table
```sql
CREATE TABLE notification_templates (
    template_id UUID PRIMARY KEY,
    name VARCHAR(100),
    channel ENUM('email', 'sms', 'push'),
    subject VARCHAR(200),  -- For email
    body TEXT,  -- Supports placeholders: {{user_name}}, {{order_id}}
    created_at TIMESTAMP
);
```

---

### Implementation Details

#### 1. Message Queue (Kafka)

**Topics:**
```
- notifications.email
- notifications.sms
- notifications.push
- notifications.in_app
```

**Message Format:**
```json
{
  "notification_id": "uuid",
  "user_id": "uuid",
  "channel": "email",
  "template": "order_confirmed",
  "variables": {
    "user_name": "John Doe",
    "order_id": "12345",
    "total": "$99.99"
  },
  "priority": "high",
  "scheduled_at": "2025-10-13T10:00:00Z"
}
```

---

#### 2. Email Worker

```javascript
const consumer = kafka.consumer({ groupId: 'email-workers' });

await consumer.subscribe({ topic: 'notifications.email' });

await consumer.run({
  eachMessage: async ({ message }) => {
    const notification = JSON.parse(message.value);

    // 1. Load template
    const template = await getTemplate(notification.template);

    // 2. Render with variables
    const rendered = render(template, notification.variables);

    // 3. Send via provider
    try {
      await sendGrid.send({
        to: notification.email,
        subject: rendered.subject,
        html: rendered.body
      });

      // 4. Update status
      await updateNotificationStatus(notification.notification_id, 'sent');
    } catch (error) {
      // 5. Retry or mark failed
      if (error.retryable) {
        await retryQueue.enqueue(notification);
      } else {
        await updateNotificationStatus(notification.notification_id, 'failed');
      }
    }
  }
});
```

---

#### 3. Push Notification Worker

**FCM (Android) + APNs (iOS):**
```javascript
async function sendPushNotification(notification) {
  const user = await getUser(notification.user_id);

  // Get device tokens
  const devices = await getDeviceTokens(user.id);

  for (const device of devices) {
    if (device.platform === 'ios') {
      await apns.send({
        token: device.token,
        notification: {
          title: notification.title,
          body: notification.body
        },
        data: notification.data
      });
    } else if (device.platform === 'android') {
      await fcm.send({
        token: device.token,
        notification: {
          title: notification.title,
          body: notification.body
        },
        data: notification.data
      });
    }
  }
}
```

---

### Scaling Strategies

**1. Queue Partitioning**
- Partition Kafka topics by user_id
- Guarantees ordering per user
- Parallel processing across users

**2. Worker Auto-Scaling**
- Scale workers based on queue length
- Metrics: messages per second, lag
- Example: If lag > 10K messages, scale up

**3. Rate Limiting**
- Limit to provider's rate limits
- Email: 1000/second (SendGrid)
- SMS: 100/second (Twilio)
- Implementation: Token bucket algorithm

**4. Batching**
- Batch notifications to same user
- Send digest emails (daily summary)
- Reduces notification fatigue

---

### Advanced Features

**1. Retry Logic with Exponential Backoff**
```javascript
async function sendWithRetry(notification, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      await send(notification);
      return 'success';
    } catch (error) {
      if (attempt === maxRetries - 1) {
        throw error;  // Final attempt failed
      }

      // Exponential backoff: 1s, 2s, 4s
      await sleep(Math.pow(2, attempt) * 1000);
    }
  }
}
```

**2. Priority Queues**
```
High Priority: Order confirmations, payment alerts
Medium Priority: Shipment updates
Low Priority: Marketing emails

Implementation:
- Separate Kafka topics for each priority
- More workers for high-priority topics
```

**3. Delivery Status Tracking**
```
- Sent: Notification handed to provider
- Delivered: Provider confirmed delivery
- Opened: User opened notification (email tracking pixel)
- Clicked: User clicked link in notification
```

**4. A/B Testing**
```javascript
// Test different subject lines
const variant = user.id % 2 === 0 ? 'A' : 'B';
const subject = variant === 'A'
  ? '🎉 Your order is confirmed!'
  : 'Order #12345 confirmed';

await sendEmail({ ...notification, subject });
await trackVariant(notification.id, variant);
```

---

### Pros and Cons

#### Queue-Based Approach

**Pros:**
- ✅ Decouples services
- ✅ Handles spikes (buffer)
- ✅ Retry mechanism
- ✅ Scalable

**Cons:**
- ❌ Eventual consistency
- ❌ Complex setup
- ❌ Monitoring needed

#### Direct Send Approach

**Pros:**
- ✅ Simple
- ✅ Immediate feedback

**Cons:**
- ❌ Tight coupling
- ❌ No buffering
- ❌ Difficult to scale

---

## 8. Rate Limiter

### Overview
Design a system to limit the number of requests a user can make to prevent abuse and ensure fair usage.

### Functional Requirements
- Limit requests per user/IP
- Multiple time windows (per second, minute, hour, day)
- Different limits per API endpoint
- Graceful handling (return 429 status)

### Non-Functional Requirements
- Low latency (< 10ms overhead)
- High availability
- Distributed (multiple servers)
- Accurate counting

---

### Algorithms

#### 1. Token Bucket

**Concept:** Bucket holds tokens, each request consumes one token

```
Bucket: [● ● ● ● ●]  (Capacity: 5)
Refill Rate: 1 token/second

Request 1: [● ● ● ●]  ✅ Allowed (1 token consumed)
Request 2: [● ● ●]    ✅ Allowed (1 token consumed)
...
Request 6: []         ❌ Denied (no tokens)

After 1 second: [●]   (1 token refilled)
```

**Implementation (Redis):**
```lua
-- Redis Lua script for atomicity
local key = KEYS[1]
local capacity = tonumber(ARGV[1])
local rate = tonumber(ARGV[2])
local now = tonumber(ARGV[3])

local bucket = redis.call('HMGET', key, 'tokens', 'last_refill')
local tokens = tonumber(bucket[1]) or capacity
local last_refill = tonumber(bucket[2]) or now

-- Calculate tokens to add
local elapsed = now - last_refill
local tokens_to_add = math.floor(elapsed * rate)
tokens = math.min(capacity, tokens + tokens_to_add)

if tokens >= 1 then
  tokens = tokens - 1
  redis.call('HMSET', key, 'tokens', tokens, 'last_refill', now)
  redis.call('EXPIRE', key, 3600)
  return 1  -- Allowed
else
  return 0  -- Denied
end
```

**Pros:**
- ✅ Smooth rate limiting
- ✅ Handles bursts (if tokens available)
- ✅ Flexible (adjust rate dynamically)

**Cons:**
- ❌ Complex implementation
- ❌ Memory overhead

---

#### 2. Fixed Window Counter

**Concept:** Count requests in fixed time windows

```
Window 1 (00:00-00:59): [5 requests]  ✅ Under limit (10)
Window 2 (01:00-01:59): [8 requests]  ✅ Under limit (10)
Window 3 (02:00-02:59): [12 requests] ❌ Over limit (10)
```

**Implementation (Redis):**
```javascript
async function isAllowed(userId, limit = 10) {
  const key = `rate_limit:${userId}:${getCurrentMinute()}`;

  const count = await redis.incr(key);

  if (count === 1) {
    await redis.expire(key, 60);  // Expire after 1 minute
  }

  return count <= limit;
}

function getCurrentMinute() {
  return Math.floor(Date.now() / 60000);  // Minute timestamp
}
```

**Pros:**
- ✅ Simple implementation
- ✅ Low memory usage
- ✅ Fast

**Cons:**
- ❌ Burst at window edge (20 requests in 2 seconds across window boundary)
- ❌ Not smooth

**Example of Edge Case:**
```
11:59:30 - 11:59:59: 10 requests ✅ (Window 1)
12:00:00 - 12:00:29: 10 requests ✅ (Window 2)

Result: 20 requests in 30 seconds (should be max 10/min)
```

---

#### 3. Sliding Window Log

**Concept:** Keep timestamp of each request, count within window

```
Requests: [10:00:15, 10:00:30, 10:00:45, 10:01:00]
Current time: 10:01:10
Window: Last 60 seconds

Filter: [10:00:30, 10:00:45, 10:01:00]  (3 requests in last minute)
```

**Implementation (Redis Sorted Set):**
```javascript
async function isAllowed(userId, limit = 10, windowSeconds = 60) {
  const key = `rate_limit:${userId}`;
  const now = Date.now();
  const windowStart = now - (windowSeconds * 1000);

  // Remove old requests
  await redis.zremrangebyscore(key, '-inf', windowStart);

  // Count requests in window
  const count = await redis.zcard(key);

  if (count < limit) {
    // Add current request
    await redis.zadd(key, now, `${now}-${Math.random()}`);
    await redis.expire(key, windowSeconds);
    return true;  // Allowed
  }

  return false;  // Denied
}
```

**Pros:**
- ✅ Accurate (true sliding window)
- ✅ No burst issues

**Cons:**
- ❌ High memory usage (stores every request)
- ❌ Slower (more operations)

---

#### 4. Sliding Window Counter (Hybrid)

**Concept:** Combine fixed window + weighted count from previous window

```
Previous Window (10:00): 8 requests
Current Window (10:01): 4 requests
Current Time: 10:01:30 (50% into window)

Estimated count = (8 * 50%) + 4 = 8 requests
```

**Implementation:**
```javascript
async function isAllowed(userId, limit = 10) {
  const now = Date.now();
  const currentWindow = Math.floor(now / 60000);
  const previousWindow = currentWindow - 1;

  const currentKey = `rate_limit:${userId}:${currentWindow}`;
  const previousKey = `rate_limit:${userId}:${previousWindow}`;

  const [currentCount, previousCount] = await Promise.all([
    redis.get(currentKey),
    redis.get(previousKey)
  ]);

  // Calculate weight of previous window
  const elapsedInWindow = (now % 60000) / 60000;  // 0 to 1
  const weightedPrevious = (previousCount || 0) * (1 - elapsedInWindow);

  const estimatedCount = weightedPrevious + (currentCount || 0);

  if (estimatedCount < limit) {
    await redis.incr(currentKey);
    await redis.expire(currentKey, 120);  // 2 minutes
    return true;
  }

  return false;
}
```

**Pros:**
- ✅ Accurate (smooth)
- ✅ Low memory (only 2 counters)
- ✅ Fast

**Cons:**
- ❌ Slightly complex
- ❌ Approximation (not exact)

---

### Distributed Rate Limiting

**Challenge:** Multiple servers need to share rate limit state

**Solution: Centralized Redis**

```
Server 1 ──┐
Server 2 ──┼──> Redis (Shared State)
Server 3 ──┘
```

**Race Condition:**
```
Server 1                  Redis                 Server 2
    │                       │                       │
    │ GET count (9)         │                       │
    ├──────────────────────>│                       │
    │                       │       GET count (9)   │
    │                       │<──────────────────────┤
    │                       │                       │
    │ INCR (10) ✅          │                       │
    ├──────────────────────>│                       │
    │                       │                       │
    │                       │       INCR (11) ❌    │
    │                       │<──────────────────────┤
```

**Fix: Lua Script (Atomic)**
```lua
local count = redis.call('INCR', KEYS[1])
if count == 1 then
  redis.call('EXPIRE', KEYS[1], ARGV[1])
end

if count > tonumber(ARGV[2]) then
  return 0  -- Denied
else
  return 1  -- Allowed
end
```

---

### Response Headers

**Standard rate limit headers:**
```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1697144400

HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1697144400
Retry-After: 60
```

---

### Comparison of Algorithms

| Algorithm | Memory | Accuracy | Bursts | Complexity |
|-----------|--------|----------|--------|------------|
| **Token Bucket** | Medium | High | Allowed | High |
| **Fixed Window** | Low | Low | Possible | Low |
| **Sliding Log** | High | Very High | Not Allowed | Medium |
| **Sliding Counter** | Low | High | Minimal | Medium |

---

## 9. Design Patterns Summary

### Common Patterns Used Across Systems

#### 1. **Caching (Cache-Aside)**
**Used in:** All systems
**Pattern:**
```
1. Check cache
2. If hit, return cached data
3. If miss, query database
4. Update cache
5. Return data
```

**Example:**
```javascript
async function getUser(userId) {
  // 1. Check cache
  let user = await redis.get(`user:${userId}`);

  if (user) {
    return JSON.parse(user);  // Cache hit
  }

  // 2. Cache miss - query DB
  user = await database.findUser(userId);

  // 3. Update cache
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));

  return user;
}
```

---

#### 2. **Load Balancing**
**Used in:** All systems
**Strategies:**
- **Round Robin:** Distribute requests evenly
- **Least Connections:** Route to server with fewest active connections
- **Consistent Hashing:** Route based on request key (user_id)

---

#### 3. **Database Sharding**
**Used in:** All scalable systems
**Strategies:**
- **Horizontal Sharding:** Split by rows (user_id, region)
- **Vertical Sharding:** Split by columns (user_profile vs user_activity)

---

#### 4. **Message Queue (Producer-Consumer)**
**Used in:** Chat, Notification, E-commerce
**Pattern:**
```
Producer → Queue → Consumer
```

**Benefits:**
- Decoupling
- Buffering
- Async processing

---

#### 5. **CQRS (Command Query Responsibility Segregation)**
**Used in:** E-commerce, Social Media
**Pattern:**
```
Write Path: API → Command Service → Write DB
Read Path: API → Query Service → Read DB (Replica/Cache)
```

**Benefits:**
- Optimize reads and writes separately
- Scale independently

---

## 10. Key Concepts & Glossary

### Performance Metrics

**Latency:** Time to complete a single request
- **Target:** < 100ms for API calls

**Throughput:** Number of requests per second
- **Target:** 10K+ requests/second

**Availability:** Uptime percentage
- **99.9%:** 8.76 hours downtime/year
- **99.99%:** 52.56 minutes downtime/year

---

### CAP Theorem

**You can only choose 2 out of 3:**

**Consistency (C):** All nodes see the same data
**Availability (A):** Every request gets a response
**Partition Tolerance (P):** System works despite network failures

**Examples:**
- **CP System:** MySQL (sacrifices availability for consistency)
- **AP System:** Cassandra (sacrifices consistency for availability)
- **CA System:** Not possible in distributed systems

---

### Database Types

#### SQL (Relational)
- **Examples:** PostgreSQL, MySQL
- **Use cases:** Structured data, complex queries, transactions
- **Pros:** ACID guarantees, strong consistency
- **Cons:** Harder to scale horizontally

#### NoSQL

**Document Store:**
- **Examples:** MongoDB
- **Use cases:** Flexible schema, nested data
- **Pros:** Schema-less, easy to scale
- **Cons:** No joins, eventual consistency

**Key-Value Store:**
- **Examples:** Redis, DynamoDB
- **Use cases:** Caching, session storage
- **Pros:** Very fast, simple
- **Cons:** No complex queries

**Wide-Column Store:**
- **Examples:** Cassandra, HBase
- **Use cases:** Time-series data, write-heavy workloads
- **Pros:** High write throughput, scalable
- **Cons:** Limited query capabilities

**Graph Database:**
- **Examples:** Neo4j
- **Use cases:** Social networks, recommendation engines
- **Pros:** Fast relationship queries
- **Cons:** Hard to scale

---

### Caching Strategies

**Cache-Aside (Lazy Loading):**
- Application checks cache first
- On miss, load from DB and update cache

**Write-Through:**
- Write to cache and DB simultaneously
- Ensures consistency

**Write-Behind (Write-Back):**
- Write to cache immediately
- Async write to DB later
- Risk of data loss

---

### Consistency Models

**Strong Consistency:**
- Read always returns latest write
- Example: SQL databases

**Eventual Consistency:**
- Reads may return stale data temporarily
- System eventually converges to consistent state
- Example: DynamoDB, Cassandra

**Read-Your-Writes:**
- User always sees their own writes
- Others may see stale data

---

### Load Balancer Algorithms

**Round Robin:**
- Distribute requests evenly in sequence

**Least Connections:**
- Route to server with fewest active connections

**IP Hash:**
- Hash client IP to determine server
- Same client always goes to same server

**Weighted Round Robin:**
- Distribute based on server capacity

---

### API Design

**RESTful Principles:**
```
GET /users          - List users
GET /users/123      - Get user
POST /users         - Create user
PUT /users/123      - Update user (full)
PATCH /users/123    - Update user (partial)
DELETE /users/123   - Delete user
```

**Pagination:**
```
GET /users?page=2&limit=20
GET /users?cursor=abc123
```

**Filtering:**
```
GET /users?status=active&role=admin
```

**Sorting:**
```
GET /users?sort=created_at&order=desc
```

---

### Security Best Practices

**Authentication:**
- JWT tokens
- OAuth 2.0
- Session cookies

**Authorization:**
- Role-Based Access Control (RBAC)
- Attribute-Based Access Control (ABAC)

**Data Protection:**
- HTTPS/TLS for all traffic
- Encrypt sensitive data at rest
- Hash passwords (bcrypt, Argon2)

**API Security:**
- Rate limiting
- Input validation
- SQL injection prevention
- CORS configuration

---

## Interview Tips

### How to Approach System Design Questions

**1. Clarify Requirements (5 minutes)**
- Functional requirements
- Non-functional requirements
- Scale estimates (users, requests, storage)

**2. High-Level Design (10 minutes)**
- Draw basic architecture
- Identify major components
- Explain data flow

**3. Deep Dive (15 minutes)**
- Database schema
- API design
- Scaling strategies
- Bottlenecks

**4. Trade-offs (5 minutes)**
- Discuss alternatives
- Explain chosen approach
- Mention limitations

---

### Common Follow-Up Questions

**"How would you scale this?"**
- Horizontal scaling (add more servers)
- Caching
- Database sharding
- CDN

**"What if a server crashes?"**
- Redundancy (multiple servers)
- Load balancer health checks
- Database replication

**"How do you handle consistency?"**
- Choose between CP and AP
- Eventual consistency vs strong consistency
- Conflict resolution

**"How do you monitor this system?"**
- Metrics (latency, throughput, error rate)
- Logging (centralized logging)
- Alerting (PagerDuty, Opsgenie)
- Tracing (distributed tracing)

---

### Estimation Techniques

**Back-of-the-envelope calculations:**

**Example: Twitter**
- Users: 500M
- Active users: 100M daily
- Tweets per day: 100M
- Average tweet size: 300 bytes
- Storage per day: 100M * 300 = 30GB
- Storage per year: 30GB * 365 = ~11TB

**Traffic Estimates:**
- Requests per second: daily_requests / 86400
- Example: 100M / 86400 = ~1,200 RPS

**Storage Estimates:**
- Images: 5MB average
- Videos: 100MB average
- Text: KB range

---

## Conclusion

This guide covers the most common system design interview questions with multiple architectural approaches, trade-offs, and scaling strategies. Remember:

1. **No single correct answer** - System design is about trade-offs
2. **Think out loud** - Explain your reasoning
3. **Ask clarifying questions** - Don't make assumptions
4. **Consider scale** - Design for growth
5. **Know the basics** - Understand databases, caching, load balancing

**Good luck with your interviews!** 🚀
