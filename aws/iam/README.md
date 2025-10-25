# AWS IAM (Identity and Access Management)

_Who can do what in AWS._

## What's IAM? (The Building Security Metaphor)

Imagine a secure office building:

- **Users** = Employees with ID badges
- **Groups** = Departments (Engineering, HR, Finance)
- **Roles** = Temporary visitor passes
- **Policies** = Rules for what each badge/pass can access

**IAM is AWS's security system.** It controls:
- **Authentication** — Who are you? (Identity)
- **Authorization** — What can you do? (Permissions)

---

## Core Concepts

### Users

**Users** = Long-term credentials for people or services.

```
User: alice@company.com
  └── Credentials:
      ├── Password (console access)
      └── Access Keys (programmatic access)
```

**When to use:**
- Individual people needing AWS console access
- CI/CD systems that need programmatic access
- Long-lived service accounts (though roles are preferred)

---

### Groups

**Groups** = Collections of users with shared permissions.

```
Group: Developers
  ├── User: alice
  ├── User: bob
  └── Permissions: EC2, Lambda, CloudWatch (read/write)

Group: Admins
  ├── User: charlie
  └── Permissions: Full access (*)
```

**Best practice:** Attach policies to groups, not individual users.

---

### Roles

**Roles** = Temporary credentials for services or federated users.

```
Role: LambdaExecutionRole
  └── Trust Policy: Lambda service can assume this role
  └── Permissions: DynamoDB read/write, CloudWatch logs
```

**When to use:**
- AWS services (Lambda, EC2, ECS) need to access other AWS resources
- Cross-account access (Service in Account A → Resource in Account B)
- Federated users (SSO, SAML) assume roles temporarily

**Key difference from users:** Roles don't have long-term credentials. They're assumed temporarily with short-lived tokens.

---

### Policies

**Policies** = JSON documents defining permissions.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

**Policy types:**
- **Identity-based** — Attached to users, groups, roles
- **Resource-based** — Attached to resources (S3 buckets, Lambda functions)
- **Managed** — Reusable policies (AWS-managed or customer-managed)
- **Inline** — Embedded directly in a single user/group/role

---

## IAM Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadOnlyS3Access",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

### Key Components:

- **Effect** — `Allow` or `Deny`
- **Action** — What operations (e.g., `s3:GetObject`, `lambda:InvokeFunction`)
- **Resource** — Which AWS resources (ARNs)
- **Condition** (optional) — When the policy applies (IP restrictions, time-based, MFA, etc.)

---

## Common IAM Patterns

### Lambda Execution Role

Lambda functions need a role to access AWS services:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:GetItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/Users"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
```

**Trust policy** (who can assume the role):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

### Cross-Account Access

Allow Service in Account A to access Resource in Account B:

**Account B (resource account):**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:role/AccountARole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Account A (assuming role):**

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::222222222222:role/CrossAccountRole \
  --role-session-name my-session
```

---

### Service Control Policies (SCPs)

**SCPs** = Guardrails for AWS Organizations. Even if a user has full admin, SCPs can block actions.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "ec2:TerminateInstances",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Environment": "production"
        }
      }
    }
  ]
}
```

**Use case:** Prevent anyone (even admins) from terminating production EC2 instances.

---

## IAM Best Practices

### 1. Principle of Least Privilege

Grant only the permissions needed, nothing more.

```json
// ❌ Too permissive
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}

// ✅ Least privilege
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/public/*"
}
```

---

### 2. Use Roles, Not Access Keys

**For AWS services:** Always use roles.
**For applications on EC2/ECS/Lambda:** Use roles, not hardcoded keys.

```javascript
// ❌ Hardcoded credentials (insecure)
const s3 = new S3Client({
  credentials: {
    accessKeyId: 'AKIAIOSFODNN7EXAMPLE',
    secretAccessKey: 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',
  },
});

// ✅ Use IAM role (credentials provided automatically)
const s3 = new S3Client(); // Pulls from role
```

---

### 3. Enable MFA for Privileged Users

Require multi-factor authentication for admin users:

```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "Bool": {
      "aws:MultiFactorAuthPresent": "true"
    }
  }
}
```

---

### 4. Rotate Credentials Regularly

- **Access keys:** Rotate every 90 days
- **Passwords:** Enforce rotation policy
- **Roles:** Use short-lived tokens (auto-rotated)

---

### 5. Use IAM Policy Simulator

Test policies before deploying:

```bash
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/alice \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/file.txt
```

---

## IAM vs STS (Security Token Service)

**IAM** = Long-term credentials (users, access keys)
**STS** = Temporary credentials (assumed roles, federated users)

```
User → AssumeRole → STS → Temporary Credentials (valid for 1 hour)
```

**Benefits of STS:**
- Credentials expire automatically
- No need to rotate manually
- Reduced risk if leaked

---

## Common IAM Gotchas

### Explicit Deny Overrides Allow

```json
// Policy 1: Allow S3 access
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}

// Policy 2: Deny S3 DeleteObject
{
  "Effect": "Deny",
  "Action": "s3:DeleteObject",
  "Resource": "*"
}

// Result: Can do everything EXCEPT delete (deny wins)
```

---

### Trust Policies vs Permission Policies

**Trust policy** = Who can assume the role
**Permission policy** = What the role can do

```
Lambda → Trust Policy (can assume role)
           ↓
        IAM Role
           ↓
     Permission Policy (can write to DynamoDB)
```

**Common mistake:** Forgetting to set trust policy, so no one can assume the role.

---

### Resource ARN Format Matters

```json
// ❌ Wrong: Missing /* (only allows bucket operations, not object operations)
"Resource": "arn:aws:s3:::my-bucket"

// ✅ Correct: Allows object operations
"Resource": [
  "arn:aws:s3:::my-bucket",     // Bucket operations (ListBucket)
  "arn:aws:s3:::my-bucket/*"    // Object operations (GetObject, PutObject)
]
```

---

### Policy Evaluation Order

1. **Explicit Deny** — Always wins
2. **Organizations SCP** — Can block even if allowed by IAM
3. **Resource Policy** — Evaluated alongside identity policy
4. **IAM Permission Policy** — What the user/role is allowed to do
5. **Session Policy** — Temporary restrictions (assumed roles)

**Default:** Implicit deny (if not explicitly allowed, it's denied).

---

## IAM CLI Commands

```bash
# List users
aws iam list-users

# Create user
aws iam create-user --user-name alice

# Attach policy to user
aws iam attach-user-policy \
  --user-name alice \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Create role
aws iam create-role \
  --role-name LambdaExecutionRole \
  --assume-role-policy-document file://trust-policy.json

# List roles
aws iam list-roles

# Get role policy
aws iam get-role-policy \
  --role-name LambdaExecutionRole \
  --policy-name MyPolicy

# Assume role (get temporary credentials)
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/MyRole \
  --role-session-name my-session
```

---

## Resources

- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
- [IAM Policy Simulator](https://policysim.aws.amazon.com/)

---

_Securing AWS, one policy at a time._
