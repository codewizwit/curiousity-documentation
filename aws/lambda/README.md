# AWS Lambda

_Running code without managing servers._

## What's Lambda? (The Food Truck Metaphor)

Traditional servers are like owning a restaurant:
- You pay rent 24/7 (even when closed)
- You manage the building, equipment, utilities
- You're responsible for maintenance and scaling

**Lambda is like a food truck:**
- You only pay when you're serving customers
- Someone else owns the truck (AWS manages servers)
- Instantly scale to multiple trucks during lunch rush
- Park it (zero cost) when there's no demand

You write the code (the recipe), AWS handles everything else.

---

## Core Concepts

### Event-Driven Execution

Lambda functions run in response to **events**:

```
API Gateway request → Lambda
S3 file upload → Lambda
DynamoDB change → Lambda
CloudWatch schedule → Lambda (cron job)
SNS/SQS message → Lambda
```

**You don't call Lambda directly.** You configure triggers, and AWS invokes your function when events occur.

---

### Stateless & Ephemeral

Each Lambda invocation is **stateless** — no memory of previous executions.

```javascript
// ❌ This doesn't work across invocations
let counter = 0;

export const handler = async () => {
  counter++;  // Resets between invocations
  return counter;
};
```

**If you need state:** Use DynamoDB, S3, or Parameter Store.

---

### Cold Starts vs Warm Invocations

**Cold start:** First invocation (or after idle) takes longer because AWS provisions a new execution environment.

**Warm invocation:** Subsequent invocations reuse the existing environment (faster).

```
Cold Start: ~1-3 seconds
Warm: ~10-100ms
```

**Optimization tips:**
- Keep package size small
- Use provisioned concurrency for critical functions
- Initialize connections outside the handler (reused on warm starts)

---

## Lambda Function Anatomy

```javascript
// handler.js
export const handler = async (event, context) => {
  // event = trigger data (API request, S3 event, etc.)
  // context = runtime information (request ID, remaining time, etc.)

  console.log('Event:', JSON.stringify(event));

  // Your business logic here
  const result = processEvent(event);

  // Return response
  return {
    statusCode: 200,
    body: JSON.stringify({ message: 'Success', result }),
  };
};
```

### Key Parameters

- **event** — Data from the trigger (HTTP request body, S3 object key, etc.)
- **context** — Metadata about the invocation (request ID, memory limit, time remaining)

---

## Common Lambda Patterns

### API Backend (Lambda + API Gateway)

```
User → API Gateway → Lambda → DynamoDB
         (HTTP)       (logic)   (data)
```

**Use case:** Serverless REST API

```javascript
export const handler = async (event) => {
  const { httpMethod, path, body } = event;

  if (httpMethod === 'GET' && path === '/users') {
    return {
      statusCode: 200,
      body: JSON.stringify(await getUsers()),
    };
  }

  if (httpMethod === 'POST' && path === '/users') {
    return {
      statusCode: 201,
      body: JSON.stringify(await createUser(JSON.parse(body))),
    };
  }

  return { statusCode: 404, body: 'Not Found' };
};
```

---

### S3 Event Processor

```
S3 Upload → Lambda → Process File → Store Result
```

**Use case:** Image resize, file parsing, ETL

```javascript
import { S3Client, GetObjectCommand } from '@aws-sdk/client-s3';

export const handler = async (event) => {
  const s3 = new S3Client();

  for (const record of event.Records) {
    const bucket = record.s3.bucket.name;
    const key = record.s3.object.key;

    // Get file from S3
    const command = new GetObjectCommand({ Bucket: bucket, Key: key });
    const response = await s3.send(command);

    // Process file
    const fileContent = await response.Body.transformToString();
    await processFile(fileContent);
  }
};
```

---

### Scheduled Jobs (Lambda + EventBridge)

```
EventBridge (cron) → Lambda → Execute Task
```

**Use case:** Nightly backups, report generation, cleanup tasks

```javascript
// Runs daily at 2 AM UTC
export const handler = async () => {
  console.log('Running scheduled task');
  await performDailyBackup();
  return 'Backup completed';
};
```

**EventBridge schedule syntax:**
```
rate(5 minutes)           # Every 5 minutes
cron(0 2 * * ? *)         # Daily at 2 AM UTC
cron(0 12 ? * MON-FRI *)  # Weekdays at noon
```

---

### Async Processing (SQS + Lambda)

```
Service → SQS Queue → Lambda → Process Message
```

**Use case:** Background jobs, decoupled processing

```javascript
export const handler = async (event) => {
  for (const record of event.Records) {
    const message = JSON.parse(record.body);
    await processMessage(message);
  }
};
```

**Benefits:** Automatic retry, dead letter queue for failures, controlled concurrency.

---

## Lambda Configuration

### Memory & CPU

- **Memory:** 128 MB - 10 GB
- **CPU:** Scales proportionally with memory
- **Duration:** Max 15 minutes

**Pricing:** Pay for GB-seconds (memory × execution time).

