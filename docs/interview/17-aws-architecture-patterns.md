# AWS Architecture Patterns & Best Practices

## Table of Contents
1. [Well-Architected Framework](#well-architected-framework)
2. [Common Architecture Patterns](#common-architecture-patterns)
3. [Serverless Architectures](#serverless-architectures)
4. [Microservices on AWS](#microservices-on-aws)
5. [CI/CD on AWS](#cicd-on-aws)
6. [Disaster Recovery Strategies](#disaster-recovery-strategies)
7. [Migration Strategies](#migration-strategies)
8. [Real-World Architecture Examples](#real-world-architecture-examples)
9. [Interview Questions](#interview-questions)

---

## Well-Architected Framework

### The 6 Pillars

AWS Well-Architected Framework provides best practices across 6 pillars:

#### 1. Operational Excellence

**Principle:** Run and monitor systems to deliver business value

**Best Practices:**
- ✅ Infrastructure as Code (CloudFormation, Terraform)
- ✅ Automate deployments (CodePipeline)
- ✅ Make frequent, small, reversible changes
- ✅ Anticipate failure
- ✅ Learn from operational failures

**Example:**

```yaml
# CloudFormation template (Infrastructure as Code)
AWSTemplateFormatVersion: '2010-09-09'
Description: School Admin System Infrastructure

Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]
    Default: dev

Resources:
  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub ${Environment}-school-admin-vpc

  # Public Subnet
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true

  # Application Load Balancer
  ALB:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: !Sub ${Environment}-school-admin-alb
      Subnets:
        - !Ref PublicSubnet1
        - !Ref PublicSubnet2
      SecurityGroups:
        - !Ref ALBSecurityGroup

  # Auto Scaling Group
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      MinSize: 2
      MaxSize: 10
      DesiredCapacity: 2
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: !GetAtt LaunchTemplate.LatestVersionNumber
      TargetGroupARNs:
        - !Ref TargetGroup
      HealthCheckType: ELB
      HealthCheckGracePeriod: 300
```

#### 2. Security

**Principle:** Protect information, systems, and assets

**Best Practices:**
- ✅ Implement strong identity foundation (IAM)
- ✅ Enable traceability (CloudTrail)
- ✅ Apply security at all layers
- ✅ Automate security best practices
- ✅ Protect data in transit and at rest
- ✅ Principle of least privilege

**Example: Secure S3 Bucket**

```typescript
import AWS from 'aws-sdk';

const s3 = new AWS.S3();

// Create bucket with encryption
await s3.createBucket({
  Bucket: 'school-admin-uploads',
  ACL: 'private'  // Not public
}).promise();

// Enable versioning (protect against accidental deletion)
await s3.putBucketVersioning({
  Bucket: 'school-admin-uploads',
  VersioningConfiguration: {
    Status: 'Enabled'
  }
}).promise();

// Enable server-side encryption
await s3.putBucketEncryption({
  Bucket: 'school-admin-uploads',
  ServerSideEncryptionConfiguration: {
    Rules: [{
      ApplyServerSideEncryptionByDefault: {
        SSEAlgorithm: 'AES256'
      }
    }]
  }
}).promise();

// Block public access
await s3.putPublicAccessBlock({
  Bucket: 'school-admin-uploads',
  PublicAccessBlockConfiguration: {
    BlockPublicAcls: true,
    BlockPublicPolicy: true,
    IgnorePublicAcls: true,
    RestrictPublicBuckets: true
  }
}).promise();

// Bucket policy (allow only specific IAM role)
await s3.putBucketPolicy({
  Bucket: 'school-admin-uploads',
  Policy: JSON.stringify({
    Version: '2012-10-17',
    Statement: [{
      Sid: 'AllowEC2Role',
      Effect: 'Allow',
      Principal: {
        AWS: 'arn:aws:iam::123456789012:role/EC2-S3-Access'
      },
      Action: ['s3:GetObject', 's3:PutObject'],
      Resource: 'arn:aws:s3:::school-admin-uploads/*'
    }]
  })
}).promise();
```

**Security Layers:**

```
┌─────────────────────────────────────────────┐
│ Layer 7: Application (WAF, API Gateway)    │
├─────────────────────────────────────────────┤
│ Layer 6: Data (Encryption at rest/transit) │
├─────────────────────────────────────────────┤
│ Layer 5: Compute (IAM roles, patches)      │
├─────────────────────────────────────────────┤
│ Layer 4: Network (Security Groups, NACLs)  │
├─────────────────────────────────────────────┤
│ Layer 3: Infrastructure (VPC, subnets)     │
├─────────────────────────────────────────────┤
│ Layer 2: Detection (CloudTrail, GuardDuty) │
├─────────────────────────────────────────────┤
│ Layer 1: Identity (IAM, MFA, Cognito)      │
└─────────────────────────────────────────────┘
```

#### 3. Reliability

**Principle:** Ensure workload performs its intended function correctly and consistently

**Best Practices:**
- ✅ Automatically recover from failure
- ✅ Test recovery procedures
- ✅ Scale horizontally
- ✅ Stop guessing capacity
- ✅ Manage change through automation

**Example: Multi-AZ Deployment**

```typescript
// CloudFormation template
{
  "DBInstance": {
    "Type": "AWS::RDS::DBInstance",
    "Properties": {
      "DBInstanceIdentifier": "school-admin-db",
      "DBInstanceClass": "db.t3.medium",
      "Engine": "mysql",
      "EngineVersion": "8.0.33",
      "MultiAZ": true,  // ✅ High availability
      "BackupRetentionPeriod": 7,  // ✅ 7 days backup
      "PreferredBackupWindow": "03:00-04:00",
      "PreferredMaintenanceWindow": "sun:04:00-sun:05:00",
      "EnableCloudwatchLogsExports": ["error", "slowquery"],
      "DeletionProtection": true,  // ✅ Prevent accidental deletion
      "StorageEncrypted": true
    }
  },

  // Auto Scaling for EC2
  "AutoScalingGroup": {
    "Type": "AWS::AutoScaling::AutoScalingGroup",
    "Properties": {
      "MinSize": 2,  // ✅ Minimum for redundancy
      "MaxSize": 10,
      "DesiredCapacity": 2,
      "HealthCheckType": "ELB",
      "HealthCheckGracePeriod": 300,
      "VPCZoneIdentifier": [
        {"Ref": "PrivateSubnet1"},  // ✅ Multiple AZs
        {"Ref": "PrivateSubnet2"}
      ]
    }
  }
}
```

#### 4. Performance Efficiency

**Principle:** Use computing resources efficiently to meet requirements

**Best Practices:**
- ✅ Democratize advanced technologies (use managed services)
- ✅ Go global in minutes (CloudFront, multi-region)
- ✅ Use serverless architectures (Lambda)
- ✅ Experiment more often
- ✅ Consider mechanical sympathy

**Example: Caching Strategy**

```
┌────────────────────────────────────────┐
│ Layer 1: Browser Cache (max-age)      │
├────────────────────────────────────────┤
│ Layer 2: CloudFront (CDN)              │
├────────────────────────────────────────┤
│ Layer 3: API Gateway Cache             │
├────────────────────────────────────────┤
│ Layer 4: Application Cache (Redis)     │
├────────────────────────────────────────┤
│ Layer 5: Database Query Cache          │
└────────────────────────────────────────┘
```

```typescript
// Multi-layer caching
import Redis from 'ioredis';

const redis = new Redis({
  host: process.env.ELASTICACHE_ENDPOINT,
  port: 6379
});

app.get('/api/students', async (req, res) => {
  const cacheKey = 'students:list';

  // Layer 1: Redis cache
  const cached = await redis.get(cacheKey);
  if (cached) {
    res.set('X-Cache', 'HIT');
    return res.json(JSON.parse(cached));
  }

  // Layer 2: Database
  const students = await db.Student.findAll();

  // Store in Redis (5 min TTL)
  await redis.setex(cacheKey, 300, JSON.stringify(students));

  // Set browser cache (1 min)
  res.set('Cache-Control', 'public, max-age=60');
  res.set('X-Cache', 'MISS');
  res.json(students);
});
```

#### 5. Cost Optimization

**Principle:** Run systems to deliver business value at the lowest price point

**Best Practices:**
- ✅ Implement cloud financial management
- ✅ Adopt consumption model (pay for what you use)
- ✅ Measure overall efficiency
- ✅ Stop spending on undifferentiated heavy lifting
- ✅ Analyze and attribute expenditure

**Example: Cost-Optimized Architecture**

```
Development Environment:
- EC2: t3.micro (spot instances)
- RDS: db.t3.micro (single AZ)
- ElastiCache: cache.t3.micro
- Cost: ~$50/month

Production Environment:
- EC2: t3.medium (reserved instances)
- RDS: db.t3.medium (multi-AZ)
- ElastiCache: cache.t3.small
- Auto Scaling: 2-10 instances
- Cost: ~$250/month

Savings:
✅ Reserved instances: 40% discount
✅ S3 lifecycle policies: 30% savings
✅ CloudFront: 80% reduction in S3 costs
✅ Auto Scaling: Only pay for needed capacity
```

#### 6. Sustainability

**Principle:** Minimize environmental impact

**Best Practices:**
- ✅ Understand your impact (carbon footprint)
- ✅ Maximize utilization (right-sizing)
- ✅ Use managed services (more efficient)
- ✅ Reduce downstream impact (caching, compression)

**Example:**

```typescript
// Use Lambda (serverless) instead of always-on EC2
// ✅ Lambda only uses resources during execution
// ✅ EC2 consumes power even when idle

// Reduce data transfer
app.use(compression());  // gzip compression

// Use efficient regions (renewable energy)
// us-west-2 (Oregon): 91% renewable energy
```

---

## Common Architecture Patterns

### 1. Three-Tier Architecture

**Classic web application pattern**

```
┌────────────────────────────────────────────┐
│              Internet                      │
└────────────────┬───────────────────────────┘
                 │
         ┌───────┴────────┐
         │  Presentation  │
         │     Tier       │
         │                │
         │ ┌────────────┐ │
         │ │CloudFront  │ │
         │ │(CDN)       │ │
         │ └─────┬──────┘ │
         │       │        │
         │ ┌─────┴──────┐ │
         │ │S3 (Static) │ │
         │ └────────────┘ │
         └───────┬────────┘
                 │
         ┌───────┴────────┐
         │  Application   │
         │     Tier       │
         │                │
         │ ┌────────────┐ │
         │ │    ALB     │ │
         │ └─────┬──────┘ │
         │       │        │
         │ ┌─────┴──────┐ │
         │ │Auto Scaling│ │
         │ │EC2 / ECS   │ │
         │ └────────────┘ │
         └───────┬────────┘
                 │
         ┌───────┴────────┐
         │     Data       │
         │     Tier       │
         │                │
         │ ┌────────────┐ │
         │ │ElastiCache │ │
         │ │(Redis)     │ │
         │ └────────────┘ │
         │       │        │
         │ ┌─────┴──────┐ │
         │ │    RDS     │ │
         │ │(Multi-AZ)  │ │
         │ └────────────┘ │
         └────────────────┘
```

**Implementation:**

```bash
# Frontend (React)
npm run build
aws s3 sync build/ s3://my-frontend-bucket
aws cloudfront create-invalidation --distribution-id E1234 --paths "/*"

# Backend (Node.js on EC2/ECS)
docker build -t school-admin-api .
docker tag school-admin-api:latest 123.dkr.ecr.us-east-1.amazonaws.com/school-admin:latest
docker push 123.dkr.ecr.us-east-1.amazonaws.com/school-admin:latest

# Deploy to ECS
aws ecs update-service --cluster prod --service api --force-new-deployment

# Database (RDS)
# Created via CloudFormation or console
```

### 2. Event-Driven Architecture

**Asynchronous, loosely coupled services**

```
┌──────────────┐
│  API Gateway │
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│   Lambda     │────►│     SQS      │
│  (Receiver)  │     │   (Queue)    │
└──────────────┘     └──────┬───────┘
                            │
                     ┌──────┼──────┬──────┐
                     │      │      │      │
                     ▼      ▼      ▼      ▼
              ┌────────┐ ┌────┐ ┌────┐ ┌────┐
              │Lambda 1│ │Lb2 │ │Lb3 │ │Lb4 │
              │(Email) │ │(DB)│ │(Log│ │(SNS│
              └────────┘ └────┘ └────┘ └────┘

Benefits:
✅ Decoupled (services don't know about each other)
✅ Scalable (each service scales independently)
✅ Resilient (failure in one doesn't affect others)
```

**Example: Student Enrollment Event**

```typescript
// 1. API receives enrollment request
import AWS from 'aws-sdk';

const sqs = new AWS.SQS();

app.post('/api/enroll', async (req, res) => {
  const { studentId, classId } = req.body;

  // Publish event to SQS
  await sqs.sendMessage({
    QueueUrl: process.env.ENROLLMENT_QUEUE_URL,
    MessageBody: JSON.stringify({
      eventType: 'STUDENT_ENROLLED',
      studentId,
      classId,
      timestamp: new Date().toISOString()
    })
  }).promise();

  res.json({ message: 'Enrollment queued', status: 'processing' });
});

// 2. Lambda processes enrollment (triggered by SQS)
export const enrollmentProcessor = async (event: any) => {
  for (const record of event.Records) {
    const enrollment = JSON.parse(record.body);

    // Process enrollment
    await db.ClassStudent.create({
      studentId: enrollment.studentId,
      classId: enrollment.classId
    });

    // Publish to SNS (fan-out to multiple subscribers)
    await sns.publish({
      TopicArn: process.env.ENROLLMENT_TOPIC_ARN,
      Message: JSON.stringify(enrollment)
    }).promise();
  }
};

// 3. Multiple Lambda functions subscribe to SNS
// Lambda A: Send email notification
export const emailNotification = async (event: any) => {
  const enrollment = JSON.parse(event.Records[0].Sns.Message);
  await sendEmail(enrollment.studentId, 'You have been enrolled!');
};

// Lambda B: Update analytics
export const analyticsUpdate = async (event: any) => {
  const enrollment = JSON.parse(event.Records[0].Sns.Message);
  await analytics.track('enrollment', enrollment);
};

// Lambda C: Update Elasticsearch
export const searchIndexUpdate = async (event: any) => {
  const enrollment = JSON.parse(event.Records[0].Sns.Message);
  await elasticsearch.index('enrollments', enrollment);
};
```

### 3. Microservices Architecture

**Independent, deployable services**

```
┌──────────────────────────────────────────┐
│          API Gateway                     │
└──────────┬───────────────────────────────┘
           │
    ┌──────┼──────┬──────┬──────┐
    │      │      │      │      │
    ▼      ▼      ▼      ▼      ▼
┌────────┐┌────┐┌────┐┌────┐┌────┐
│Student ││Class│Teacher│Auth││Report│
│Service ││Svc ││Svc   ││Svc ││Svc  │
└───┬────┘└─┬──┘└─┬────┘└─┬──┘└─┬───┘
    │       │     │       │     │
    ▼       ▼     ▼       ▼     ▼
┌────────┐┌────┐┌────┐┌────┐┌────┐
│DynamoDB││RDS ││RDS ││Cognito│Redis│
└────────┘└────┘└────┘└────┘└────┘

Each service:
✅ Independent deployment
✅ Own database
✅ Own scaling
✅ Own tech stack (if needed)
```

**Example: Student Service**

```typescript
// student-service/index.ts
import express from 'express';
import AWS from 'aws-sdk';

const app = express();
const dynamodb = new AWS.DynamoDB.DocumentClient();

// Get student
app.get('/students/:id', async (req, res) => {
  const student = await dynamodb.get({
    TableName: 'Students',
    Key: { id: req.params.id }
  }).promise();

  res.json(student.Item);
});

// Create student
app.post('/students', async (req, res) => {
  await dynamodb.put({
    TableName: 'Students',
    Item: {
      id: uuid(),
      ...req.body,
      createdAt: new Date().toISOString()
    }
  }).promise();

  res.status(201).json({ message: 'Student created' });
});

// Service-to-service communication
import axios from 'axios';

app.get('/students/:id/classes', async (req, res) => {
  // Call Class Service (service mesh / API Gateway)
  const classes = await axios.get(
    `${process.env.CLASS_SERVICE_URL}/classes?studentId=${req.params.id}`
  );

  res.json(classes.data);
});

app.listen(3000);
```

**Service Discovery:**

```typescript
// Using AWS Cloud Map for service discovery
import AWS from 'aws-sdk';

const servicediscovery = new AWS.ServiceDiscovery();

// Register service
await servicediscovery.registerInstance({
  ServiceId: 'srv-student-service',
  InstanceId: process.env.INSTANCE_ID,
  Attributes: {
    AWS_INSTANCE_IPV4: process.env.INSTANCE_IP,
    AWS_INSTANCE_PORT: '3000'
  }
}).promise();

// Discover service
const instances = await servicediscovery.discoverInstances({
  NamespaceName: 'school-admin',
  ServiceName: 'class-service'
}).promise();

const classServiceUrl = `http://${instances.Instances[0].Attributes.AWS_INSTANCE_IPV4}:${instances.Instances[0].Attributes.AWS_INSTANCE_PORT}`;
```

### 4. CQRS (Command Query Responsibility Segregation)

**Separate read and write paths**

```
Write Path (Commands):
Client → API Gateway → Lambda → DynamoDB (write)
                                    ↓
                            DynamoDB Streams
                                    ↓
                               Lambda (sync)
                                    ↓
                            Elasticsearch (read)

Read Path (Queries):
Client → API Gateway → Lambda → Elasticsearch (read)

Benefits:
✅ Optimized read queries (Elasticsearch)
✅ Optimized writes (DynamoDB)
✅ Independent scaling
```

**Example:**

```typescript
// Write model (Command)
app.post('/api/students', async (req, res) => {
  const student = {
    id: uuid(),
    ...req.body,
    version: 1,
    createdAt: new Date().toISOString()
  };

  // Write to DynamoDB (optimized for writes)
  await dynamodb.put({
    TableName: 'Students',
    Item: student
  }).promise();

  res.status(201).json({ id: student.id });
});

// DynamoDB Stream → Lambda (sync to read model)
export const syncToElasticsearch = async (event: any) => {
  for (const record of event.Records) {
    if (record.eventName === 'INSERT' || record.eventName === 'MODIFY') {
      const student = record.dynamodb.NewImage;

      // Sync to Elasticsearch (optimized for reads)
      await elasticsearch.index({
        index: 'students',
        id: student.id,
        body: AWS.DynamoDB.Converter.unmarshall(student)
      });
    }
  }
};

// Read model (Query) - Elasticsearch
app.get('/api/students/search', async (req, res) => {
  const { query } = req.query;

  // Complex search queries (not efficient in DynamoDB)
  const results = await elasticsearch.search({
    index: 'students',
    body: {
      query: {
        multi_match: {
          query,
          fields: ['name', 'email', 'classes'],
          fuzziness: 'AUTO'
        }
      }
    }
  });

  res.json(results.hits.hits.map(hit => hit._source));
});
```

---

## Serverless Architectures

### 1. Serverless Web Application

```
┌──────────────┐
│   Route 53   │ (DNS)
└──────┬───────┘
       │
┌──────┴───────┐
│  CloudFront  │ (CDN)
└──────┬───────┘
       │
   ┌───┴────┬────────────────┐
   │        │                │
   ▼        ▼                ▼
┌────┐  ┌────────────┐  ┌────────────┐
│ S3 │  │API Gateway │  │  Cognito   │
│    │  └─────┬──────┘  │ (Auth)     │
└────┘        │         └────────────┘
   ▲          │
   │          ▼
   │    ┌──────────┐
   │    │  Lambda  │
   │    └────┬─────┘
   │         │
   │    ┌────┴────┬────────┐
   │    │         │        │
   │    ▼         ▼        ▼
   │ ┌──────┐ ┌─────┐  ┌─────┐
   │ │DynamoDB│SQS │  │ SNS │
   │ └──────┘ └─────┘  └─────┘
   │
   └──── Thumbnail Lambda
```

**Full Example:**

```typescript
// 1. Frontend (React) → S3
// Build and deploy
npm run build
aws s3 sync build/ s3://school-admin-frontend --delete

// 2. API (Lambda) → API Gateway
// lambda/students/list.ts
export const handler = async (event: any) => {
  const students = await dynamodb.scan({
    TableName: process.env.STUDENTS_TABLE
  }).promise();

  return {
    statusCode: 200,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(students.Items)
  };
};

// 3. Authentication (Cognito)
import { CognitoIdentityProvider } from 'aws-sdk';

const cognito = new CognitoIdentityProvider();

// Sign up
app.post('/auth/signup', async (req, res) => {
  const { email, password } = req.body;

  await cognito.signUp({
    ClientId: process.env.COGNITO_CLIENT_ID,
    Username: email,
    Password: password,
    UserAttributes: [
      { Name: 'email', Value: email }
    ]
  }).promise();

  res.json({ message: 'Please check email for verification code' });
});

// Sign in
app.post('/auth/signin', async (req, res) => {
  const { email, password } = req.body;

  const result = await cognito.initiateAuth({
    ClientId: process.env.COGNITO_CLIENT_ID,
    AuthFlow: 'USER_PASSWORD_AUTH',
    AuthParameters: {
      USERNAME: email,
      PASSWORD: password
    }
  }).promise();

  res.json({
    token: result.AuthenticationResult.IdToken,
    refreshToken: result.AuthenticationResult.RefreshToken
  });
});

// 4. Authorizer (Lambda)
export const authorizer = async (event: any) => {
  const token = event.authorizationToken.replace('Bearer ', '');

  try {
    // Verify JWT token
    const decoded = await verifyToken(token);

    return {
      principalId: decoded.sub,
      policyDocument: {
        Version: '2012-10-17',
        Statement: [{
          Action: 'execute-api:Invoke',
          Effect: 'Allow',
          Resource: event.methodArn
        }]
      },
      context: {
        userId: decoded.sub,
        email: decoded.email
      }
    };
  } catch (error) {
    throw new Error('Unauthorized');
  }
};
```

**Cost Comparison (10K users):**

```
Traditional (EC2):
- EC2 t3.small (24/7): $15/month
- RDS db.t3.micro: $15/month
- Load Balancer: $20/month
Total: $50/month

Serverless:
- S3 hosting: $1/month
- Lambda (100K requests/day): $10/month
- DynamoDB (on-demand): $8/month
- API Gateway: $3.50/month
- CloudFront: $5/month
Total: $27.50/month

Savings: 45%
```

### 2. Serverless Data Processing Pipeline

```
S3 Upload → Lambda → Transform → Store

Example: CSV Import Pipeline
┌─────────────────────────────────────────────┐
│ 1. User uploads CSV to S3                  │
│    ↓                                        │
│ 2. S3 triggers Lambda                       │
│    ↓                                        │
│ 3. Lambda validates CSV                     │
│    ↓                                        │
│ 4. Lambda writes to SQS (batch)             │
│    ↓                                        │
│ 5. SQS triggers processing Lambda           │
│    ↓                                        │
│ 6. Lambda writes to DynamoDB                │
│    ↓                                        │
│ 7. DynamoDB Stream → Lambda → SNS           │
│    ↓                                        │
│ 8. SNS notifies user (email)                │
└─────────────────────────────────────────────┘
```

**Implementation:**

```typescript
// Step 1: S3 upload trigger
export const csvUploadHandler = async (event: any) => {
  const bucket = event.Records[0].s3.bucket.name;
  const key = event.Records[0].s3.object.key;

  // Download CSV
  const csv = await s3.getObject({ Bucket: bucket, Key: key }).promise();
  const rows = parseCSV(csv.Body.toString());

  // Validate
  const errors = validateCSV(rows);
  if (errors.length > 0) {
    await sns.publish({
      TopicArn: process.env.ERROR_TOPIC,
      Message: JSON.stringify({ errors })
    }).promise();
    return;
  }

  // Batch send to SQS (max 10 messages per batch)
  for (let i = 0; i < rows.length; i += 10) {
    const batch = rows.slice(i, i + 10);

    await sqs.sendMessageBatch({
      QueueUrl: process.env.IMPORT_QUEUE_URL,
      Entries: batch.map((row, index) => ({
        Id: `${i + index}`,
        MessageBody: JSON.stringify(row)
      }))
    }).promise();
  }

  console.log(`Queued ${rows.length} rows for processing`);
};

// Step 2: SQS processing (parallel Lambda invocations)
export const processRow = async (event: any) => {
  for (const record of event.Records) {
    const data = JSON.parse(record.body);

    // Create student
    await dynamodb.put({
      TableName: 'Students',
      Item: {
        id: uuid(),
        ...data,
        importedAt: new Date().toISOString()
      }
    }).promise();
  }
};

// Step 3: DynamoDB Stream → SNS notification
export const notifyCompletion = async (event: any) => {
  const processedCount = event.Records.length;

  await sns.publish({
    TopicArn: process.env.NOTIFICATION_TOPIC,
    Message: `CSV import completed. ${processedCount} students imported.`
  }).promise();
};
```

---

## Microservices on AWS

### Container Orchestration Options

**1. ECS (Elastic Container Service)**
```
AWS-native, simpler
Best for: AWS-only deployments
```

**2. EKS (Elastic Kubernetes Service)**
```
Standard Kubernetes
Best for: Multi-cloud, existing K8s expertise
```

**3. Fargate**
```
Serverless containers (works with ECS/EKS)
Best for: No infrastructure management
```

### ECS Microservices Architecture

```
┌──────────────────────────────────────────┐
│          Application Load Balancer       │
└──────────────┬───────────────────────────┘
               │
       ┌───────┼────────┬────────┐
       │       │        │        │
       ▼       ▼        ▼        ▼
    ┌────┐  ┌────┐  ┌────┐  ┌────┐
    │Svc │  │Svc │  │Svc │  │Svc │
    │ 1  │  │ 2  │  │ 3  │  │ 4  │
    └─┬──┘  └─┬──┘  └─┬──┘  └─┬──┘
      │       │       │       │
      └───────┼───────┼───────┘
              │       │
         ┌────┴───┐ ┌┴────┐
         │RDS     │ │Redis│
         └────────┘ └─────┘
```

**Task Definition (ECS):**

```json
{
  "family": "student-service",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "student-api",
      "image": "123.dkr.ecr.us-east-1.amazonaws.com/student-service:latest",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        { "name": "NODE_ENV", "value": "production" },
        { "name": "SERVICE_NAME", "value": "student-service" }
      ],
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123:secret:db-pass"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/student-service",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "api"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3
      }
    }
  ]
}
```

**Service Discovery (AWS Cloud Map):**

```typescript
// Service A (Student Service)
import AWS from 'aws-sdk';

const servicediscovery = new AWS.ServiceDiscovery();

// Discover Class Service
const instances = await servicediscovery.discoverInstances({
  NamespaceName: 'school-admin.local',
  ServiceName: 'class-service'
}).promise();

const classServiceEndpoint = `http://${instances.Instances[0].Attributes.AWS_INSTANCE_IPV4}:3000`;

// Call Class Service
const classes = await fetch(`${classServiceEndpoint}/classes?studentId=123`);
```

---

## CI/CD on AWS

### AWS CodePipeline Architecture

```
┌──────────────────────────────────────────────────┐
│                 CodePipeline                     │
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │  Source  │→ │  Build   │→ │  Deploy  │      │
│  │          │  │          │  │          │      │
│  │ GitHub/  │  │CodeBuild │  │CodeDeploy│      │
│  │CodeCommit│  │          │  │  or ECS  │      │
│  └──────────┘  └──────────┘  └──────────┘      │
└──────────────────────────────────────────────────┘
```

**buildspec.yml (CodeBuild):**

```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/school-admin
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}

  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG

  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"student-api","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json

artifacts:
  files: imagedefinitions.json
```

**CloudFormation Template (Full Pipeline):**

```yaml
Resources:
  CodePipeline:
    Type: AWS::CodePipeline::Pipeline
    Properties:
      RoleArn: !GetAtt CodePipelineRole.Arn
      Stages:
        - Name: Source
          Actions:
            - Name: SourceAction
              ActionTypeId:
                Category: Source
                Owner: ThirdParty
                Provider: GitHub
                Version: '1'
              Configuration:
                Owner: myorg
                Repo: school-admin-system
                Branch: main
                OAuthToken: !Ref GitHubToken
              OutputArtifacts:
                - Name: SourceOutput

        - Name: Build
          Actions:
            - Name: BuildAction
              ActionTypeId:
                Category: Build
                Owner: AWS
                Provider: CodeBuild
                Version: '1'
              Configuration:
                ProjectName: !Ref CodeBuildProject
              InputArtifacts:
                - Name: SourceOutput
              OutputArtifacts:
                - Name: BuildOutput

        - Name: Deploy
          Actions:
            - Name: DeployAction
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Provider: ECS
                Version: '1'
              Configuration:
                ClusterName: !Ref ECSCluster
                ServiceName: !Ref ECSService
                FileName: imagedefinitions.json
              InputArtifacts:
                - Name: BuildOutput
```

---

## Disaster Recovery Strategies

### RTO and RPO

```
RTO (Recovery Time Objective): How long to recover
RPO (Recovery Point Objective): How much data loss acceptable

Example:
RPO = 1 hour → Backups every hour (lose max 1 hour of data)
RTO = 4 hours → Must be back online within 4 hours
```

### DR Strategies (Cheap → Expensive)

#### 1. Backup and Restore (Cheapest, Slowest)

```
Normal:
Primary Region (us-east-1) → Active
Backup (S3) → Snapshots

Disaster:
1. Restore from S3 backups
2. Launch infrastructure (CloudFormation)
3. Restore database

RTO: Hours to days
RPO: Hours (last backup)
Cost: $
```

#### 2. Pilot Light

```
Normal:
Primary Region → Active
DR Region → Database running (minimal), app servers OFF

Disaster:
1. Scale up database
2. Launch app servers (Auto Scaling)
3. Update DNS (Route 53)

RTO: 10 minutes to 1 hour
RPO: Minutes (database replication)
Cost: $$
```

#### 3. Warm Standby

```
Normal:
Primary Region → Active (full capacity)
DR Region → Active (reduced capacity, e.g., 20%)

Disaster:
1. Scale up DR region to 100%
2. Update DNS (Route 53)

RTO: Minutes
RPO: Seconds (real-time replication)
Cost: $$$
```

#### 4. Multi-Region Active-Active (Most Expensive)

```
Normal:
Primary Region → Active (50% traffic)
Secondary Region → Active (50% traffic)

Disaster:
Already active! Automatic failover.

RTO: Seconds (automatic)
RPO: 0 (real-time replication)
Cost: $$$$
```

**Implementation: Multi-Region**

```typescript
// DynamoDB Global Tables (multi-region replication)
const params = {
  GlobalTableName: 'Students',
  ReplicationGroup: [
    { RegionName: 'us-east-1' },
    { RegionName: 'eu-west-1' },
    { RegionName: 'ap-southeast-1' }
  ]
};

await dynamodb.createGlobalTable(params).promise();

// Route 53 Health Checks + Failover
{
  "HealthCheck": {
    "Type": "HTTPS",
    "ResourcePath": "/health",
    "FullyQualifiedDomainName": "api.school-admin.com",
    "Port": 443,
    "RequestInterval": 30,
    "FailureThreshold": 3
  },

  "RecordSet": {
    "Name": "api.school-admin.com",
    "Type": "A",
    "SetIdentifier": "Primary",
    "Failover": "PRIMARY",
    "AliasTarget": {
      "DNSName": "us-east-1-alb.amazonaws.com",
      "HostedZoneId": "Z123456",
      "EvaluateTargetHealth": true
    }
  },

  "RecordSetSecondary": {
    "Name": "api.school-admin.com",
    "Type": "A",
    "SetIdentifier": "Secondary",
    "Failover": "SECONDARY",
    "AliasTarget": {
      "DNSName": "eu-west-1-alb.amazonaws.com",
      "HostedZoneId": "Z789012",
      "EvaluateTargetHealth": true
    }
  }
}
```

---

## Migration Strategies (6 R's)

### 1. Rehost ("Lift and Shift")

```
Current: On-premise VM
AWS: EC2 instance (same OS, same app)

Tools: AWS Application Migration Service
Time: Days
Risk: Low
Cost Savings: 30%
```

### 2. Replatform ("Lift, Tinker, and Shift")

```
Current: Self-managed MySQL
AWS: RDS MySQL (managed)

Benefits:
✅ Automated backups
✅ Automated patching
✅ Multi-AZ
✅ Read replicas

Time: Weeks
Risk: Low-Medium
Cost Savings: 40%
```

### 3. Repurchase ("Drop and Shop")

```
Current: On-premise email server
AWS: Amazon WorkMail (SaaS)

Time: Days
Risk: Low
Cost Savings: 50%
```

### 4. Refactor / Re-architect

```
Current: Monolithic app on EC2
AWS: Microservices on Lambda + API Gateway

Time: Months
Risk: High
Cost Savings: 60%
Performance: 10x improvement
```

### 5. Retire

```
Decommission unused applications
Cost Savings: 100% (for that app)
```

### 6. Retain

```
Keep on-premise (for now)
Reasons: Compliance, not ready
```

---

## Real-World Architecture Examples

### Example 1: E-commerce Platform (High Traffic)

```
┌────────────────────────────────────────────┐
│              Route 53 (DNS)                │
└──────────────┬─────────────────────────────┘
               │
┌──────────────┴─────────────────────────────┐
│           CloudFront (CDN)                 │
└──────────────┬─────────────────────────────┘
               │
      ┌────────┴────────┐
      │                 │
┌─────┴──────┐    ┌─────┴──────┐
│  S3        │    │   ALB      │
│ (Static)   │    │            │
└────────────┘    └─────┬──────┘
                        │
                 ┌──────┴──────┬──────┐
                 │             │      │
           ┌─────┴────┐  ┌─────┴────┐ ┌─────┐
           │ ECS      │  │ ECS      │ │ ECS │
           │ (Product)│  │ (Cart)   │ │(Order)│
           └─────┬────┘  └─────┬────┘ └──┬──┘
                 │             │          │
         ┌───────┴─────┬───────┴────┬─────┴────┐
         │             │            │          │
      ┌──┴──┐     ┌────┴────┐  ┌────┴───┐  ┌───┴───┐
      │Redis│     │DynamoDB │  │  RDS   │  │  SQS  │
      │     │     │(Products│  │(Orders)│  │(Jobs) │
      └─────┘     └─────────┘  └────────┘  └───┬───┘
                                                │
                                         ┌──────┴─────┐
                                         │  Lambda    │
                                         │ (Workers)  │
                                         └────────────┘

Traffic: 100K RPS
Users: 10M
Cost: ~$15,000/month
```

### Example 2: School Admin System (Your Project!)

```
Development:
- Frontend: S3 + CloudFront
- Backend: ECS Fargate (1 task)
- Database: RDS MySQL (db.t3.micro, single AZ)
- Cache: ElastiCache (cache.t3.micro)
Cost: ~$80/month

Production:
- Frontend: S3 + CloudFront
- Backend: ECS Fargate (Auto Scaling 2-10 tasks)
- Database: RDS MySQL (db.t3.medium, Multi-AZ)
- Cache: ElastiCache Redis (cache.t3.small)
- Storage: S3 (CSV uploads)
Cost: ~$300/month

Features:
✅ Auto-scaling (handle traffic spikes)
✅ High availability (Multi-AZ)
✅ Automated backups (7 days)
✅ CDN (fast global access)
✅ Monitoring (CloudWatch)
✅ CI/CD (CodePipeline)
```

---

## Interview Questions

### Q1: How would you migrate your school admin system to AWS?

**Answer in next document...**

(Continuing in next file due to length)
