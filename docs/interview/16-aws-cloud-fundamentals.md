# AWS & Cloud Computing Fundamentals

## Table of Contents
1. [Cloud Computing Basics](#cloud-computing-basics)
2. [AWS Global Infrastructure](#aws-global-infrastructure)
3. [Core AWS Services Overview](#core-aws-services-overview)
4. [Compute Services](#compute-services)
5. [Storage Services](#storage-services)
6. [Database Services](#database-services)
7. [Networking & Content Delivery](#networking--content-delivery)
8. [Security & Identity](#security--identity)
9. [Monitoring & Management](#monitoring--management)
10. [Serverless Architecture](#serverless-architecture)
11. [Cost Optimization](#cost-optimization)
12. [Common Interview Questions](#common-interview-questions)

---

## Cloud Computing Basics

### What is Cloud Computing?

**Cloud Computing:** On-demand delivery of IT resources over the internet with pay-as-you-go pricing

**Traditional IT:**
```
Buy Server ($10,000) → Install → Maintain → Pay electricity
- High upfront cost
- Capacity planning difficult
- Unused capacity waste
- Maintenance overhead
```

**Cloud Computing:**
```
Rent Server ($50/month) → Use immediately → Scale up/down
- No upfront cost
- Pay only for what you use
- Scale instantly
- No maintenance
```

### Cloud Service Models

#### 1. IaaS (Infrastructure as a Service)

```
You Manage:
├── Applications
├── Data
├── Runtime
├── Middleware
└── Operating System

Cloud Provider Manages:
├── Virtualization
├── Servers
├── Storage
└── Networking

Examples: AWS EC2, Google Compute Engine, Azure VMs
```

**Use Case:** Full control over infrastructure
**Your Responsibility:** Everything except hardware

#### 2. PaaS (Platform as a Service)

```
You Manage:
├── Applications
└── Data

Cloud Provider Manages:
├── Runtime
├── Middleware
├── Operating System
├── Virtualization
├── Servers
├── Storage
└── Networking

Examples: AWS Elastic Beanstalk, Google App Engine, Heroku
```

**Use Case:** Focus on application, not infrastructure
**Your Responsibility:** Just code and data

#### 3. SaaS (Software as a Service)

```
Cloud Provider Manages:
└── Everything (including application)

Examples: Gmail, Salesforce, Dropbox, Slack
```

**Use Case:** Ready-to-use applications
**Your Responsibility:** Just use it

### Cloud Deployment Models

#### 1. Public Cloud

```
┌─────────────────────────────────┐
│      AWS / Azure / GCP          │
│  (Shared infrastructure)        │
│                                 │
│  ┌─────┐ ┌─────┐ ┌─────┐       │
│  │You  │ │Other│ │Other│       │
│  │     │ │Users│ │Users│       │
│  └─────┘ └─────┘ └─────┘       │
└─────────────────────────────────┘
```

**Pros:**
- ✅ No upfront cost
- ✅ High scalability
- ✅ Pay-as-you-go

**Cons:**
- ❌ Less control
- ❌ Security concerns (shared)

#### 2. Private Cloud

```
┌─────────────────────────────────┐
│   Your Company's Data Center    │
│  (Dedicated infrastructure)     │
│                                 │
│  ┌───────────────────────────┐ │
│  │   Only Your Resources     │ │
│  │                           │ │
│  └───────────────────────────┘ │
└─────────────────────────────────┘
```

**Pros:**
- ✅ Full control
- ✅ Better security
- ✅ Compliance (HIPAA, etc.)

**Cons:**
- ❌ High upfront cost
- ❌ Limited scalability
- ❌ Maintenance required

#### 3. Hybrid Cloud

```
┌──────────────┐       ┌──────────────┐
│ Private Cloud│◄─────►│ Public Cloud │
│ (Sensitive   │       │ (General     │
│  data)       │       │  workloads)  │
└──────────────┘       └──────────────┘
```

**Use Case:** Keep sensitive data on-premise, burst to cloud for peak loads

### Why AWS?

**Market Share (2024):**
```
AWS:    32%  (Leader)
Azure:  23%
GCP:    10%
Others: 35%
```

**Advantages:**
- ✅ Largest ecosystem (200+ services)
- ✅ Most mature (launched 2006)
- ✅ Extensive documentation
- ✅ Large community
- ✅ Global presence (30+ regions)

---

## AWS Global Infrastructure

### Regions

**Region:** Geographic area with multiple data centers

```
┌────────────────────────────────────┐
│         AWS Region (us-east-1)     │
│                                    │
│  ┌──────────┐  ┌──────────┐       │
│  │   AZ-1   │  │   AZ-2   │       │
│  │ (Data    │  │ (Data    │       │
│  │  Center) │  │  Center) │       │
│  └──────────┘  └──────────┘       │
└────────────────────────────────────┘
```

**Popular Regions:**
- `us-east-1`: N. Virginia (oldest, most services)
- `us-west-2`: Oregon
- `eu-west-1`: Ireland
- `ap-southeast-1`: Singapore
- `ap-northeast-1`: Tokyo

**How to Choose Region:**
1. **Latency:** Choose closest to users
2. **Compliance:** GDPR (EU), data residency laws
3. **Cost:** Pricing varies by region
4. **Service Availability:** Not all services in all regions

### Availability Zones (AZs)

**AZ:** One or more discrete data centers with redundant power, networking

```
Region: us-east-1
├── AZ: us-east-1a (Data Center 1)
├── AZ: us-east-1b (Data Center 2)
├── AZ: us-east-1c (Data Center 3)
└── AZ: us-east-1d (Data Center 4)

Each AZ is isolated for fault tolerance
Connected via high-speed, low-latency network
```

**Best Practice:** Deploy across multiple AZs for high availability

```
┌─────────────────────────────────┐
│         us-east-1               │
│                                 │
│  ┌──────────┐  ┌──────────┐    │
│  │  AZ-1a   │  │  AZ-1b   │    │
│  │          │  │          │    │
│  │ ┌──────┐ │  │ ┌──────┐ │    │
│  │ │App   │ │  │ │App   │ │    │
│  │ │Server│ │  │ │Server│ │    │
│  │ └──────┘ │  │ └──────┘ │    │
│  │          │  │          │    │
│  │ ┌──────┐ │  │ ┌──────┐ │    │
│  │ │DB    │←┼──┼→│DB    │ │    │
│  │ │Primary│ │ │ │Standby│ │   │
│  │ └──────┘ │  │ └──────┘ │    │
│  └──────────┘  └──────────┘    │
└─────────────────────────────────┘

If AZ-1a fails, AZ-1b continues serving
```

### Edge Locations

**Edge Location:** CDN endpoints for CloudFront

```
User (Japan) → Edge Location (Tokyo) → S3 Bucket (US)
                       ↓
                   Cached content
                   (Fast delivery)

200+ Edge Locations globally
```

---

## Core AWS Services Overview

### AWS Services by Category

```
Compute:
├── EC2 (Virtual Servers)
├── Lambda (Serverless Functions)
├── ECS (Container Orchestration)
├── EKS (Kubernetes)
└── Fargate (Serverless Containers)

Storage:
├── S3 (Object Storage)
├── EBS (Block Storage)
├── EFS (File Storage)
└── Glacier (Archive Storage)

Database:
├── RDS (Relational)
├── DynamoDB (NoSQL Key-Value)
├── Aurora (High-Performance MySQL/PostgreSQL)
├── ElastiCache (In-Memory Cache)
└── DocumentDB (MongoDB-compatible)

Networking:
├── VPC (Virtual Private Cloud)
├── CloudFront (CDN)
├── Route 53 (DNS)
├── ELB (Load Balancer)
└── API Gateway (API Management)

Security:
├── IAM (Identity & Access Management)
├── Cognito (User Authentication)
├── Secrets Manager
└── WAF (Web Application Firewall)

Monitoring:
├── CloudWatch (Metrics & Logs)
├── X-Ray (Distributed Tracing)
└── CloudTrail (Audit Logs)

DevOps:
├── CodePipeline (CI/CD)
├── CodeBuild (Build Service)
├── CodeDeploy (Deployment)
└── CloudFormation (Infrastructure as Code)

Messaging:
├── SQS (Message Queue)
├── SNS (Pub/Sub)
└── EventBridge (Event Bus)
```

---

## Compute Services

### 1. EC2 (Elastic Compute Cloud)

**What:** Virtual servers in the cloud

#### Instance Types

```
General Purpose (T3, M5):
- Balanced CPU, memory, network
- Use: Web servers, small databases
- Example: t3.medium (2 vCPU, 4GB RAM)

Compute Optimized (C5):
- High CPU performance
- Use: Video encoding, scientific modeling
- Example: c5.xlarge (4 vCPU, 8GB RAM)

Memory Optimized (R5):
- High RAM
- Use: In-memory databases, big data
- Example: r5.large (2 vCPU, 16GB RAM)

Storage Optimized (I3):
- High disk I/O
- Use: NoSQL databases, data warehousing
- Example: i3.large (2 vCPU, 15GB RAM, 475GB SSD)

GPU Instances (P3):
- Graphics processing
- Use: Machine learning, rendering
```

#### Pricing Models

**1. On-Demand**
```
Pay per hour/second
No commitment
Use: Short-term, unpredictable workloads

Price: $0.0832/hour (t3.medium)
```

**2. Reserved Instances**
```
1 or 3 year commitment
Up to 75% discount
Use: Steady-state workloads

Price: $0.0499/hour (t3.medium, 1-year)
Savings: 40% vs On-Demand
```

**3. Spot Instances**
```
Bid on spare capacity
Up to 90% discount
Can be terminated anytime
Use: Fault-tolerant, flexible workloads

Price: $0.0250/hour (t3.medium)
Savings: 70% vs On-Demand
```

**4. Savings Plans**
```
Commit to $ amount per hour (e.g., $10/hour)
Up to 72% discount
Flexible (any instance type, region)
```

#### Example: Deploying Node.js App on EC2

```bash
# 1. Launch EC2 instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \  # Amazon Linux 2
  --instance-type t3.medium \
  --key-name my-key-pair \
  --security-group-ids sg-12345678

# 2. SSH into instance
ssh -i my-key-pair.pem ec2-user@ec2-54-123-45-67.compute-1.amazonaws.com

# 3. Install Node.js
sudo yum update -y
sudo yum install -y nodejs npm

# 4. Deploy application
git clone https://github.com/user/school-admin-system.git
cd school-admin-system/typescript
npm install
npm run build

# 5. Start application (with PM2 for auto-restart)
sudo npm install -g pm2
pm2 start dist/server.js
pm2 startup  # Auto-start on reboot
pm2 save
```

#### Auto Scaling

```
┌────────────────────────────────────┐
│      Auto Scaling Group            │
│                                    │
│  Min: 2 instances                  │
│  Desired: 3 instances              │
│  Max: 10 instances                 │
│                                    │
│  ┌─────┐ ┌─────┐ ┌─────┐          │
│  │ EC2 │ │ EC2 │ │ EC2 │          │
│  │  1  │ │  2  │ │  3  │          │
│  └─────┘ └─────┘ └─────┘          │
│                                    │
│  When CPU > 70% → Add instance     │
│  When CPU < 30% → Remove instance  │
└────────────────────────────────────┘
```

**Configuration:**

```javascript
// CloudFormation template
{
  "AutoScalingGroup": {
    "Type": "AWS::AutoScaling::AutoScalingGroup",
    "Properties": {
      "MinSize": "2",
      "MaxSize": "10",
      "DesiredCapacity": "3",
      "LaunchTemplate": {
        "LaunchTemplateId": "lt-12345678",
        "Version": "$Latest"
      },
      "TargetGroupARNs": ["arn:aws:elasticloadbalancing:..."],
      "HealthCheckType": "ELB",
      "HealthCheckGracePeriod": 300
    }
  },
  "ScaleUpPolicy": {
    "Type": "AWS::AutoScaling::ScalingPolicy",
    "Properties": {
      "AdjustmentType": "ChangeInCapacity",
      "AutoScalingGroupName": { "Ref": "AutoScalingGroup" },
      "Cooldown": "60",
      "ScalingAdjustment": "1"  // Add 1 instance
    }
  }
}
```

### 2. Lambda (Serverless)

**What:** Run code without managing servers (pay per execution)

**Characteristics:**
- ✅ No server management
- ✅ Auto-scaling (handles millions of requests)
- ✅ Pay only for execution time
- ✅ Event-driven (triggers)
- ❌ 15-minute max execution time
- ❌ Cold start latency (100-1000ms)

**Pricing:**
```
$0.20 per 1 million requests
$0.0000166667 per GB-second

Example: 1M requests/month, 512MB, 1s execution
Requests: 1M × $0.20/1M = $0.20
Compute: 1M × 0.5GB × 1s × $0.0000166667 = $8.33
Total: $8.53/month

vs EC2 t3.small (2GB RAM) = $15/month (runs 24/7)
```

**Example: Image Thumbnail Generator**

```typescript
// lambda/thumbnail.ts
import AWS from 'aws-sdk';
import sharp from 'sharp';

const s3 = new AWS.S3();

export const handler = async (event: any) => {
  // Triggered when image uploaded to S3
  const bucket = event.Records[0].s3.bucket.name;
  const key = event.Records[0].s3.object.key;

  // Download image
  const image = await s3.getObject({ Bucket: bucket, Key: key }).promise();

  // Create thumbnail (200x200)
  const thumbnail = await sharp(image.Body as Buffer)
    .resize(200, 200)
    .toBuffer();

  // Upload thumbnail
  await s3.putObject({
    Bucket: bucket,
    Key: `thumbnails/${key}`,
    Body: thumbnail,
    ContentType: 'image/jpeg'
  }).promise();

  return { statusCode: 200, body: 'Thumbnail created' };
};
```

**Deploy:**

```bash
# Package Lambda function
zip -r function.zip index.js node_modules/

# Create Lambda function
aws lambda create-function \
  --function-name thumbnail-generator \
  --runtime nodejs18.x \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler index.handler \
  --zip-file fileb://function.zip \
  --memory-size 512 \
  --timeout 60

# Add S3 trigger
aws lambda add-permission \
  --function-name thumbnail-generator \
  --statement-id s3-trigger \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-bucket
```

**Use Cases:**
- ✅ API backends (with API Gateway)
- ✅ Event processing (S3 uploads, DynamoDB changes)
- ✅ Scheduled tasks (cron jobs)
- ✅ Data transformation (ETL)
- ❌ Long-running tasks (> 15 min)
- ❌ Stateful applications

### 3. ECS (Elastic Container Service)

**What:** Docker container orchestration

```
┌────────────────────────────────────┐
│         ECS Cluster                │
│                                    │
│  ┌──────────────────────────────┐ │
│  │   Task Definition            │ │
│  │   (Blueprint)                │ │
│  │   - Image: app:latest        │ │
│  │   - CPU: 512                 │ │
│  │   - Memory: 1024             │ │
│  └──────────────────────────────┘ │
│                                    │
│  ┌─────────┐  ┌─────────┐         │
│  │ Task 1  │  │ Task 2  │         │
│  │ ┌─────┐ │  │ ┌─────┐ │         │
│  │ │App  │ │  │ │App  │ │         │
│  │ │Cont.│ │  │ │Cont.│ │         │
│  │ └─────┘ │  │ └─────┘ │         │
│  └─────────┘  └─────────┘         │
└────────────────────────────────────┘
```

**Launch Types:**

**1. EC2 Launch Type:**
- You manage EC2 instances
- More control
- Cheaper

**2. Fargate Launch Type:**
- AWS manages infrastructure
- Serverless containers
- Easier, more expensive

**Example: Deploy School Admin System**

```json
// task-definition.json
{
  "family": "school-admin",
  "containerDefinitions": [
    {
      "name": "api",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/school-admin:latest",
      "cpu": 512,
      "memory": 1024,
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        { "name": "NODE_ENV", "value": "production" },
        { "name": "DB_HOST", "value": "rds-instance.us-east-1.rds.amazonaws.com" }
      ],
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789012:secret:db-password"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/school-admin",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "api"
        }
      }
    }
  ],
  "requiresCompatibilities": ["FARGATE"],
  "networkMode": "awsvpc",
  "cpu": "512",
  "memory": "1024"
}
```

```bash
# Register task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# Create service
aws ecs create-service \
  --cluster school-admin-cluster \
  --service-name api-service \
  --task-definition school-admin:1 \
  --desired-count 3 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-12345],securityGroups=[sg-12345],assignPublicIp=ENABLED}" \
  --load-balancers "targetGroupArn=arn:aws:elasticloadbalancing:...,containerName=api,containerPort=3000"
```

### 4. EKS (Elastic Kubernetes Service)

**What:** Managed Kubernetes

**Use When:**
- Already using Kubernetes
- Multi-cloud strategy
- Need Kubernetes features

**ECS vs EKS:**
```
ECS:
✅ Simpler
✅ Better AWS integration
✅ Cheaper
❌ AWS-specific

EKS:
✅ Standard Kubernetes
✅ Multi-cloud portability
✅ Larger ecosystem
❌ More complex
❌ More expensive
```

---

## Storage Services

### 1. S3 (Simple Storage Service)

**What:** Object storage (files, images, videos, backups)

**Characteristics:**
- ✅ Unlimited storage
- ✅ Highly durable (99.999999999% - 11 nines)
- ✅ Highly available (99.99%)
- ✅ Scalable
- ✅ Pay per GB stored + transfer

**Concepts:**

```
Bucket: Container for objects (globally unique name)
└── Object: File (up to 5TB)
    ├── Key: File path (e.g., "photos/2024/img.jpg")
    ├── Value: File content
    ├── Metadata: Content-Type, etc.
    └── Permissions: Public/Private
```

**Storage Classes:**

```
┌──────────────────────────────────────────────────┐
│ Storage Class      │ Use Case        │ Cost      │
├──────────────────────────────────────────────────┤
│ S3 Standard        │ Frequently      │ $$$$      │
│                    │ accessed        │           │
├──────────────────────────────────────────────────┤
│ S3 Intelligent-    │ Unknown access  │ $$$       │
│ Tiering            │ patterns        │ (Auto)    │
├──────────────────────────────────────────────────┤
│ S3 Standard-IA     │ Infrequent      │ $$        │
│ (Infrequent Access)│ access          │           │
├──────────────────────────────────────────────────┤
│ S3 One Zone-IA     │ Non-critical    │ $         │
│                    │ infrequent      │           │
├──────────────────────────────────────────────────┤
│ S3 Glacier         │ Archive         │ ¢         │
│                    │ (mins-hours)    │           │
├──────────────────────────────────────────────────┤
│ S3 Glacier Deep    │ Long-term       │ ¢¢        │
│ Archive            │ (12 hours)      │           │
└──────────────────────────────────────────────────┘
```

**Example: File Upload**

```typescript
import AWS from 'aws-sdk';
import multer from 'multer';
import multerS3 from 'multer-s3';

const s3 = new AWS.S3();

// Direct upload to S3 (no local storage)
const upload = multer({
  storage: multerS3({
    s3: s3,
    bucket: 'school-admin-uploads',
    acl: 'private',  // or 'public-read'
    metadata: (req, file, cb) => {
      cb(null, { uploadedBy: req.user.id });
    },
    key: (req, file, cb) => {
      const filename = `${Date.now()}-${file.originalname}`;
      cb(null, `uploads/${filename}`);
    }
  })
});

app.post('/upload', upload.single('file'), (req, res) => {
  res.json({
    url: req.file.location,  // S3 URL
    key: req.file.key
  });
});

// Download file
app.get('/download/:key', async (req, res) => {
  const params = {
    Bucket: 'school-admin-uploads',
    Key: req.params.key
  };

  const stream = s3.getObject(params).createReadStream();
  stream.pipe(res);
});

// Generate pre-signed URL (temporary access)
app.get('/presigned-url/:key', async (req, res) => {
  const url = s3.getSignedUrl('getObject', {
    Bucket: 'school-admin-uploads',
    Key: req.params.key,
    Expires: 3600  // 1 hour
  });

  res.json({ url });
});
```

**S3 Lifecycle Policies:**

```json
{
  "Rules": [
    {
      "Id": "Move old files to Glacier",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"  // After 30 days
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"  // After 90 days
        }
      ],
      "Expiration": {
        "Days": 365  // Delete after 1 year
      }
    }
  ]
}
```

### 2. EBS (Elastic Block Store)

**What:** Block storage for EC2 (like a hard drive)

**Characteristics:**
- Attached to single EC2 instance
- Persists beyond instance lifecycle
- Can snapshot for backup

**Volume Types:**

```
General Purpose SSD (gp3):
- Balanced price/performance
- 3,000-16,000 IOPS
- Use: Boot volumes, dev/test

Provisioned IOPS SSD (io2):
- High performance
- Up to 64,000 IOPS
- Use: Databases, critical workloads

Throughput Optimized HDD (st1):
- Low-cost HDD
- High throughput
- Use: Big data, data warehouses

Cold HDD (sc1):
- Lowest cost
- Infrequent access
- Use: Archives
```

**Example:**

```bash
# Create EBS volume
aws ec2 create-volume \
  --volume-type gp3 \
  --size 100 \  # 100 GB
  --availability-zone us-east-1a

# Attach to EC2 instance
aws ec2 attach-volume \
  --volume-id vol-12345678 \
  --instance-id i-12345678 \
  --device /dev/sdf

# SSH to instance and mount
sudo mkfs -t ext4 /dev/sdf
sudo mkdir /data
sudo mount /dev/sdf /data
```

### 3. EFS (Elastic File System)

**What:** Network file system (shared storage for multiple EC2 instances)

```
┌─────────┐  ┌─────────┐  ┌─────────┐
│  EC2 1  │  │  EC2 2  │  │  EC2 3  │
└────┬────┘  └────┬────┘  └────┬────┘
     │            │            │
     └────────────┼────────────┘
                  │
            ┌─────┴─────┐
            │    EFS    │
            │ (Shared)  │
            └───────────┘
```

**Use Cases:**
- Content management (WordPress)
- Shared development environment
- Home directories

**EBS vs EFS:**

```
EBS:
- Single EC2 instance
- Higher performance
- Lower cost

EFS:
- Multiple EC2 instances
- Shared storage
- Higher cost
```

---

## Database Services

### 1. RDS (Relational Database Service)

**What:** Managed MySQL, PostgreSQL, MariaDB, Oracle, SQL Server

**Benefits:**
- ✅ Automated backups
- ✅ Automated patching
- ✅ Multi-AZ (high availability)
- ✅ Read replicas (scalability)
- ✅ Monitoring (CloudWatch)

**Example: MySQL for School Admin System**

```typescript
// Configure RDS connection
import { Sequelize } from 'sequelize';

const sequelize = new Sequelize({
  dialect: 'mysql',
  host: process.env.RDS_HOSTNAME,  // mydb.c9akciq32.us-east-1.rds.amazonaws.com
  port: 3306,
  username: process.env.RDS_USERNAME,
  password: process.env.RDS_PASSWORD,
  database: process.env.RDS_DB_NAME,
  pool: {
    max: 10,
    min: 1,
    acquire: 30000,
    idle: 10000
  },
  logging: false
});
```

```bash
# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier school-admin-db \
  --db-instance-class db.t3.medium \
  --engine mysql \
  --engine-version 8.0.33 \
  --master-username admin \
  --master-user-password MySecurePassword123 \
  --allocated-storage 100 \  # 100 GB
  --storage-type gp3 \
  --multi-az \  # High availability (standby in another AZ)
  --backup-retention-period 7 \  # 7 days
  --preferred-backup-window "03:00-04:00" \
  --vpc-security-group-ids sg-12345678

# Create read replica (for read scaling)
aws rds create-db-instance-read-replica \
  --db-instance-identifier school-admin-db-replica \
  --source-db-instance-identifier school-admin-db \
  --db-instance-class db.t3.medium
```

**Multi-AZ (High Availability):**

```
┌───────────────────────────────────────┐
│         RDS Multi-AZ                  │
│                                       │
│  ┌──────────┐          ┌──────────┐  │
│  │   AZ-1a  │          │   AZ-1b  │  │
│  │          │          │          │  │
│  │ ┌──────┐ │  Sync    │ ┌──────┐ │  │
│  │ │Primary│◄┼─────────┼─┤Standby│ │ │
│  │ │ DB    │ │  Repl.  │ │  DB   │ │ │
│  │ └───┬──┘ │          │ └──────┘ │  │
│  └─────┼────┘          └──────────┘  │
│        │                              │
│   ┌────┴────┐                         │
│   │ Clients │                         │
│   └─────────┘                         │
│                                       │
│  If Primary fails → Auto-failover     │
│  to Standby (1-2 min)                 │
└───────────────────────────────────────┘
```

**Read Replicas (Scalability):**

```
┌────────────┐
│  Primary   │ ← Writes
│     DB     │
└──────┬─────┘
       │ Async Replication
  ┌────┼────┬────┐
  │    │    │    │
┌─┴─┐ ┌┴──┐ ┌┴──┐
│R1 │ │R2 │ │R3 │ ← Reads
└───┘ └───┘ └───┘

Up to 15 read replicas
```

### 2. DynamoDB

**What:** Managed NoSQL key-value database

**Characteristics:**
- ✅ Serverless (no infrastructure)
- ✅ Single-digit millisecond latency
- ✅ Auto-scaling
- ✅ Global tables (multi-region)
- ✅ Pay per request or provisioned capacity

**Data Model:**

```
Table: Students
┌──────────────┬──────────┬────────────────────────┐
│ email (PK)   │ name     │ attributes             │
├──────────────┼──────────┼────────────────────────┤
│ john@s.com   │ John Doe │ { grade: 10, ... }     │
│ jane@s.com   │ Jane     │ { grade: 11, ... }     │
└──────────────┴──────────┴────────────────────────┘

Primary Key: Partition Key (required)
Optional: Partition Key + Sort Key
```

**Example:**

```typescript
import AWS from 'aws-sdk';

const dynamodb = new AWS.DynamoDB.DocumentClient();

// Create student
async function createStudent(student: any) {
  await dynamodb.put({
    TableName: 'Students',
    Item: {
      email: student.email,  // Primary key
      name: student.name,
      grade: student.grade,
      createdAt: new Date().toISOString()
    }
  }).promise();
}

// Get student
async function getStudent(email: string) {
  const result = await dynamodb.get({
    TableName: 'Students',
    Key: { email }
  }).promise();

  return result.Item;
}

// Query students by grade (requires GSI - Global Secondary Index)
async function getStudentsByGrade(grade: number) {
  const result = await dynamodb.query({
    TableName: 'Students',
    IndexName: 'GradeIndex',  // GSI on grade
    KeyConditionExpression: 'grade = :grade',
    ExpressionAttributeValues: {
      ':grade': grade
    }
  }).promise();

  return result.Items;
}

// Update student
async function updateStudent(email: string, updates: any) {
  await dynamodb.update({
    TableName: 'Students',
    Key: { email },
    UpdateExpression: 'SET #name = :name, grade = :grade',
    ExpressionAttributeNames: {
      '#name': 'name'  // 'name' is reserved word
    },
    ExpressionAttributeValues: {
      ':name': updates.name,
      ':grade': updates.grade
    }
  }).promise();
}

// Delete student
async function deleteStudent(email: string) {
  await dynamodb.delete({
    TableName: 'Students',
    Key: { email }
  }).promise();
}
```

**RDS vs DynamoDB:**

```
Use RDS when:
✓ Complex queries (JOINs)
✓ Relationships between entities
✓ ACID transactions
✓ Existing SQL application

Use DynamoDB when:
✓ Simple key-value queries
✓ Need single-digit ms latency
✓ Serverless preferred
✓ Massive scale (millions of requests/sec)
✓ Global distribution needed
```

### 3. ElastiCache

**What:** Managed Redis or Memcached (in-memory cache)

**Use Cases:**
- Session storage
- Caching database queries
- Real-time leaderboards
- Pub/Sub messaging

**Example: Caching with Redis**

```typescript
import Redis from 'ioredis';

const redis = new Redis({
  host: 'myredis.cache.amazonaws.com',  // ElastiCache endpoint
  port: 6379
});

// Cache student data
async function getStudent(id: number) {
  const cacheKey = `student:${id}`;

  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }

  // Cache miss - query database
  const student = await db.Student.findByPk(id);

  // Store in cache (1 hour TTL)
  await redis.setex(cacheKey, 3600, JSON.stringify(student));

  return student;
}
```

---

## Networking & Content Delivery

### 1. VPC (Virtual Private Cloud)

**What:** Isolated network in AWS

```
┌─────────────────────────────────────────────────┐
│              VPC (10.0.0.0/16)                  │
│                                                 │
│  ┌──────────────────┐  ┌──────────────────┐    │
│  │ Public Subnet    │  │ Private Subnet   │    │
│  │ (10.0.1.0/24)    │  │ (10.0.2.0/24)    │    │
│  │                  │  │                  │    │
│  │ ┌──────────────┐ │  │ ┌──────────────┐ │    │
│  │ │ Web Server   │ │  │ │   Database   │ │    │
│  │ │ (EC2)        │─┼──┼→│   (RDS)      │ │    │
│  │ └──────────────┘ │  │ └──────────────┘ │    │
│  │                  │  │                  │    │
│  └──────────────────┘  └──────────────────┘    │
│           │                                     │
│    ┌──────┴──────┐                              │
│    │Internet     │                              │
│    │Gateway (IGW)│                              │
│    └──────┬──────┘                              │
└───────────┼─────────────────────────────────────┘
            │
         Internet
```

**Components:**

**Subnets:**
- Public: Has route to Internet Gateway (internet access)
- Private: No direct internet access (more secure)

**Route Table:**
```
Destination      Target
10.0.0.0/16     local (within VPC)
0.0.0.0/0       igw-12345 (Internet Gateway)
```

**Security Group (Stateful Firewall):**
```
Inbound Rules:
Port 80 (HTTP) from 0.0.0.0/0
Port 443 (HTTPS) from 0.0.0.0/0
Port 22 (SSH) from MY_IP only

Outbound Rules:
All traffic allowed (default)
```

**Network ACL (Stateless Firewall):**
- Subnet-level
- Numbered rules (processed in order)
- Both inbound and outbound rules required

### 2. CloudFront (CDN)

**What:** Content Delivery Network (cache content globally)

```
User (Japan) → CloudFront Edge (Tokyo) → S3 (us-east-1)
                       ↓
                  Cached (fast!)

User (Europe) → CloudFront Edge (London) → S3 (us-east-1)
                       ↓
                  Cached (fast!)
```

**Example: Serve Static Assets**

```typescript
// Upload to S3
await s3.putObject({
  Bucket: 'my-assets',
  Key: 'images/logo.png',
  Body: file,
  CacheControl: 'max-age=31536000'  // 1 year
}).promise();

// CloudFront URL (cached globally)
// https://d123456.cloudfront.net/images/logo.png

// Benefits:
// ✅ Low latency (< 50ms globally)
// ✅ Reduced S3 costs (fewer direct requests)
// ✅ DDoS protection
```

### 3. Route 53 (DNS)

**What:** Managed DNS service

**Routing Policies:**

**1. Simple:** One IP
```
example.com → 54.123.45.67
```

**2. Weighted:** Split traffic by percentage
```
example.com:
  70% → 54.123.45.67 (v1)
  30% → 54.123.45.68 (v2 - testing)
```

**3. Latency-based:** Route to nearest region
```
User in US → us-east-1 (50ms)
User in EU → eu-west-1 (20ms)
```

**4. Failover:** Active-passive
```
Primary: 54.123.45.67 (healthy)
Secondary: 54.123.45.68 (if primary fails)
```

**5. Geolocation:** Route by location
```
Users in US → us-east-1
Users in EU → eu-west-1
Users in Asia → ap-southeast-1
```

### 4. ELB (Elastic Load Balancer)

**Types:**

**1. Application Load Balancer (ALB) - Layer 7**
```
┌─────────────┐
│     ALB     │
└──────┬──────┘
       │
  ┌────┴────┬────────────┐
  │         │            │
/api/* → Server 1    /static/* → S3
/admin/* → Server 2
```

**Features:**
- HTTP/HTTPS routing
- Path-based routing (`/api/*` → Service A)
- Host-based routing (`api.example.com` → Service A)
- WebSocket support

**2. Network Load Balancer (NLB) - Layer 4**
```
Ultra-high performance (millions of requests/sec)
TCP/UDP traffic
Static IP support
Use: Gaming, IoT, extreme performance needs
```

**3. Classic Load Balancer (Deprecated)**

---

## Security & Identity

### 1. IAM (Identity & Access Management)

**What:** Control who can access what in AWS

**Concepts:**

**Users:** Individual people
```
User: john@company.com
Permissions: Can read S3, cannot delete
```

**Groups:** Collection of users
```
Group: Developers
Members: John, Jane, Bob
Permissions: EC2, RDS, S3 access
```

**Roles:** Temporary credentials for services
```
Role: EC2-S3-Access
Trusted Entity: EC2
Permissions: Read/Write S3

EC2 instance assumes role → Can access S3
```

**Policies:** JSON documents defining permissions
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteBucket",
      "Resource": "*"
    }
  ]
}
```

**Best Practices:**
- ✅ Use roles for EC2/Lambda (not access keys)
- ✅ Enable MFA for root account
- ✅ Principle of least privilege
- ✅ Rotate credentials regularly
- ❌ Never commit access keys to git
- ❌ Don't use root account for daily tasks

**Example: EC2 accesses S3**

```typescript
// ❌ Bad: Hardcoded credentials
const s3 = new AWS.S3({
  accessKeyId: 'AKIAIOSFODNN7EXAMPLE',  // NEVER do this!
  secretAccessKey: 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY'
});

// ✅ Good: Use IAM role
// 1. Create IAM role with S3 access
// 2. Attach role to EC2 instance
// 3. SDK automatically uses role credentials
const s3 = new AWS.S3();  // No credentials needed!
await s3.putObject({ Bucket: 'my-bucket', Key: 'file.txt', Body: 'data' }).promise();
```

### 2. Secrets Manager

**What:** Store and rotate secrets (passwords, API keys)

```typescript
import AWS from 'aws-sdk';

const secretsManager = new AWS.SecretsManager();

// Store secret
await secretsManager.createSecret({
  Name: 'prod/db/password',
  SecretString: JSON.stringify({
    username: 'admin',
    password: 'MySecurePassword123'
  })
}).promise();

// Retrieve secret
const secret = await secretsManager.getSecretValue({
  SecretId: 'prod/db/password'
}).promise();

const { username, password } = JSON.parse(secret.SecretString);

// Use in database connection
const sequelize = new Sequelize({
  host: process.env.DB_HOST,
  username,
  password,
  database: 'school_admin'
});
```

---

## Monitoring & Management

### 1. CloudWatch

**What:** Monitoring and logging

**Metrics:**
```typescript
import AWS from 'aws-sdk';

const cloudwatch = new AWS.CloudWatch();

// Publish custom metric
await cloudwatch.putMetricData({
  Namespace: 'SchoolAdmin',
  MetricData: [
    {
      MetricName: 'StudentEnrollments',
      Value: 50,
      Unit: 'Count',
      Timestamp: new Date()
    }
  ]
}).promise();

// Create alarm
await cloudwatch.putMetricAlarm({
  AlarmName: 'HighCPU',
  MetricName: 'CPUUtilization',
  Namespace: 'AWS/EC2',
  Statistic: 'Average',
  Period: 300,  // 5 minutes
  EvaluationPeriods: 2,
  Threshold: 80,  // 80%
  ComparisonOperator: 'GreaterThanThreshold',
  AlarmActions: ['arn:aws:sns:us-east-1:123456789012:alerts']
}).promise();
```

**Logs:**
```typescript
import winston from 'winston';
import CloudWatchTransport from 'winston-cloudwatch';

const logger = winston.createLogger({
  transports: [
    new CloudWatchTransport({
      logGroupName: '/aws/school-admin/api',
      logStreamName: `${new Date().toISOString().split('T')[0]}-${process.pid}`,
      awsRegion: 'us-east-1'
    })
  ]
});

logger.info('Student enrolled', {
  studentId: 123,
  classId: 456
});
```

### 2. X-Ray (Distributed Tracing)

**What:** Trace requests across services

```typescript
import AWSXRay from 'aws-xray-sdk';

// Wrap AWS SDK
const AWS = AWSXRay.captureAWS(require('aws-sdk'));

// Trace Express app
const app = express();
app.use(AWSXRay.express.openSegment('SchoolAdminAPI'));

app.get('/student/:id', async (req, res) => {
  // Automatically traced
  const student = await db.Student.findByPk(req.params.id);

  // Custom subsegment
  const segment = AWSXRay.getSegment();
  const subsegment = segment.addNewSubsegment('external-api');

  try {
    const grades = await fetch(`https://api.grades.com/student/${req.params.id}`);
    subsegment.close();
  } catch (error) {
    subsegment.addError(error);
    subsegment.close();
  }

  res.json(student);
});

app.use(AWSXRay.express.closeSegment());
```

**X-Ray shows:**
```
GET /student/123 (250ms total)
├── Database query (50ms)
├── External API call (150ms) ← Bottleneck!
└── Response serialization (50ms)
```

---

## Serverless Architecture

### Full Serverless Stack

```
Client
  ↓
CloudFront (CDN)
  ↓
S3 (Static hosting - React)
  ↓
API Gateway
  ↓
Lambda (Node.js API)
  ↓
DynamoDB / RDS

No servers to manage!
Pay only for execution time
Auto-scales to millions of requests
```

**Example: School Admin System (Serverless)**

```typescript
// lambda/students/create.ts
import AWS from 'aws-sdk';

const dynamodb = new AWS.DynamoDB.DocumentClient();

export const handler = async (event: any) => {
  const student = JSON.parse(event.body);

  // Validate
  if (!student.email || !student.name) {
    return {
      statusCode: 400,
      body: JSON.stringify({ error: 'Email and name required' })
    };
  }

  // Create student
  await dynamodb.put({
    TableName: process.env.STUDENTS_TABLE,
    Item: {
      email: student.email,
      name: student.name,
      grade: student.grade,
      createdAt: new Date().toISOString()
    }
  }).promise();

  return {
    statusCode: 201,
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*'
    },
    body: JSON.stringify({ message: 'Student created' })
  };
};
```

**API Gateway Configuration:**

```
POST /students → Lambda: students-create
GET /students → Lambda: students-list
GET /students/{id} → Lambda: students-get
PUT /students/{id} → Lambda: students-update
DELETE /students/{id} → Lambda: students-delete
```

**Infrastructure as Code (SAM template):**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  StudentsTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: students
      AttributeDefinitions:
        - AttributeName: email
          AttributeType: S
      KeySchema:
        - AttributeName: email
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST

  CreateStudentFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: students/create.handler
      Runtime: nodejs18.x
      Environment:
        Variables:
          STUDENTS_TABLE: !Ref StudentsTable
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref StudentsTable
      Events:
        CreateAPI:
          Type: Api
          Properties:
            Path: /students
            Method: POST
```

---

## Cost Optimization

### Cost Optimization Strategies

**1. Right-sizing:** Use appropriate instance sizes
```
❌ t3.2xlarge (8 vCPU, 32GB RAM) = $300/month
   Actual usage: 20% CPU, 8GB RAM

✅ t3.large (2 vCPU, 8GB RAM) = $75/month
   Savings: $225/month (75%)
```

**2. Reserved Instances / Savings Plans**
```
On-Demand: $0.0832/hour × 730 hours = $60.74/month
Reserved (1-year): $0.0499/hour × 730 hours = $36.43/month
Savings: $24.31/month (40%)
```

**3. Spot Instances** (for fault-tolerant workloads)
```
On-Demand: $0.0832/hour
Spot: $0.0250/hour
Savings: 70%
```

**4. Auto-scaling:** Scale down when not needed
```
Night/Weekend: 2 instances (minimum)
Business Hours: 10 instances (scale up)
Savings: ~50% vs running 10 instances 24/7
```

**5. S3 Lifecycle Policies**
```
Day 0-30: S3 Standard ($0.023/GB)
Day 30-90: S3 IA ($0.0125/GB)
Day 90+: Glacier ($0.004/GB)

1TB data for 1 year:
Without lifecycle: $276
With lifecycle: $180
Savings: $96 (35%)
```

**6. CloudFront (reduce S3 costs)**
```
Without CDN:
1M requests to S3 = $0.40 (requests) + $90 (transfer)

With CloudFront:
1M requests = $10 (CloudFront) + $0.04 (S3, 10% miss rate)
Savings: $80.36 (89%)
```

**7. Lambda vs EC2**
```
EC2 t3.small (24/7): $15/month
Lambda (100K requests/day, 512MB, 1s): $10/month

For variable workloads, Lambda often cheaper
```

### Cost Monitoring

```typescript
// AWS Cost Explorer API
import AWS from 'aws-sdk';

const ce = new AWS.CostExplorer();

// Get monthly costs
const costs = await ce.getCostAndUsage({
  TimePeriod: {
    Start: '2024-01-01',
    End: '2024-01-31'
  },
  Granularity: 'MONTHLY',
  Metrics: ['UnblendedCost'],
  GroupBy: [
    { Type: 'DIMENSION', Key: 'SERVICE' }
  ]
}).promise();

console.log(costs.ResultsByTime[0].Groups);
// EC2: $500
// RDS: $200
// S3: $50
// Lambda: $10
```

**Set Budget Alerts:**

```bash
# Create budget alert
aws budgets create-budget \
  --account-id 123456789012 \
  --budget '{
    "BudgetName": "Monthly AWS Budget",
    "BudgetLimit": {
      "Amount": "1000",
      "Unit": "USD"
    },
    "TimeUnit": "MONTHLY",
    "BudgetType": "COST"
  }' \
  --notifications-with-subscribers '[
    {
      "Notification": {
        "NotificationType": "ACTUAL",
        "ComparisonOperator": "GREATER_THAN",
        "Threshold": 80
      },
      "Subscribers": [{
        "SubscriptionType": "EMAIL",
        "Address": "admin@example.com"
      }]
    }
  ]'
```

---

## Common Interview Questions

### Q1: Explain the difference between EC2 and Lambda

**Answer:**

"**EC2 (Elastic Compute Cloud):**
- Virtual servers you manage
- You choose instance type, OS, etc.
- Always running (pay for uptime)
- Full control
- Use for: Long-running applications, specific requirements

**Lambda:**
- Serverless functions
- No server management
- Pay only for execution time
- Auto-scales automatically
- 15-minute max execution time
- Use for: Event-driven, sporadic workloads, APIs

**Example:**
For my school admin system, I would use:
- **EC2** if I need to run the application 24/7 with full control over the environment
- **Lambda** if I want serverless APIs that auto-scale and only pay for execution time

**Cost comparison (100K API requests/day):**
- EC2 t3.small (24/7): ~$15/month
- Lambda (1s execution): ~$10/month

Lambda is cheaper for variable workloads, EC2 better for consistent 24/7 usage."

### Q2: How would you deploy your school admin system to AWS?

**Answer:**

"I would use a scalable, highly available architecture:

**Architecture:**

```
Internet
  ↓
Route 53 (DNS: school-admin.com)
  ↓
CloudFront (CDN for static assets)
  ↓
Application Load Balancer (ALB)
  ↓
Auto Scaling Group (EC2 instances)
├── AZ-1a: 2 instances (minimum)
└── AZ-1b: 2 instances (minimum)
  ↓
RDS MySQL (Multi-AZ)
├── Primary (AZ-1a)
└── Standby (AZ-1b)
  ↓
ElastiCache Redis (caching)
  ↓
S3 (file uploads, CSV imports)
```

**Components:**

1. **Frontend (React):**
   - Build: `npm run build`
   - Upload to S3
   - Serve via CloudFront
   - Benefits: Low latency, cheap, scalable

2. **Backend (Node.js/Express):**
   - Containerize with Docker
   - Deploy to ECS with Fargate (or EC2 instances)
   - Auto-scaling: 2-10 instances based on CPU
   - Behind ALB for load balancing

3. **Database:**
   - RDS MySQL (Multi-AZ for high availability)
   - db.t3.medium instance
   - Automated backups (7 days retention)
   - Read replica for read-heavy queries

4. **Caching:**
   - ElastiCache Redis
   - Cache student/class data (5-minute TTL)
   - Session storage

5. **File Storage:**
   - S3 for CSV imports
   - Lifecycle policy: Delete after 30 days

**Security:**
- VPC with public/private subnets
- EC2 in private subnets (no direct internet access)
- RDS in private subnet
- Security groups (allow only necessary ports)
- IAM roles (no hardcoded credentials)
- Secrets Manager for database password

**Monitoring:**
- CloudWatch for metrics and logs
- Alarms for high CPU, errors
- X-Ray for distributed tracing

**Cost estimate (monthly):**
- EC2 (2× t3.medium, reserved): $60
- RDS (db.t3.medium, Multi-AZ): $100
- ElastiCache (cache.t3.micro): $12
- S3: $5
- CloudFront: $10
- Load Balancer: $20
- **Total: ~$207/month**"

### Q3: How do you ensure high availability in AWS?

**Answer:**

"High availability means minimizing downtime. I use several strategies:

**1. Multi-AZ Deployment:**
Deploy across multiple Availability Zones (data centers)
```
If AZ-1a fails → AZ-1b continues serving
```

**2. Auto Scaling:**
Automatically replace failed instances
```
If instance fails → Auto Scaling launches new instance
```

**3. Load Balancing:**
Distribute traffic, health checks
```
If instance unhealthy → ALB stops routing to it
```

**4. Database:**
RDS Multi-AZ (automatic failover)
```
Primary fails → Standby promoted (1-2 min)
```

**5. Immutable Infrastructure:**
Treat servers as disposable
```
Don't fix servers → Replace them
```

**6. Backups:**
Automated RDS backups, S3 versioning
```
Disaster → Restore from backup
```

**Example Architecture:**
```
┌───────────────────────────────┐
│      Load Balancer (ALB)      │ ← Single entry point
└───────────┬───────────────────┘
            │
    ┌───────┼───────┐
    │               │
┌───┴────┐      ┌───┴────┐
│ AZ-1a  │      │ AZ-1b  │ ← Multiple AZs
│        │      │        │
│ ┌────┐ │      │ ┌────┐ │
│ │EC2 │ │      │ │EC2 │ │ ← Auto Scaling
│ └────┘ │      │ └────┘ │
│        │      │        │
│ ┌────┐ │      │ ┌────┐ │
│ │RDS │←┼──────┼→│RDS │ │ ← Multi-AZ
│ │Pri │ │ Sync │ │Stby│ │
│ └────┘ │      │ └────┘ │
└────────┘      └────────┘
```

**Result: 99.99% availability (< 5 min downtime/month)**"

### Q4: What is the difference between S3 and EBS?

**Answer:**

```
S3 (Simple Storage Service):
- Object storage (files, images, videos)
- Accessed via HTTP/HTTPS
- Virtually unlimited storage
- Highly durable (11 nines)
- Independent of EC2
- Lower cost
- Use: Backups, media files, static websites

EBS (Elastic Block Store):
- Block storage (like hard drive)
- Attached to single EC2 instance
- Limited to 16 TB per volume
- Lower latency (local to EC2)
- Persistent (survives instance stop)
- Higher cost
- Use: Database storage, OS drives

Example:
- Store user uploads → S3
- MySQL database storage → EBS
- Application logs → S3 (after rotation)
- EC2 root volume → EBS
```

---

## Summary: AWS Essentials Checklist

```
☐ Understand cloud basics
  ☐ IaaS, PaaS, SaaS
  ☐ Public, Private, Hybrid cloud

☐ Know AWS global infrastructure
  ☐ Regions, Availability Zones
  ☐ Edge locations

☐ Master core services
  ☐ Compute: EC2, Lambda, ECS
  ☐ Storage: S3, EBS, EFS
  ☐ Database: RDS, DynamoDB
  ☐ Networking: VPC, CloudFront, ALB

☐ Security
  ☐ IAM (users, roles, policies)
  ☐ Security groups
  ☐ Secrets Manager

☐ Monitoring
  ☐ CloudWatch (metrics, logs)
  ☐ X-Ray (tracing)

☐ Cost optimization
  ☐ Right-sizing
  ☐ Reserved instances
  ☐ Auto-scaling
  ☐ S3 lifecycle policies

☐ High availability
  ☐ Multi-AZ deployment
  ☐ Auto Scaling
  ☐ Load balancing
```

**Next:** AWS Architecture Patterns & Advanced Services
