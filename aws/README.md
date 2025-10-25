# AWS (Amazon Web Services)

_Cloud computing for applications that scale._

## What's AWS? (The Rental Property Metaphor)

Imagine you need office space for your business. You have three options:

1. **Buy a building** (Traditional servers) — You own it, maintain it, pay for it whether you use it or not
2. **Rent office space** (AWS) — Pay for what you use, someone else handles maintenance, scale up or down as needed
3. **Co-working space with services** (AWS Managed Services) — Not just space, but furniture, internet, coffee, cleaning — all included

**AWS is option 2 and 3 combined.** You rent compute resources (servers, databases, storage) on-demand, and AWS manages the infrastructure. You can choose how much responsibility you want:

- **IaaS** (Infrastructure as a Service) — Rent the raw space, you furnish it (EC2, S3)
- **PaaS** (Platform as a Service) — Rent a furnished office, you bring your work (Elastic Beanstalk, RDS)
- **FaaS** (Function as a Service) — Rent a desk by the hour, only when you need it (Lambda)

---

## Core Concepts

### Regions & Availability Zones

AWS operates in **regions** (geographic locations like `us-east-1` for N. Virginia) and **availability zones** (data centers within a region).

**Why it matters:**
- **Latency** — Deploy closer to your users
- **Compliance** — Keep data in specific countries/regions
- **High availability** — Spread across availability zones for redundancy

```
Region: us-east-1 (N. Virginia)
├── Availability Zone: us-east-1a
├── Availability Zone: us-east-1b
└── Availability Zone: us-east-1c
```

If one availability zone fails, your app can failover to another.

---

### Pay-As-You-Go Pricing

AWS charges for what you use:
- **Compute** — Pay per second/hour of server runtime
- **Storage** — Pay per GB stored
- **Data transfer** — Pay for data out (not in)
- **Requests** — Pay per API call (Lambda, API Gateway)

**The mindset shift:** Turn resources off when not in use. Scale up during traffic spikes, scale down during quiet periods.

---

## Key AWS Services

### Compute

**[Lambda](./lambda/README.md)** — Serverless functions. Run code without managing servers. Pay only when your code runs.

**EC2** (Elastic Compute Cloud) — Virtual servers in the cloud. Full control over the OS and configuration.

**ECS/EKS** — Container orchestration. Run Docker containers at scale (ECS = AWS native, EKS = Kubernetes).

---

### Storage

**S3** (Simple Storage Service) — Object storage for files. Think: unlimited, scalable file storage in the cloud.

**EBS** (Elastic Block Store) — Block storage for EC2 instances. Like a hard drive attached to your server.

**EFS** (Elastic File System) — Shared file storage that can be mounted by multiple EC2 instances.

---

### Database

**RDS** (Relational Database Service) — Managed databases (PostgreSQL, MySQL, etc.). AWS handles backups, patches, scaling.

**DynamoDB** — NoSQL database. Fast, scalable, fully managed key-value store.

**Aurora** — AWS's high-performance MySQL/PostgreSQL-compatible database.

---

### Networking

**VPC** (Virtual Private Cloud) — Your own isolated network in AWS. Control IP ranges, subnets, routing.

**API Gateway** — Managed API service. Create RESTful or WebSocket APIs that trigger Lambda or connect to backends.

**CloudFront** — CDN (Content Delivery Network). Cache content globally for faster delivery.

---

### Security & Identity

**[IAM](./iam/README.md)** (Identity and Access Management) — Control who can access what in AWS. Users, roles, policies, permissions.

**Secrets Manager** — Store and rotate secrets (API keys, database passwords) securely.

**KMS** (Key Management Service) — Encryption key management for securing data.

---

### Infrastructure as Code

**[CloudFormation](./cloudformation/README.md)** — Define AWS infrastructure in YAML/JSON templates. Version control your infrastructure.

**CDK** (Cloud Development Kit) — Define infrastructure using programming languages (TypeScript, Python). Generates CloudFormation.

---

## Common AWS Patterns

### Serverless Architecture

Combine Lambda + API Gateway + DynamoDB for fully managed, auto-scaling applications.

```
User Request
    ↓
API Gateway (HTTP endpoint)
    ↓
Lambda (business logic)
    ↓
DynamoDB (data storage)
```

**Benefits:** No servers to manage, automatic scaling, pay per request.

---

### Microservices with Containers

