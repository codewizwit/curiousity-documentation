# AWS CloudFormation

_Infrastructure as code — version control your cloud._

## What's CloudFormation? (The Blueprint Metaphor)

Building a house, you have two options:

1. **Manual construction** — Tell workers where to place each brick, wire, pipe (error-prone, hard to replicate)
2. **Blueprint-driven** — Give workers architectural plans, they build it consistently every time

**CloudFormation is the blueprint for AWS infrastructure.** You write a template (YAML/JSON) describing what you want, and CloudFormation creates it for you.

**Benefits:**
- **Version control** — Infrastructure changes tracked in Git
- **Repeatable** — Deploy identical environments (dev, staging, prod)
- **Rollback** — Undo changes automatically if something fails
- **Documentation** — The template IS your infrastructure documentation

---

## Core Concepts

### Stack

A **stack** is a collection of AWS resources managed as a single unit.

```
Stack: MyApp
├── Lambda Function
├── DynamoDB Table
├── API Gateway
└── IAM Role
```

**Operations:**
- **Create** — Provision all resources
- **Update** — Modify existing resources
- **Delete** — Tear down everything

---

### Template

A **template** defines your infrastructure in YAML or JSON.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: My application infrastructure

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-app-bucket

  MyFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: my-function
      Runtime: nodejs18.x
      Handler: index.handler
      Code:
        ZipFile: |
          exports.handler = async () => {
            return { statusCode: 200, body: 'Hello' };
          };
      Role: !GetAtt MyFunctionRole.Arn
```

---

### Resources

**Resources** are the AWS services you're creating (S3, Lambda, DynamoDB, etc.).

```yaml
Resources:
  MyTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: Users
      AttributeDefinitions:
        - AttributeName: userId
          AttributeType: S
      KeySchema:
        - AttributeName: userId
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST
```

**Each resource has:**
- **Type** — AWS service (e.g., `AWS::S3::Bucket`)
- **Properties** — Configuration (bucket name, runtime, etc.)
- **Logical ID** — Name you reference in the template (`MyTable`)

---

### Parameters

**Parameters** make templates reusable by accepting input values.

```yaml
Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - staging
      - prod
    Description: Deployment environment

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'my-app-${Environment}-bucket'
```

**Usage:**

```bash
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=Environment,ParameterValue=prod
```

---

### Outputs

**Outputs** export values from a stack (used by other stacks or for reference).

```yaml
Outputs:
  BucketName:
    Description: S3 bucket name
    Value: !Ref MyBucket
    Export:
      Name: !Sub '${AWS::StackName}-BucketName'

  ApiEndpoint:
    Description: API Gateway endpoint
    Value: !Sub 'https://${MyApi}.execute-api.${AWS::Region}.amazonaws.com/prod'
```

**Reference in another stack:**

```yaml
Resources:
  MyFunction:
    Type: AWS::Lambda::Function
    Properties:
      Environment:
        Variables:
          BUCKET: !ImportValue my-stack-BucketName
```

---

### Intrinsic Functions

CloudFormation provides built-in functions for dynamic values:

#### !Ref

Reference a parameter or resource logical ID:

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket

  MyFunction:
    Type: AWS::Lambda::Function
    Properties:
      Environment:
        Variables:
          BUCKET_NAME: !Ref MyBucket  # Gets bucket name
```

---

#### !GetAtt

Get an attribute from a resource:

```yaml
Resources:
  MyRole:
    Type: AWS::IAM::Role

  MyFunction:
    Type: AWS::Lambda::Function
    Properties:
      Role: !GetAtt MyRole.Arn  # Gets the role ARN
```

---

#### !Sub

String substitution with variables:

```yaml
!Sub 'arn:aws:s3:::my-bucket-${Environment}'
!Sub 'https://${MyApi}.execute-api.${AWS::Region}.amazonaws.com'
```

---

#### !Join

Join strings with a delimiter:

```yaml
!Join
  - '-'
  - - my-app
    - !Ref Environment
    - bucket
# Result: my-app-dev-bucket
```

---

#### !If

Conditional logic:

```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !If
        - IsProduction
        - my-prod-bucket
        - my-dev-bucket
```

---

## Common CloudFormation Patterns

### Lambda Function with API Gateway

```yaml
Resources:
  MyApi:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: MyApi

  MyResource:
    Type: AWS::ApiGateway::Resource
    Properties:
      RestApiId: !Ref MyApi
      ParentId: !GetAtt MyApi.RootResourceId
      PathPart: users

  MyMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      RestApiId: !Ref MyApi
      ResourceId: !Ref MyResource
      HttpMethod: GET
      AuthorizationType: NONE
      Integration:
        Type: AWS_PROXY
        IntegrationHttpMethod: POST
        Uri: !Sub 'arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${MyFunction.Arn}/invocations'

  MyFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: my-function
      Runtime: nodejs18.x
      Handler: index.handler
      Code:
        ZipFile: |
          exports.handler = async () => ({
            statusCode: 200,
            body: JSON.stringify({ message: 'Hello' })
          });
      Role: !GetAtt MyFunctionRole.Arn
```

---

### DynamoDB Table with Lambda