```
128 MB for 100ms = cheap
1 GB for 10 seconds = moderate
10 GB for 5 minutes = expensive
```

**Tip:** More memory = faster execution = potentially cheaper overall.

---

### Environment Variables

Store configuration without hardcoding:

```javascript
export const handler = async () => {
  const apiKey = process.env.API_KEY;
  const dbUrl = process.env.DATABASE_URL;

  // Use configuration
};
```

**Set in Lambda console or IaC:**
```yaml
# CloudFormation
Environment:
  Variables:
    API_KEY: !Ref ApiKeyParameter
    DATABASE_URL: !Ref DatabaseUrl
```

---

### Layers

**Layers** = shared code/dependencies across functions.

```
Lambda Function 1 ─┐
Lambda Function 2 ─┼─→ Shared Layer (node_modules, utils)
Lambda Function 3 ─┘
```

**Use case:** Common dependencies, utility functions, SDKs.

**Benefits:** Smaller deployment packages, DRY code, faster deployments.

---

### VPC Configuration

By default, Lambda runs in AWS's network (can access internet, public AWS services).

**To access private resources** (RDS in VPC, private APIs), attach Lambda to your VPC:

```yaml
VpcConfig:
  SubnetIds:
    - subnet-abc123
    - subnet-def456
  SecurityGroupIds:
    - sg-xyz789
```

**Trade-off:** VPC Lambdas have longer cold starts (~10 seconds).

---

## Lambda Best Practices

### 1. Initialize Outside the Handler

Connections, SDK clients, and heavy initialization should happen **outside** the handler (reused on warm invocations).

```javascript
// ✅ Initialized once (reused)
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
const dynamodb = new DynamoDBClient();

export const handler = async (event) => {
  // Use dynamodb client
};
```

```javascript
// ❌ Initialized every time (slow)
export const handler = async (event) => {
  const dynamodb = new DynamoDBClient();
  // Use dynamodb client
};
```

---

### 2. Keep Package Size Small

Smaller packages = faster cold starts.

- Use tree-shaking (webpack, esbuild)
- Don't include dev dependencies
- Use layers for heavy dependencies

---

### 3. Handle Errors Gracefully

Lambda retries failed invocations automatically. Return errors intentionally:

```javascript
export const handler = async (event) => {
  try {
    await processEvent(event);
    return { statusCode: 200, body: 'Success' };
  } catch (error) {
    console.error('Error:', error);

    // Temporary error? Let Lambda retry
    if (error.code === 'ServiceUnavailable') {
      throw error;
    }

    // Permanent error? Return failure (don't retry)
    return { statusCode: 400, body: 'Invalid request' };
  }
};
```

---

### 4. Use Dead Letter Queues

Failed invocations (after retries) go to a Dead Letter Queue (SQS or SNS) for investigation.

```yaml
DeadLetterConfig:
  TargetArn: !GetAtt FailedInvocationsQueue.Arn
```

---

### 5. Monitor with CloudWatch

```javascript
// Custom metrics
console.log(JSON.stringify({
  metric: 'ProcessingTime',
  value: 1234,
  unit: 'Milliseconds'
}));
```

**CloudWatch Metrics:**
- Invocations
- Errors
- Duration
- Throttles
- Concurrent executions

---

## Lambda Limits & Quotas

| Limit | Value |
|-------|-------|
| Max execution time | 15 minutes |
| Max memory | 10 GB |
| Deployment package size | 50 MB (zipped), 250 MB (unzipped) |
| /tmp storage | 512 MB - 10 GB |
| Concurrent executions | 1,000 (default, can request increase) |
| Payload size (synchronous) | 6 MB |
| Payload size (asynchronous) | 256 KB |

---

## Lambda Pricing

**Free Tier:**
- 1 million requests/month
- 400,000 GB-seconds/month

**After free tier:**
- $0.20 per 1 million requests
- $0.0000166667 per GB-second

**Example:**
- 128 MB function, 100ms execution
- 1 million invocations
- Cost: $0.20 (requests) + $0.21 (compute) = **$0.41/month**

---

## Common Lambda Gotchas

### Cold Start Latency

**Problem:** First invocation slow.
**Solutions:** Provisioned concurrency, keep package small, warm with scheduled pings.

---

### Timeout Too Short

**Problem:** Function times out before completing.
**Solution:** Increase timeout (max 15 minutes), or split work into smaller chunks.

---

### Concurrent Execution Limit

**Problem:** Throttling errors during traffic spikes.
**Solution:** Request limit increase, use reserved concurrency, implement SQS buffer.

---

### VPC Cold Start Delay

**Problem:** VPC Lambdas take ~10 seconds to cold start.
**Solution:** Use Hyperplane ENIs (AWS improved this), provisioned concurrency, or avoid VPC if possible.

---

## Resources

- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [Lambda Pricing Calculator](https://aws.amazon.com/lambda/pricing/)
- [AWS SAM (Serverless Application Model)](https://aws.amazon.com/serverless/sam/)

---

_Code that runs on demand — no servers, just functions._