Run microservices in containers (Docker) on ECS or EKS.

```
Application Load Balancer
    ↓
┌─────────┬─────────┬─────────┐
│ Service │ Service │ Service │
│    A    │    B    │    C    │
└─────────┴─────────┴─────────┘
  (ECS Tasks / Kubernetes Pods)
```

**Benefits:** Independent deployment, language flexibility, isolated scaling.

---

### Event-Driven Architecture

Use SNS (Simple Notification Service) and SQS (Simple Queue Service) to decouple services.

```
Service A → SNS Topic → Multiple Subscribers
                         ├── Lambda Function
                         ├── SQS Queue → Service B
                         └── Email Notification
```

**Benefits:** Loose coupling, async processing, resilience to failures.

---

## AWS Best Practices

### Security

1. **Principle of Least Privilege** — Grant minimum permissions needed (via IAM)
2. **Enable MFA** — Multi-factor authentication for all users
3. **Encrypt at rest and in transit** — Use KMS for encryption keys
4. **Never hardcode credentials** — Use IAM roles, Secrets Manager, or Parameter Store
5. **Enable CloudTrail** — Audit all API calls for security monitoring

---

### Cost Optimization

1. **Right-size resources** — Don't over-provision EC2 instances
2. **Use Auto Scaling** — Scale based on demand, not peak capacity
3. **Leverage Reserved Instances** — Save up to 75% for predictable workloads
4. **Delete unused resources** — Old snapshots, unattached EBS volumes, idle instances
5. **Use Lambda for sporadic workloads** — Pay per invocation, not per hour

---

### High Availability

1. **Deploy across multiple Availability Zones** — Survive data center failures
2. **Use Load Balancers** — Distribute traffic, health check instances
3. **Implement Auto Scaling** — Replace failed instances automatically
4. **Enable Multi-AZ for databases** — RDS/Aurora failover to standby
5. **Use CloudFront** — Cache content globally, reduce load on origin

---

### Monitoring & Observability

1. **CloudWatch Logs** — Centralized log aggregation
2. **CloudWatch Metrics** — Track resource utilization, custom metrics
3. **CloudWatch Alarms** — Alert on thresholds (high CPU, error rates)
4. **X-Ray** — Distributed tracing for microservices
5. **Cost Explorer** — Track spending, identify cost drivers

---

## Common AWS Gotchas

### Lambda Cold Starts

**Problem:** First invocation after idle period is slow (cold start).
**Solution:** Keep functions warm with scheduled pings, use provisioned concurrency, optimize package size.

---

### IAM Policy Confusion

**Problem:** Permissions denied even though policy looks correct.
**Solution:** Check for explicit deny (overrides allow), ensure trust relationships are correct, test with IAM Policy Simulator.

---

### S3 Eventual Consistency

**Problem:** After uploading a file, it might not be immediately available.
**Solution:** S3 now has strong read-after-write consistency (as of Dec 2020), but cached responses can still cause stale reads.

---

### VPC Networking Complexity

**Problem:** Instances can't reach the internet or talk to each other.
**Solution:** Check route tables, security groups, NACLs, and internet/NAT gateway configuration.

---

### CloudFormation Stack Failures

**Problem:** Stack creation fails, resources are stuck in rollback.
**Solution:** Check CloudFormation events for error messages, validate template syntax, ensure IAM permissions are correct.

---

## AWS CLI Basics

```bash
# Configure AWS CLI
aws configure

# List S3 buckets
aws s3 ls

# Invoke Lambda function
aws lambda invoke --function-name myFunction output.json

# Describe EC2 instances
aws ec2 describe-instances

# Get CloudFormation stack status
aws cloudformation describe-stacks --stack-name myStack

# View CloudWatch logs
aws logs tail /aws/lambda/myFunction --follow
```

---

## Resources

- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS CLI Reference](https://docs.aws.amazon.com/cli/)

---

## Deep Dives

- **[Lambda: Serverless Functions](./lambda/README.md)** — Running code without servers
- **[IAM: Identity & Access Management](./iam/README.md)** — Security, roles, and policies
- **[CloudFormation: Infrastructure as Code](./cloudformation/README.md)** — Version-controlled infrastructure

---

_Renting the cloud, one service at a time._ 🌭

← [Back to Home](../README.md) | [Lambda](./lambda/README.md) | [IAM](./iam/README.md) | [CloudFormation](./cloudformation/README.md)