```yaml
Resources:
  UsersTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: Users
      AttributeDefinitions:
        - AttributeName: userId
          AttributeType: S
      KeySchema:
        - AttributeName: userId
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST

  MyFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: process-users
      Runtime: nodejs18.x
      Handler: index.handler
      Environment:
        Variables:
          TABLE_NAME: !Ref UsersTable
      Role: !GetAtt MyFunctionRole.Arn
      Code:
        ZipFile: |
          const { DynamoDBClient, GetItemCommand } = require('@aws-sdk/client-dynamodb');
          const dynamodb = new DynamoDBClient();

          exports.handler = async (event) => {
            const tableName = process.env.TABLE_NAME;
            // Query DynamoDB
          };

  MyFunctionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: DynamoDBAccess
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - dynamodb:GetItem
                  - dynamodb:PutItem
                  - dynamodb:Query
                Resource: !GetAtt UsersTable.Arn
```

---

### S3 Bucket with Event Trigger

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-upload-bucket
      NotificationConfiguration:
        LambdaConfigurations:
          - Event: s3:ObjectCreated:*
            Function: !GetAtt ProcessFileFunction.Arn

  ProcessFileFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: process-s3-file
      Runtime: python3.9
      Handler: index.handler
      Role: !GetAtt ProcessFileFunctionRole.Arn
      Code:
        ZipFile: |
          def handler(event, context):
              for record in event['Records']:
                  bucket = record['s3']['bucket']['name']
                  key = record['s3']['object']['key']
                  print(f'File uploaded: {bucket}/{key}')

  LambdaInvokePermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref ProcessFileFunction
      Action: lambda:InvokeFunction
      Principal: s3.amazonaws.com
      SourceArn: !GetAtt MyBucket.Arn
```

---

## CloudFormation Best Practices

### 1. Use Parameters for Reusability

```yaml
Parameters:
  Environment:
    Type: String
  InstanceType:
    Type: String
    Default: t3.micro

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      Tags:
        - Key: Environment
          Value: !Ref Environment
```

---

### 2. Use Nested Stacks for Large Templates

Break large templates into smaller, reusable stacks:

```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-templates/network.yaml

  DatabaseStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-templates/database.yaml
      Parameters:
        VpcId: !GetAtt NetworkStack.Outputs.VpcId
```

---

### 3. Enable Stack Termination Protection

Prevent accidental deletion of critical stacks:

```bash
aws cloudformation update-termination-protection \
  --stack-name my-production-stack \
  --enable-termination-protection
```

---

### 4. Use Change Sets for Updates

Preview changes before applying:

```bash
# Create change set
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-changes \
  --template-body file://template.yaml

# Review changes
aws cloudformation describe-change-set \
  --stack-name my-stack \
  --change-set-name my-changes

# Apply changes
aws cloudformation execute-change-set \
  --stack-name my-stack \
  --change-set-name my-changes
```

---

### 5. Tag Everything

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      Tags:
        - Key: Environment
          Value: !Ref Environment
        - Key: Project
          Value: MyApp
        - Key: ManagedBy
          Value: CloudFormation
```

---

## CloudFormation CLI Commands

```bash
# Create stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters file://parameters.json \
  --capabilities CAPABILITY_IAM

# Update stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Describe stack
aws cloudformation describe-stacks --stack-name my-stack

# List stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE

# Get stack events
aws cloudformation describe-stack-events --stack-name my-stack

# Validate template
aws cloudformation validate-template --template-body file://template.yaml
```

---

## Common CloudFormation Gotchas

### Stack Update Failures

**Problem:** Stack stuck in `UPDATE_ROLLBACK_FAILED`.
**Solution:** Fix the underlying issue (IAM permissions, resource conflicts), then continue rollback.

```bash
aws cloudformation continue-update-rollback --stack-name my-stack
```

---

### Resource Replacement

**Problem:** Some property updates require resource replacement (data loss).
**Solution:** Check CloudFormation docs for "Update requires: Replacement" before updating.

---

### Circular Dependencies

**Problem:** Resource A depends on B, B depends on A.
**Solution:** Use `DependsOn` carefully, restructure dependencies, or split into separate stacks.

---

### IAM Permissions

**Problem:** `InsufficientCapabilitiesException` when creating IAM resources.
**Solution:** Add `--capabilities CAPABILITY_IAM` or `CAPABILITY_NAMED_IAM` flag.

```bash
aws cloudformation create-stack \
  --capabilities CAPABILITY_NAMED_IAM \
  ...
```

---

## CloudFormation vs Terraform

| Feature | CloudFormation | Terraform |
|---------|----------------|-----------|
| **Provider** | AWS only | Multi-cloud |
| **State management** | AWS-managed | Remote backend (S3, etc.) |
| **Language** | YAML/JSON | HCL |
| **Drift detection** | Built-in | `terraform plan` |
| **Cost** | Free | Free (Terraform Cloud costs) |
| **Community modules** | AWS Quick Starts | Terraform Registry |

**Use CloudFormation if:** AWS-only, prefer AWS-native tools
**Use Terraform if:** Multi-cloud, prefer HCL syntax, need advanced features

---

## Resources

- [CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [CloudFormation Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)
- [CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
- [AWS CDK](https://aws.amazon.com/cdk/) — Define infrastructure using TypeScript/Python (generates CloudFormation)

---

_Infrastructure as code — because clicking in the console doesn't scale._
