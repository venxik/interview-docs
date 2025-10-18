# AWS Services Comparison & Interview Guide

## Table of Contents
1. [AWS Service Selection Guide](#aws-service-selection-guide)
2. [Service Comparisons](#service-comparisons)
3. [Common Interview Questions](#common-interview-questions)
4. [Scenario-Based Questions](#scenario-based-questions)
5. [Best Practices Summary](#best-practices-summary)

---

## AWS Service Selection Guide

### Decision Trees

#### Compute Services Decision Tree

```
Need to run code?
│
├─ Manage servers?
│  ├─ YES → EC2
│  │   ├─ Containers? → ECS/EKS
│  │   └─ VMs? → EC2
│  │
│  └─ NO → Serverless
│      ├─ HTTP APIs? → Lambda + API Gateway
│      ├─ Event-driven? → Lambda
│      └─ Long-running containers? → Fargate
│
├─ Need persistent compute?
│  └─ EC2 with Auto Scaling
│
└─ Batch processing?
    ├─ Short tasks (< 15 min)? → Lambda
    └─ Long tasks? → Batch or ECS
```

#### Storage Services Decision Tree

```
What type of data?
│
├─ Files (objects)?
│  ├─ Frequently accessed? → S3 Standard
│  ├─ Infrequent access? → S3 IA
│  └─ Archive? → S3 Glacier
│
├─ Block storage (disk)?
│  ├─ For EC2? → EBS
│  └─ Shared across EC2? → EFS
│
├─ File system?
│  ├─ Windows? → FSx for Windows
│  ├─ Lustre (HPC)? → FSx for Lustre
│  └─ Linux NFS? → EFS
│
└─ Temporary data?
    └─ Instance Store (ephemeral)
```

#### Database Services Decision Tree

```
What type of database?
│
├─ Relational (SQL)?
│  ├─ Need PostgreSQL/MySQL? → RDS
│  ├─ Need performance? → Aurora
│  └─ Serverless? → Aurora Serverless
│
├─ NoSQL?
│  ├─ Key-value? → DynamoDB
│  ├─ Document? → DocumentDB (MongoDB)
│  ├─ Graph? → Neptune
│  └─ Time-series? → Timestream
│
├─ In-memory cache?
│  ├─ Simple cache? → ElastiCache (Memcached)
│  └─ Advanced (pub/sub, lists)? → ElastiCache (Redis)
│
└─ Data warehouse?
    └─ Redshift
```

---

## Service Comparisons

### Compute: EC2 vs Lambda vs Fargate vs ECS

| Feature | EC2 | Lambda | Fargate | ECS |
|---------|-----|--------|---------|-----|
| **Management** | Full control | Fully managed | Managed containers | Container orchestration |
| **Pricing** | Per hour | Per execution | Per task | Per hour (EC2) |
| **Scaling** | Manual/Auto Scaling | Automatic | Automatic | Manual/Auto Scaling |
| **Cold Start** | None | Yes (100-1000ms) | None | None |
| **Max Duration** | Unlimited | 15 minutes | Unlimited | Unlimited |
| **Use Case** | Long-running, custom | Event-driven, APIs | Serverless containers | Microservices |
| **Cost (1M req)** | $15-50/month | $10/month | $20/month | $30/month |

**When to use:**

```typescript
// EC2: Need full control, specific OS/software
// Example: Legacy app, GPU processing, Windows server
const ec2 = new AWS.EC2();
await ec2.runInstances({
  ImageId: 'ami-12345',
  InstanceType: 't3.medium',
  MinCount: 1,
  MaxCount: 1
}).promise();

// Lambda: Event-driven, short tasks
// Example: API endpoints, S3 triggers, scheduled tasks
export const handler = async (event: any) => {
  // Auto-scales, pay per execution
  return { statusCode: 200, body: 'OK' };
};

// Fargate: Containers without managing servers
// Example: Microservices, batch jobs
{
  "launchType": "FARGATE",  // No EC2 to manage
  "networkMode": "awsvpc"
}

// ECS: Container orchestration with control
// Example: Complex microservices, need EC2 access
{
  "launchType": "EC2",  // You manage EC2 instances
  "clusterName": "my-cluster"
}
```

### Storage: S3 vs EBS vs EFS

| Feature | S3 | EBS | EFS |
|---------|-----|-----|-----|
| **Type** | Object storage | Block storage | File system |
| **Access** | HTTP/HTTPS | Attached to EC2 | NFS mount |
| **Size** | Unlimited | 16 TB per volume | Unlimited |
| **Availability** | 99.99% | 99.99% | 99.99% |
| **Durability** | 99.999999999% | 99.999% | 99.999999999% |
| **Use Case** | Static files, backups | Database storage | Shared storage |
| **Cost (100 GB)** | $2.30/month | $10/month | $30/month |

**Examples:**

```typescript
// S3: Object storage (files, images, videos)
import AWS from 'aws-sdk';
const s3 = new AWS.S3();

// Upload file
await s3.putObject({
  Bucket: 'my-bucket',
  Key: 'photos/image.jpg',
  Body: fileBuffer,
  ContentType: 'image/jpeg'
}).promise();

// Generate pre-signed URL (temporary access)
const url = s3.getSignedUrl('getObject', {
  Bucket: 'my-bucket',
  Key: 'photos/image.jpg',
  Expires: 3600  // 1 hour
});

// EBS: Block storage (database, OS disk)
// Attached to EC2 instance
{
  "VolumeType": "gp3",
  "Size": 100,  // 100 GB
  "Iops": 3000,
  "AvailabilityZone": "us-east-1a"
}

// Mount in EC2:
// sudo mkfs -t ext4 /dev/sdf
// sudo mount /dev/sdf /data

// EFS: Shared file system (multiple EC2 instances)
import { EFS } from 'aws-sdk';
const efs = new EFS();

// Create file system
const fs = await efs.createFileSystem({
  PerformanceMode: 'generalPurpose',
  Encrypted: true
}).promise();

// Mount in EC2 (multiple instances can mount):
// sudo mount -t nfs4 fs-12345.efs.us-east-1.amazonaws.com:/ /mnt/efs
```

### Database: RDS vs DynamoDB vs Aurora

| Feature | RDS | DynamoDB | Aurora |
|---------|-----|----------|--------|
| **Type** | Relational (SQL) | NoSQL (Key-Value) | Relational (SQL) |
| **Management** | Managed | Fully managed | Fully managed |
| **Scaling** | Vertical (manual) | Horizontal (auto) | Horizontal (auto) |
| **Latency** | 10-50ms | Single-digit ms | 5-10ms |
| **Query** | SQL (complex) | Simple key-value | SQL (complex) |
| **Transactions** | ACID | Limited | ACID |
| **Cost (100 GB)** | $30/month | $25/month | $60/month |

**When to use:**

```typescript
// RDS: Traditional relational database
// Example: Complex joins, ACID transactions, existing MySQL app
import { Sequelize } from 'sequelize';

const sequelize = new Sequelize({
  dialect: 'mysql',
  host: process.env.RDS_ENDPOINT,
  username: 'admin',
  password: process.env.DB_PASSWORD,
  database: 'school_admin'
});

// Complex queries with joins
const students = await db.query(`
  SELECT s.*, c.class_name
  FROM students s
  JOIN class_students cs ON s.id = cs.student_id
  JOIN classes c ON cs.class_id = c.id
  WHERE c.grade = 10
`);

// DynamoDB: High-scale, simple queries
// Example: User sessions, game leaderboards, simple CRUD
import { DynamoDB } from 'aws-sdk';

const dynamodb = new DynamoDB.DocumentClient();

// Simple key-value lookups (< 10ms)
const student = await dynamodb.get({
  TableName: 'Students',
  Key: { email: 'john@school.com' }
}).promise();

// Aurora: High-performance MySQL/PostgreSQL
// Example: Need RDS but with better performance and scaling
{
  "Engine": "aurora-mysql",
  "EngineVersion": "5.7.mysql_aurora.2.10.1",
  "DBClusterIdentifier": "school-admin-cluster",
  "MasterUsername": "admin",
  "DatabaseName": "school_admin",
  "EnableHttpEndpoint": true,  // Data API (serverless queries)

  // Auto-scaling read replicas
  "ScalingConfiguration": {
    "MinCapacity": 2,
    "MaxCapacity": 16,
    "AutoPause": true
  }
}
```

### Load Balancers: ALB vs NLB vs CLB

| Feature | ALB (Layer 7) | NLB (Layer 4) | CLB (Deprecated) |
|---------|---------------|---------------|------------------|
| **Layer** | HTTP/HTTPS | TCP/UDP | HTTP/TCP |
| **Routing** | Path, host, header | IP, port | Basic |
| **Performance** | Good | Excellent | Good |
| **Use Case** | Web apps, APIs | Gaming, IoT | Legacy |
| **WebSocket** | Yes | Yes | No |
| **Static IP** | No | Yes | No |
| **Cost** | $$ | $$ | $ |

**Examples:**

```typescript
// ALB: Application Load Balancer (Layer 7 - HTTP)
{
  "Name": "school-admin-alb",
  "Scheme": "internet-facing",
  "Type": "application",

  // Path-based routing
  "Listeners": [
    {
      "Port": 443,
      "Protocol": "HTTPS",
      "DefaultActions": [
        {
          "Type": "forward",
          "TargetGroupArn": "arn:aws:elasticloadbalancing:..."
        }
      ],
      "Rules": [
        {
          "Conditions": [{ "Field": "path-pattern", "Values": ["/api/*"] }],
          "Actions": [{ "Type": "forward", "TargetGroupArn": "api-tg" }]
        },
        {
          "Conditions": [{ "Field": "path-pattern", "Values": ["/admin/*"] }],
          "Actions": [{ "Type": "forward", "TargetGroupArn": "admin-tg" }]
        }
      ]
    }
  ]
}

// NLB: Network Load Balancer (Layer 4 - TCP)
// Use for: Extreme performance, static IP, non-HTTP protocols
{
  "Name": "gaming-nlb",
  "Scheme": "internet-facing",
  "Type": "network",

  "Listeners": [
    {
      "Port": 25565,  // Minecraft server
      "Protocol": "TCP",
      "DefaultActions": [
        { "Type": "forward", "TargetGroupArn": "game-servers-tg" }
      ]
    }
  ]
}
```

---

## Common Interview Questions

### Q1: How would you migrate your school admin system to AWS?

**Answer:**

"I would use a phased migration approach:

**Phase 1: Assessment (Week 1)**

1. **Inventory current infrastructure:**
   - Application: Node.js/Express API
   - Database: MySQL (self-hosted)
   - Frontend: React SPA
   - File storage: Local filesystem (CSV uploads)

2. **Define requirements:**
   - Availability: 99.9% (8.76 hours downtime/year)
   - Performance: < 200ms API response (p95)
   - Budget: ~$300/month
   - Compliance: None (educational data, but not FERPA-compliant required)

**Phase 2: Design Architecture (Week 2)**

```
┌─────────────────────────────────────────┐
│ Route 53 (DNS: school-admin.com)       │
└─────────────┬───────────────────────────┘
              │
┌─────────────┴───────────────────────────┐
│ CloudFront (CDN for static assets)      │
└─────────────┬───────────────────────────┘
              │
      ┌───────┴────────┐
      │                │
┌─────┴──────┐   ┌─────┴──────┐
│ S3         │   │    ALB     │
│ (Frontend) │   └─────┬──────┘
└────────────┘         │
                 ┌─────┴─────┐
                 │           │
            ┌────┴───┐  ┌────┴───┐
            │ ECS    │  │ ECS    │  ← Auto Scaling (2-10)
            │ Task 1 │  │ Task 2 │
            └────┬───┘  └────┬───┘
                 │           │
                 └─────┬─────┘
                       │
                 ┌─────┴─────┐
                 │           │
            ┌────┴────┐ ┌────┴─────┐
            │RDS MySQL│ │ElastiCache│
            │(Multi-AZ│ │  (Redis)  │
            └────┬────┘ └──────────┘
                 │
            ┌────┴────┐
            │   S3    │
            │ (Uploads│
            └─────────┘
```

**Phase 3: Migration (Weeks 3-4)**

**Step 1: Database (Day 1-2)**
```bash
# Export from current MySQL
mysqldump -u admin -p school_admin > backup.sql

# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier school-admin-db \
  --db-instance-class db.t3.medium \
  --engine mysql \
  --engine-version 8.0.33 \
  --master-username admin \
  --master-user-password [secure-password] \
  --allocated-storage 100 \
  --multi-az \
  --backup-retention-period 7

# Import to RDS
mysql -h mydb.c9akciq32.us-east-1.rds.amazonaws.com -u admin -p school_admin < backup.sql

# Set up replication (during transition)
# Current DB → RDS (real-time sync)
```

**Step 2: Backend (Day 3-4)**
```bash
# Containerize application
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
CMD ["node", "dist/server.js"]

# Build and push to ECR
aws ecr create-repository --repository-name school-admin-api
docker build -t school-admin-api .
docker tag school-admin-api:latest 123.dkr.ecr.us-east-1.amazonaws.com/school-admin-api:latest
docker push 123.dkr.ecr.us-east-1.amazonaws.com/school-admin-api:latest

# Deploy to ECS Fargate
aws ecs create-cluster --cluster-name school-admin-cluster
aws ecs create-service \
  --cluster school-admin-cluster \
  --service-name api-service \
  --task-definition school-admin-api:1 \
  --desired-count 2 \
  --launch-type FARGATE \
  --load-balancers targetGroupArn=arn:aws:elasticloadbalancing:...,containerName=api,containerPort=3000
```

**Step 3: Frontend (Day 5)**
```bash
# Build React app
npm run build

# Upload to S3
aws s3 mb s3://school-admin-frontend
aws s3 sync build/ s3://school-admin-frontend

# Enable S3 static hosting
aws s3 website s3://school-admin-frontend \
  --index-document index.html \
  --error-document index.html

# Create CloudFront distribution
aws cloudfront create-distribution \
  --origin-domain-name school-admin-frontend.s3.amazonaws.com \
  --default-root-object index.html
```

**Step 4: DNS Cutover (Day 6)**
```bash
# Update Route 53
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456 \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "school-admin.com",
        "Type": "A",
        "AliasTarget": {
          "DNSName": "d1234.cloudfront.net",
          "HostedZoneId": "Z2FDTNDATAQYW2",
          "EvaluateTargetHealth": false
        }
      }
    }]
  }'

# Monitor traffic shift (DNS propagation ~24 hours)
```

**Phase 4: Optimization (Week 5)**

1. **Caching:**
   - Add ElastiCache Redis
   - Cache student/class queries (5-min TTL)
   - Result: 80% cache hit rate, 50ms → 5ms latency

2. **Auto Scaling:**
   - Set up ECS Auto Scaling (2-10 tasks based on CPU)
   - Result: Handle 10x traffic spikes

3. **Monitoring:**
   - CloudWatch dashboards
   - Alarms (CPU > 80%, Error rate > 1%)
   - X-Ray distributed tracing

**Phase 5: Decommission (Week 6)**

- Monitor AWS environment (1 week)
- Shut down old servers
- Cancel old hosting

**Total Migration Time:** 6 weeks

**Cost Comparison:**
- **Before:** Self-hosted VPS ($50/month) + Time (maintenance)
- **After:** AWS ($300/month) + No maintenance

**Benefits:**
- ✅ Auto-scaling (handle traffic spikes)
- ✅ High availability (99.9%+)
- ✅ Automated backups
- ✅ No server maintenance
- ✅ Global CDN (faster for users)"

### Q2: Your application suddenly gets 10x traffic. How do you handle it?

**Answer:**

"**Immediate actions (0-5 minutes):**

1. **Check Auto Scaling:**
```bash
# Verify Auto Scaling is working
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names school-admin-asg

# If not scaling fast enough, manually increase
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name school-admin-asg \
  --desired-capacity 20  # Scale to 20 instances immediately
```

2. **Increase database connections:**
```typescript
// Update connection pool
const sequelize = new Sequelize({
  pool: {
    max: 50,  // Increase from 10
    min: 5
  }
});
```

3. **Enable aggressive caching:**
```typescript
// Increase cache TTL temporarily
await redis.setex('students:list', 3600, data);  // 1 hour instead of 5 min
```

**Short-term (Hours):**

1. **Add read replicas (database):**
```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier school-admin-db-replica-2 \
  --source-db-instance-identifier school-admin-db
```

2. **CloudFront optimization:**
```bash
# Increase CloudFront cache TTL
aws cloudfront update-distribution \
  --id E1234 \
  --distribution-config '{
    \"DefaultCacheBehavior\": {
      \"DefaultTTL\": 3600  // 1 hour
    }
  }'
```

3. **Throttle non-critical operations:**
```typescript
// Disable analytics, email notifications temporarily
if (isHighTraffic()) {
  // Skip non-essential work
  return;
}
```

**Long-term (Days):**

1. **Optimize queries:**
```sql
-- Add indexes for slow queries
CREATE INDEX idx_student_grade ON students(grade);
CREATE INDEX idx_class_created ON classes(created_at DESC);
```

2. **Implement rate limiting:**
```typescript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100  // Max 100 requests per window per IP
});

app.use('/api/', limiter);
```

3. **Add monitoring:**
```typescript
// CloudWatch alarms
{
  \"AlarmName\": \"HighTraffic\",
  \"MetricName\": \"RequestCount\",
  \"Threshold\": 10000,
  \"ComparisonOperator\": \"GreaterThanThreshold\",
  \"AlarmActions\": [\"arn:aws:sns:us-east-1:123:alerts\"]
}
```

**Architecture improvements:**

```
Before:
Client → ALB → 2 EC2 instances → RDS

After (10x traffic):
Client
  ↓
CloudFront (cache static + API responses)
  ↓
ALB
  ↓
Auto Scaling (2-20 EC2 instances)
  ↓
ElastiCache Redis (aggressive caching)
  ↓
RDS Primary + 3 Read Replicas

Result:
✅ Handle 10x traffic (100 RPS → 1000 RPS)
✅ Maintain < 200ms latency
✅ No downtime
```

**Cost impact:**
- Normal: $300/month
- 10x traffic: $600/month (Auto Scaling + Read Replicas)
- After optimization: $400/month (better caching)"

### Q3: How do you secure data at rest and in transit in AWS?

**Answer:**

"I implement security at multiple layers:

**Data in Transit (Network):**

1. **HTTPS/TLS everywhere:**
```typescript
// Force HTTPS
app.use((req, res, next) => {
  if (!req.secure && process.env.NODE_ENV === 'production') {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});

// ALB with SSL/TLS certificate
{
  \"Certificates\": [{
    \"CertificateArn\": \"arn:aws:acm:us-east-1:123:certificate/abc\"
  }],
  \"Port\": 443,
  \"Protocol\": \"HTTPS\",
  \"SslPolicy\": \"ELBSecurityPolicy-TLS-1-2-2017-01\"
}
```

2. **VPC isolation:**
```
Public Subnet (ALB only)
  ↓
Private Subnet (EC2, no internet access)
  ↓
Private Subnet (RDS, no internet access)

Security Group rules:
- ALB: Allow 443 from 0.0.0.0/0
- EC2: Allow 3000 from ALB only
- RDS: Allow 3306 from EC2 only
```

3. **Database SSL:**
```typescript
const sequelize = new Sequelize({
  dialectOptions: {
    ssl: {
      require: true,
      rejectUnauthorized: true
    }
  }
});
```

**Data at Rest (Storage):**

1. **S3 encryption:**
```typescript
// Server-side encryption
await s3.putObject({
  Bucket: 'my-bucket',
  Key: 'file.txt',
  Body: data,
  ServerSideEncryption: 'AES256'  // Or 'aws:kms'
}).promise();

// Enable default encryption
await s3.putBucketEncryption({
  Bucket: 'my-bucket',
  ServerSideEncryptionConfiguration: {
    Rules: [{
      ApplyServerSideEncryptionByDefault: {
        SSEAlgorithm: 'AES256'
      }
    }]
  }
}).promise();
```

2. **RDS encryption:**
```bash
aws rds create-db-instance \
  --db-instance-identifier school-admin-db \
  --storage-encrypted \  # ✅ Encrypt at rest
  --kms-key-id arn:aws:kms:us-east-1:123:key/abc
```

3. **EBS encryption:**
```bash
aws ec2 create-volume \
  --size 100 \
  --encrypted \  # ✅ Encrypt at rest
  --kms-key-id arn:aws:kms:us-east-1:123:key/abc
```

4. **Secrets management:**
```typescript
import { SecretsManager } from 'aws-sdk';

const secretsManager = new SecretsManager();

// Store secrets
await secretsManager.createSecret({
  Name: 'prod/db/password',
  SecretString: JSON.stringify({
    username: 'admin',
    password: 'MySecurePassword123'
  })
}).promise();

// Retrieve secrets (in application)
const secret = await secretsManager.getSecretValue({
  SecretId: 'prod/db/password'
}).promise();

const { username, password } = JSON.parse(secret.SecretString);
```

**Access Control:**

1. **IAM roles (not access keys):**
```typescript
// ✅ EC2/Lambda automatically gets credentials from role
const s3 = new AWS.S3();  // No access keys needed!

// ❌ NEVER hardcode
const s3 = new AWS.S3({
  accessKeyId: 'AKIAIOSFODNN7EXAMPLE',  // DON'T!
  secretAccessKey: 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY'
});
```

2. **Least privilege policies:**
```json
{
  \"Version\": \"2012-10-17\",
  \"Statement\": [{
    \"Effect\": \"Allow\",
    \"Action\": [
      \"s3:GetObject\",
      \"s3:PutObject\"
    ],
    \"Resource\": \"arn:aws:s3:::my-bucket/uploads/*\"
  }]
}
```

**Monitoring:**

1. **CloudTrail (audit logs):**
```bash
# Enable CloudTrail
aws cloudtrail create-trail \
  --name school-admin-trail \
  --s3-bucket-name cloudtrail-logs

# Log all API calls
# Who accessed what, when, from where
```

2. **GuardDuty (threat detection):**
```bash
# Enable GuardDuty
aws guardduty create-detector \
  --enable

# Detects:
# - Unusual API calls
# - Compromised EC2 instances
# - Reconnaissance attacks
```

**Result:**
✅ Data encrypted in transit (TLS)
✅ Data encrypted at rest (AES-256)
✅ No hardcoded secrets
✅ Least privilege access
✅ Complete audit trail"

### Q4: Explain your disaster recovery strategy

**Answer:**

"I use a **Warm Standby** approach with Multi-AZ and automated backups:

**Architecture:**

```
Primary Region (us-east-1):
┌────────────────────────────────┐
│ ALB (Active)                   │
│   ↓                            │
│ ECS (2-10 tasks)               │
│   ↓                            │
│ RDS Multi-AZ                   │
│ ├─ AZ-1a (Primary)             │
│ └─ AZ-1b (Standby, auto-sync)  │
│   ↓                            │
│ S3 (Cross-region replication)  │
└────────────────────────────────┘
         ↓ (continuous replication)
DR Region (us-west-2):
┌────────────────────────────────┐
│ ALB (Standby)                  │
│   ↓                            │
│ ECS (0 tasks, auto-scale ready)│
│   ↓                            │
│ RDS Read Replica (async)       │
│   ↓                            │
│ S3 (Replicated)                │
└────────────────────────────────┘
```

**RTO and RPO:**
- **RPO (Recovery Point Objective):** 5 minutes
  - RDS replication lag < 5 min
  - Max data loss: 5 minutes

- **RTO (Recovery Time Objective):** 15 minutes
  - Scale up DR region
  - Promote read replica to primary
  - Update Route 53 DNS

**Implementation:**

1. **Multi-AZ (within region):**
```bash
# RDS Multi-AZ (automatic failover)
aws rds create-db-instance \
  --multi-az \  # Standby in different AZ
  --backup-retention-period 7

# If AZ-1a fails → Auto-failover to AZ-1b (1-2 min)
```

2. **Cross-region replication:**
```bash
# RDS Cross-region Read Replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier school-admin-db-dr \
  --source-db-instance-identifier school-admin-db \
  --source-region us-east-1 \
  --region us-west-2

# S3 Cross-region replication
aws s3api put-bucket-replication \
  --bucket source-bucket \
  --replication-configuration '{
    \"Role\": \"arn:aws:iam::123:role/s3-replication\",
    \"Rules\": [{
      \"Status\": \"Enabled\",
      \"Priority\": 1,
      \"Destination\": {
        \"Bucket\": \"arn:aws:s3:::dr-bucket\",
        \"ReplicationTime\": {
          \"Status\": \"Enabled\",
          \"Time\": { \"Minutes\": 15 }
        }
      }
    }]
  }'
```

3. **Automated backups:**
```bash
# RDS automated backups (7 days)
# Daily snapshots at 3:00 AM
# 5-minute transaction logs

# Manual snapshot before major changes
aws rds create-db-snapshot \
  --db-instance-identifier school-admin-db \
  --db-snapshot-identifier pre-migration-snapshot
```

4. **Route 53 health checks + failover:**
```json
{
  \"HealthCheck\": {
    \"Type\": \"HTTPS\",
    \"ResourcePath\": \"/health\",
    \"FullyQualifiedDomainName\": \"api.school-admin.com\",
    \"RequestInterval\": 30,
    \"FailureThreshold\": 3
  },

  \"RecordSet\": {
    \"Name\": \"api.school-admin.com\",
    \"Type\": \"A\",
    \"SetIdentifier\": \"Primary\",
    \"Failover\": \"PRIMARY\",
    \"AliasTarget\": {
      \"DNSName\": \"us-east-1-alb.amazonaws.com\",
      \"EvaluateTargetHealth\": true
    }
  },

  \"RecordSetDR\": {
    \"Name\": \"api.school-admin.com\",
    \"Type\": \"A\",
    \"SetIdentifier\": \"DR\",
    \"Failover\": \"SECONDARY\",
    \"AliasTarget\": {
      \"DNSName\": \"us-west-2-alb.amazonaws.com\",
      \"EvaluateTargetHealth\": true
    }
  }
}
```

**Disaster scenarios:**

**Scenario 1: AZ failure (e.g., AZ-1a down)**
- **Impact:** 50% capacity lost
- **Recovery:** Automatic (RDS failover, Auto Scaling launches new instances in AZ-1b)
- **Time:** 2-5 minutes
- **Data loss:** None

**Scenario 2: Region failure (e.g., us-east-1 down)**
- **Impact:** Entire primary region down
- **Recovery steps:**
  1. Promote us-west-2 read replica to primary (5 min)
  2. Scale up ECS tasks in us-west-2 (5 min)
  3. Route 53 health check fails → Auto-failover to DR (5 min)
- **Time:** 15 minutes
- **Data loss:** < 5 minutes (replication lag)

**Testing:**
- Monthly DR drill (failover to DR region)
- Verify RTO/RPO targets met
- Document lessons learned

**Cost:**
- Multi-AZ: +100% database cost ($100 → $200/month)
- Cross-region replica: +50% database cost (+$100/month)
- S3 replication: +20% storage cost (+$5/month)
- **Total DR cost:** +$205/month
- **vs downtime cost:** 1 hour downtime = $10,000 lost revenue

Worth it? **Yes!**"

---

## Scenario-Based Questions

### Q5: Design a system to process 1 million CSV uploads per day

**Answer:**

"**Requirements:**
- 1M CSV files/day (~12 files/second)
- Each file: 100-1000 rows
- Process: Validate → Transform → Store in database
- Constraints: < $100/month, < 5 min processing time

**Architecture:**

```
User Upload → S3 → Lambda (validate) → SQS → Lambda (process) → DynamoDB

┌──────────┐
│  Client  │
└─────┬────┘
      │ Upload CSV
      ▼
┌──────────────┐
│      S3      │
│  (Uploads)   │
└──────┬───────┘
       │ S3 Event
       ▼
┌──────────────┐
│   Lambda     │ ← Triggered by S3
│  (Validate)  │
└──────┬───────┘
       │ If valid
       ▼
┌──────────────┐
│     SQS      │ ← Queue for batch processing
│   (Queue)    │
└──────┬───────┘
       │ Batch (10 messages)
       ▼
┌──────────────┐
│   Lambda     │ ← Process in parallel
│  (Process)   │    (up to 1000 concurrent)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  DynamoDB    │
│  (Results)   │
└──────────────┘
```

**Implementation:**

```typescript
// Step 1: S3 upload with pre-signed URL
app.post('/api/upload-url', async (req, res) => {
  const { filename } = req.body;

  const url = s3.getSignedUrl('putObject', {
    Bucket: 'csv-uploads',
    Key: `uploads/${Date.now()}-${filename}`,
    Expires: 300,  // 5 minutes
    ContentType: 'text/csv'
  });

  res.json({ uploadUrl: url });
});

// Client uploads directly to S3 (no server involved)
await fetch(uploadUrl, {
  method: 'PUT',
  body: csvFile,
  headers: { 'Content-Type': 'text/csv' }
});

// Step 2: Lambda validates CSV (S3 trigger)
export const validateCSV = async (event: any) => {
  const bucket = event.Records[0].s3.bucket.name;
  const key = event.Records[0].s3.object.key;

  // Download CSV
  const csv = await s3.getObject({ Bucket: bucket, Key: key }).promise();
  const rows = parseCSV(csv.Body.toString());

  // Validate
  const errors = validateRows(rows);

  if (errors.length > 0) {
    // Store errors in DynamoDB
    await dynamodb.put({
      TableName: 'ValidationErrors',
      Item: {
        fileKey: key,
        errors,
        status: 'FAILED'
      }
    }).promise();

    // Notify user
    await sns.publish({
      TopicArn: process.env.ERROR_TOPIC,
      Message: JSON.stringify({ fileKey: key, errors })
    }).promise();

    return;
  }

  // Valid → Send to SQS for processing
  for (let i = 0; i < rows.length; i += 10) {
    const batch = rows.slice(i, i + 10);

    await sqs.sendMessageBatch({
      QueueUrl: process.env.PROCESS_QUEUE_URL,
      Entries: batch.map((row, idx) => ({
        Id: `${i + idx}`,
        MessageBody: JSON.stringify({ fileKey: key, row })
      }))
    }).promise();
  }

  console.log(`Queued ${rows.length} rows from ${key}`);
};

// Step 3: Lambda processes rows (SQS trigger)
export const processRows = async (event: any) => {
  const writes = [];

  for (const record of event.Records) {
    const { fileKey, row } = JSON.parse(record.body);

    // Transform and store in DynamoDB
    writes.push(
      dynamodb.put({
        TableName: 'Students',
        Item: {
          id: uuid(),
          ...transformRow(row),
          importedFrom: fileKey,
          importedAt: new Date().toISOString()
        }
      }).promise()
    );
  }

  await Promise.all(writes);
};

// Step 4: Cleanup (delete old files after 7 days)
// S3 Lifecycle Policy
{
  \"Rules\": [{
    \"Id\": \"DeleteOldUploads\",
    \"Status\": \"Enabled\",
    \"Expiration\": { \"Days\": 7 }
  }]
}
```

**Scaling:**

```
1M files/day = 12 files/second

Lambda concurrency:
- Validate: 12 concurrent (1 per file)
- Process: 1200 concurrent (100 rows/file, 10 parallel)

SQS throughput: 3000 messages/sec (plenty)
DynamoDB write capacity: 120,000 WCU (auto-scaled)

Cost (1M files/day):
- S3 storage (7 days): $5/month
- Lambda invocations: $20/month
- SQS: $5/month
- DynamoDB: $30/month
Total: $60/month ✅ Under budget
```

**Monitoring:**

```typescript
// CloudWatch metrics
await cloudwatch.putMetricData({
  Namespace: 'CSVProcessing',
  MetricData: [{
    MetricName: 'FilesProcessed',
    Value: 1,
    Unit: 'Count'
  }, {
    MetricName: 'RowsProcessed',
    Value: rows.length,
    Unit: 'Count'
  }, {
    MetricName: 'ProcessingTime',
    Value: duration,
    Unit: 'Milliseconds'
  }]
}).promise();
```

**Result:**
✅ Handles 1M files/day
✅ < 5 min processing (parallel Lambda)
✅ < $100/month
✅ Auto-scales
✅ Fault-tolerant (SQS retries)"

---

## Best Practices Summary

### Cost Optimization Checklist

```
☐ Right-size resources
  ☐ Use smallest instance type that meets needs
  ☐ Monitor actual usage (CloudWatch)

☐ Use Reserved Instances / Savings Plans
  ☐ For predictable workloads (40-75% discount)

☐ Use Spot Instances
  ☐ For fault-tolerant workloads (up to 90% discount)

☐ Implement Auto Scaling
  ☐ Scale down when not needed

☐ Use S3 Lifecycle Policies
  ☐ Move old data to cheaper storage classes

☐ Enable CloudFront
  ☐ Reduce S3 transfer costs (80-90% savings)

☐ Delete unused resources
  ☐ Old snapshots, unused volumes, idle load balancers

☐ Set budget alerts
  ☐ Get notified before overspending
```

### Security Checklist

```
☐ Enable MFA on root account
☐ Use IAM roles (not access keys)
☐ Principle of least privilege
☐ Enable CloudTrail (audit logging)
☐ Encrypt data at rest (S3, RDS, EBS)
☐ Encrypt data in transit (HTTPS, SSL/TLS)
☐ Use Secrets Manager (no hardcoded secrets)
☐ Enable GuardDuty (threat detection)
☐ Use VPC (network isolation)
☐ Security Groups (allow only necessary traffic)
☐ Regular security audits (AWS Trusted Advisor)
```

### Reliability Checklist

```
☐ Multi-AZ deployment
  ☐ RDS Multi-AZ
  ☐ EC2 in multiple AZs

☐ Auto Scaling
  ☐ Handle traffic spikes
  ☐ Replace failed instances

☐ Load Balancing
  ☐ Health checks
  ☐ Distribute traffic

☐ Automated backups
  ☐ RDS (7+ days)
  ☐ S3 versioning

☐ Disaster recovery plan
  ☐ Backup and restore
  ☐ Cross-region replication

☐ Monitoring and alerting
  ☐ CloudWatch alarms
  ☐ SNS notifications
```

### Performance Checklist

```
☐ Use caching
  ☐ ElastiCache (Redis)
  ☐ CloudFront (CDN)
  ☐ API Gateway cache

☐ Use CDN (CloudFront)
  ☐ Static assets
  ☐ API responses

☐ Optimize database
  ☐ Indexes
  ☐ Read replicas
  ☐ Connection pooling

☐ Use Auto Scaling
  ☐ Scale based on demand

☐ Asynchronous processing
  ☐ SQS + Lambda
  ☐ Don't block user requests
```

---

## AWS Certification Relevance

**For AWS Certified Solutions Architect - Associate:**

Topics covered that align with exam:
- ✅ EC2, Lambda, ECS, Fargate
- ✅ S3, EBS, EFS storage classes
- ✅ RDS, DynamoDB, Aurora
- ✅ VPC, Security Groups, NACLs
- ✅ ALB, NLB, CloudFront
- ✅ IAM, Secrets Manager
- ✅ CloudWatch, CloudTrail
- ✅ Well-Architected Framework
- ✅ Disaster recovery strategies
- ✅ Cost optimization

**Study tips:**
1. Hands-on practice (free tier)
2. Understand trade-offs (when to use what)
3. Focus on scenarios (not memorization)
4. Know pricing models

---

## Final Interview Tips

**When asked "How would you...?":**

1. **Clarify requirements:**
   - Scale? (100 users vs 1M users)
   - Budget?
   - Compliance?
   - Performance requirements?

2. **Start simple:**
   - Don't over-engineer
   - Add complexity only when needed

3. **Explain trade-offs:**
   - "I chose Lambda over EC2 because..."
   - "The downside is cold starts, but..."

4. **Show cost awareness:**
   - Estimate costs
   - Mention optimizations

5. **Security first:**
   - Always mention encryption, IAM, VPC
   - "Data encrypted at rest and in transit"

6. **Think operational:**
   - Monitoring
   - Backups
   - Disaster recovery

**Example structure:**

```
"For your school admin system, I would recommend:

1. Requirements: 1000 students, < $300/month, 99.9% availability

2. Architecture:
   - Frontend: S3 + CloudFront
   - Backend: ECS Fargate (Auto Scaling)
   - Database: RDS MySQL (Multi-AZ)
   - Cache: ElastiCache Redis

3. Why this architecture:
   - Cost-effective (~$300/month)
   - Auto-scales (handle traffic spikes)
   - High availability (Multi-AZ)
   - No server management (ECS Fargate)

4. Trade-offs:
   - Could use Lambda (cheaper) but chose ECS for longer request times
   - Could use DynamoDB (faster) but chose RDS for existing MySQL schema

5. Security:
   - VPC with private subnets
   - IAM roles (no access keys)
   - Secrets Manager for passwords
   - HTTPS only

6. Monitoring:
   - CloudWatch metrics + alarms
   - X-Ray tracing
   - Cost budgets

Would you like me to dive deeper into any component?"
```

**Remember:** Interviewers want to see:
- ✅ Problem-solving approach
- ✅ Trade-off analysis
- ✅ Cost awareness
- ✅ Security mindset
- ✅ Real-world experience

Good luck with your interview! 🚀
