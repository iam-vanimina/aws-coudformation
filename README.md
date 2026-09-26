# aws-coudformation
# ☁️ AWS CloudFormation — Step-by-Step Hands-On Learning

![AWS](https://img.shields.io/badge/AWS-CloudFormation-orange?logo=amazonaws)
![IaC](https://img.shields.io/badge/IaC-CloudFormation-blue)
![YAML](https://img.shields.io/badge/Template-YAML-red)
![License](https://img.shields.io/badge/License-MIT-green)

A **step-by-step hands-on AWS CloudFormation learning repository** covering Infrastructure as Code (IaC), from your first CloudFormation stack to production-style AWS infrastructure.

The goal is to learn CloudFormation by **building real infrastructure**, not just reading theory.

---

## 📚 What You Will Learn

```text
CloudFormation
│
├── 01. Introduction
├── 02. AWS CLI Setup
├── 03. First CloudFormation Stack
├── 04. Parameters
├── 05. Outputs
├── 06. Resources
├── 07. Mappings
├── 08. Conditions
├── 09. Intrinsic Functions
├── 10. Metadata
├── 11. References
├── 12. Cross-Stack References
├── 13. Nested Stacks
├── 14. IAM
├── 15. VPC
├── 16. EC2
├── 17. Security Groups
├── 18. Load Balancer
├── 19. Auto Scaling
├── 20. RDS
├── 21. S3
├── 22. CloudWatch
├── 23. SNS
├── 24. Secrets Manager
├── 25. CloudFormation Modules
├── 26. StackSets
├── 27. Change Sets
├── 28. Drift Detection
├── 29. CloudFormation Hooks
├── 30. Production Architecture
└── 31. CI/CD with GitHub Actions
```

---

# 🏗️ Architecture

The labs gradually build infrastructure like this:

```text
                         AWS ACCOUNT
                              │
                     CloudFormation
                              │
             ┌────────────────┼────────────────┐
             │                │                │
            VPC              IAM              S3
             │
      ┌──────┴──────┐
      │             │
 Public Subnet   Private Subnet
      │             │
      │             ├── EC2
      │             ├── RDS
      │             └── Application
      │
 Application Load Balancer
      │
      ▼
   EC2 / ASG
      │
      ▼
    RDS
```

---

# 1. Prerequisites

Before starting, install:

* AWS Account
* AWS CLI
* Git
* VS Code
* Basic Linux commands
* Basic AWS knowledge
* YAML basics

Verify AWS CLI:

```bash
aws --version
```

Configure credentials:

```bash
aws configure
```

Test:

```bash
aws sts get-caller-identity
```

Expected:

```json
{
    "UserId": "XXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-user"
}
```

---

# 2. Create the Repository

Create a Git repository:

```bash
mkdir aws-cloudformation-learning
cd aws-cloudformation-learning

git init
```

Recommended structure:

```text
aws-cloudformation-learning/
│
├── README.md
│
├── 01-basics/
│   ├── hello-world.yaml
│   ├── s3-bucket.yaml
│   └── ec2.yaml
│
├── 02-parameters/
│   ├── parameters.yaml
│   └── parameter-validation.yaml
│
├── 03-functions/
│   ├── ref.yaml
│   ├── getatt.yaml
│   ├── join.yaml
│   ├── sub.yaml
│   └── select.yaml
│
├── 04-conditions/
│   └── conditions.yaml
│
├── 05-mappings/
│   └── mappings.yaml
│
├── 06-iam/
│   ├── iam-role.yaml
│   └── iam-policy.yaml
│
├── 07-vpc/
│   ├── vpc.yaml
│   ├── subnets.yaml
│   ├── route-tables.yaml
│   └── vpc-complete.yaml
│
├── 08-ec2/
│   ├── ec2.yaml
│   └── ec2-userdata.yaml
│
├── 09-load-balancer/
│   ├── alb.yaml
│   └── alb-target-group.yaml
│
├── 10-auto-scaling/
│   └── autoscaling.yaml
│
├── 11-rds/
│   └── rds.yaml
│
├── 12-s3/
│   └── s3.yaml
│
├── 13-cloudwatch/
│   └── monitoring.yaml
│
├── 14-nested-stacks/
│   ├── network.yaml
│   └── application.yaml
│
├── 15-cross-stack/
│   ├── network.yaml
│   └── application.yaml
│
├── 16-production/
│   ├── network/
│   ├── security/
│   ├── compute/
│   ├── database/
│   └── monitoring/
│
├── 17-cicd/
│   └── github-actions.yaml
│
├── scripts/
│   ├── create-stack.sh
│   ├── update-stack.sh
│   └── delete-stack.sh
│
└── docs/
    ├── architecture.md
    ├── troubleshooting.md
    └── best-practices.md
```

---

# 3. Your First CloudFormation Template

Create:

```text
01-basics/hello-world.yaml
```

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: My first AWS CloudFormation stack

Resources:

  MyBucket:
    Type: AWS::S3::Bucket
```

---

# 4. Validate the Template

Always validate before deployment.

```bash
aws cloudformation validate-template \
  --template-body file://01-basics/hello-world.yaml
```

If valid, CloudFormation returns template information.

---

# 5. Create the Stack

```bash
aws cloudformation create-stack \
  --stack-name my-first-stack \
  --template-body file://01-basics/hello-world.yaml
```

Check:

```bash
aws cloudformation describe-stacks \
  --stack-name my-first-stack
```

Check stack events:

```bash
aws cloudformation describe-stack-events \
  --stack-name my-first-stack
```

---

# 6. Check the S3 Bucket

```bash
aws s3 ls
```

CloudFormation created the bucket.

This is the basic IaC workflow:

```text
YAML
  │
  ▼
Validate
  │
  ▼
Create Stack
  │
  ▼
CloudFormation
  │
  ▼
AWS Resources
```

---

# 7. Delete the Stack

For learning resources:

```bash
aws cloudformation delete-stack \
  --stack-name my-first-stack
```

Check:

```bash
aws cloudformation describe-stacks \
  --stack-name my-first-stack
```

After deletion, the stack should no longer exist.

---

# 8. CloudFormation Parameters

Parameters allow users to provide values when deploying a template.

Example:

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: CloudFormation Parameters Demo

Parameters:

  Environment:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - test
      - prod

Resources:

  MyBucket:
    Type: AWS::S3::Bucket

Outputs:

  EnvironmentName:
    Description: Selected environment
    Value: !Ref Environment
```

Deploy:

```bash
aws cloudformation create-stack \
  --stack-name parameter-demo \
  --template-body file://02-parameters/parameters.yaml \
  --parameters ParameterKey=Environment,ParameterValue=dev
```

---

# 9. CloudFormation Outputs

Outputs expose useful information after deployment.

```yaml
Outputs:

  BucketName:
    Description: S3 Bucket Name
    Value: !Ref MyBucket
```

Retrieve:

```bash
aws cloudformation describe-stacks \
  --stack-name parameter-demo \
  --query 'Stacks[0].Outputs'
```

---

# 10. Intrinsic Functions

CloudFormation provides built-in functions.

Important functions:

```text
Ref
Fn::GetAtt
Fn::Join
Fn::Sub
Fn::Select
Fn::Split
Fn::FindInMap
Fn::If
Fn::Equals
Fn::ImportValue
```

Short syntax:

```yaml
!Ref MyBucket
```

```yaml
!GetAtt MyBucket.Arn
```

```yaml
!Sub "Hello ${AWS::Region}"
```

---

# 11. AWS Pseudo Parameters

CloudFormation provides predefined parameters.

Important examples:

```yaml
AWS::AccountId
AWS::Region
AWS::StackName
AWS::StackId
AWS::Partition
AWS::URLSuffix
```

Example:

```yaml
Outputs:

  Region:
    Value: !Ref AWS::Region

  Account:
    Value: !Ref AWS::AccountId

  Stack:
    Value: !Ref AWS::StackName
```

---

# 12. Conditions

Conditions allow resources or properties to change depending on parameters.

Example:

```yaml
Parameters:

  Environment:
    Type: String
    AllowedValues:
      - dev
      - prod

Conditions:

  IsProduction: !Equals
    - !Ref Environment
    - prod
```

Use:

```yaml
Condition: IsProduction
```

Example:

```yaml
Resources:

  ProductionBucket:
    Type: AWS::S3::Bucket
    Condition: IsProduction
```

---

# 13. Mappings

Mappings are useful for static configuration.

Example:

```yaml
Mappings:

  RegionMap:

    us-east-1:
      AMI: ami-example

    ap-south-1:
      AMI: ami-example
```

Retrieve:

```yaml
!FindInMap
  - RegionMap
  - !Ref AWS::Region
  - AMI
```

---

# 14. IAM with CloudFormation

Create an IAM role:

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Resources:

  EC2Role:
    Type: AWS::IAM::Role

    Properties:

      AssumeRolePolicyDocument:

        Version: '2012-10-17'

        Statement:

          - Effect: Allow

            Principal:
              Service:
                - ec2.amazonaws.com

            Action:
              - sts:AssumeRole

      ManagedPolicyArns:

        - arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

Outputs:

  RoleArn:
    Value: !GetAtt EC2Role.Arn
```

---

# 15. Build a VPC

Next, create:

```text
VPC
│
├── Internet Gateway
│
├── Public Subnet
│
├── Private Subnet
│
├── Route Tables
│
├── Routes
│
└── Security Groups
```

Basic VPC:

```yaml
Resources:

  VPC:
    Type: AWS::EC2::VPC

    Properties:

      CidrBlock: 10.0.0.0/16

      Tags:
        - Key: Name
          Value: cloudformation-vpc
```

---

# 16. Create an EC2 Instance

Example:

```yaml
Parameters:

  AmiId:
    Type: AWS::EC2::Image::Id

  InstanceType:
    Type: String
    Default: t3.micro

Resources:

  WebServer:

    Type: AWS::EC2::Instance

    Properties:

      ImageId: !Ref AmiId

      InstanceType: !Ref InstanceType

      Tags:

        - Key: Name
          Value: CloudFormation-WebServer
```

---

# 17. EC2 UserData

Install software automatically:

```yaml
UserData:
  Fn::Base64: |
    #!/bin/bash

    dnf update -y

    dnf install -y nginx

    systemctl enable nginx

    systemctl start nginx
```

This allows CloudFormation to provision infrastructure and bootstrap software.

---

# 18. Security Groups

Example:

```yaml
WebSecurityGroup:

  Type: AWS::EC2::SecurityGroup

  Properties:

    GroupDescription: Web Server Security Group

    SecurityGroupIngress:

      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0

      - IpProtocol: tcp
        FromPort: 443
        ToPort: 443
        CidrIp: 0.0.0.0/0
```

For production, avoid unnecessarily exposing services to:

```text
0.0.0.0/0
```

---

# 19. Application Load Balancer

Architecture:

```text
Internet
   │
   ▼
ALB
   │
   ▼
Target Group
   │
   ├── EC2
   ├── EC2
   └── EC2
```

CloudFormation resources:

```text
AWS::ElasticLoadBalancingV2::LoadBalancer
AWS::ElasticLoadBalancingV2::TargetGroup
AWS::ElasticLoadBalancingV2::Listener
```

---

# 20. Auto Scaling

Architecture:

```text
                ALB
                 │
          ┌──────┴──────┐
          │             │
        EC2           EC2
          │             │
          └──────┬──────┘
                 │
              ASG
```

Resources:

```text
AWS::AutoScaling::LaunchTemplate
AWS::AutoScaling::AutoScalingGroup
```

Example:

```yaml
WebAutoScalingGroup:

  Type: AWS::AutoScaling::AutoScalingGroup

  Properties:

    MinSize: 2

    MaxSize: 4

    DesiredCapacity: 2

    VPCZoneIdentifier:
      - !Ref PublicSubnet1
      - !Ref PublicSubnet2
```

---

# 21. RDS

Create a database using:

```text
AWS::RDS::DBInstance
```

Example:

```yaml
Database:

  Type: AWS::RDS::DBInstance

  DeletionPolicy: Snapshot

  Properties:

    Engine: mysql

    DBInstanceClass: db.t3.micro

    AllocatedStorage: 20

    MasterUsername: admin

    ManageMasterUserPassword: true
```

For production databases, carefully design:

```text
Backup
Encryption
Multi-AZ
DeletionPolicy
Secrets
Network isolation
Monitoring
Maintenance
```

---

# 22. S3

Example:

```yaml
ApplicationBucket:

  Type: AWS::S3::Bucket

  Properties:

    BucketEncryption:

      ServerSideEncryptionConfiguration:

        - ServerSideEncryptionByDefault:
            SSEAlgorithm: AES256

    PublicAccessBlockConfiguration:

      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
```

---

# 23. Stack Updates

Change your template.

Then:

```bash
aws cloudformation update-stack \
  --stack-name my-first-stack \
  --template-body file://template.yaml
```

Monitor:

```bash
aws cloudformation describe-stack-events \
  --stack-name my-first-stack
```

---

# 24. Change Sets

Before changing production infrastructure, create a Change Set.

```bash
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-change \
  --template-body file://template.yaml
```

View:

```bash
aws cloudformation describe-change-set \
  --stack-name my-stack \
  --change-set-name my-change
```

Execute:

```bash
aws cloudformation execute-change-set \
  --stack-name my-stack \
  --change-set-name my-change
```

This helps review what CloudFormation intends to change.

---

# 25. Drift Detection

CloudFormation can detect infrastructure changes made outside CloudFormation.

Start detection:

```bash
aws cloudformation detect-stack-drift \
  --stack-name my-stack
```

Check:

```bash
aws cloudformation describe-stack-drift-detection-status \
  --stack-drift-detection-id <ID>
```

Retrieve resources:

```bash
aws cloudformation describe-stack-resource-drifts \
  --stack-name my-stack
```

Example:

```text
CloudFormation
      │
      ├── Expected configuration
      │
      ▼
     AWS
      │
      └── Actual configuration

Expected != Actual
        │
        ▼
       DRIFT
```

---

# 26. Nested Stacks

Large infrastructure can be separated into stacks.

```text
Root Stack
│
├── Network Stack
│
├── Security Stack
│
├── Compute Stack
│
├── Database Stack
│
└── Monitoring Stack
```

Example:

```yaml
Resources:

  NetworkStack:

    Type: AWS::CloudFormation::Stack

    Properties:

      TemplateURL: network.yaml
```

---

# 27. Cross-Stack References

Export values:

```yaml
Outputs:

  VpcId:

    Value: !Ref VPC

    Export:
      Name: MyVpcId
```

Import:

```yaml
VpcId:
  Fn::ImportValue: MyVpcId
```

This allows independent CloudFormation stacks to communicate.

---

# 28. StackSets

StackSets allow infrastructure deployment across:

```text
Multiple AWS Accounts
        +
Multiple AWS Regions
```

Example:

```text
Management Account
        │
        ▼
   CloudFormation
        │
 ┌──────┼──────┐
 ▼      ▼      ▼
Acct A Acct B Acct C
 │       │      │
us-east ap-south eu-west
```

Useful for organization-wide infrastructure such as:

* IAM
* CloudTrail
* Config
* Guardrails
* Security resources

---

# 29. CloudFormation CLI Workflow

Useful commands:

```bash
aws cloudformation list-stacks
```

```bash
aws cloudformation describe-stacks
```

```bash
aws cloudformation describe-stack-events
```

```bash
aws cloudformation describe-stack-resources
```

```bash
aws cloudformation validate-template
```

```bash
aws cloudformation create-stack
```

```bash
aws cloudformation update-stack
```

```bash
aws cloudformation delete-stack
```

---

# 30. Production Project

Build a complete 3-tier application:

```text
                       Internet
                           │
                           ▼
                  Application LB
                           │
                  ┌────────┴────────┐
                  │                 │
               EC2/ASG           EC2/ASG
                  │                 │
                  └────────┬────────┘
                           │
                    Private Subnet
                           │
                           ▼
                          RDS
```

Infrastructure:

```text
VPC
│
├── Internet Gateway
│
├── Public Subnet A
│   └── ALB
│
├── Public Subnet B
│   └── ALB
│
├── Private App Subnet A
│   └── EC2
│
├── Private App Subnet B
│   └── EC2
│
├── Private DB Subnet A
│   └── RDS
│
└── Private DB Subnet B
    └── RDS
```

Add:

```text
IAM
Security Groups
ALB
Auto Scaling
RDS
CloudWatch
SNS
Secrets Manager
CloudTrail
S3
```

---

# 31. CI/CD

CloudFormation templates should eventually be deployed through CI/CD.

Example:

```text
Developer
    │
    ▼
GitHub
    │
    ▼
Pull Request
    │
    ▼
Validation
    │
    ├── YAML validation
    ├── cfn-lint
    ├── Security scanning
    └── Change Set
    │
    ▼
Approval
    │
    ▼
CloudFormation
    │
    ▼
AWS
```

Recommended tools:

```text
Git
GitHub
GitHub Actions
AWS CLI
cfn-lint
CloudFormation
AWS IAM
```

---

# 32. CloudFormation Best Practices

## Use Parameters

Avoid hardcoding environment-specific values.

```yaml
Parameters:
  Environment:
    Type: String
```

## Use Outputs

Expose important resource identifiers.

```yaml
Outputs:
  VpcId:
    Value: !Ref VPC
```

## Use Tags

```yaml
Tags:
  - Key: Environment
    Value: !Ref Environment
```

## Protect Databases

Consider:

```yaml
DeletionPolicy: Snapshot
```

For resources where accidental deletion must be prevented, evaluate:

```yaml
DeletionPolicy: Retain
```

Use these deliberately because they affect cleanup behavior.

## Use Change Sets

Review production changes before execution.

## Avoid Hardcoding IDs

Instead of:

```yaml
ImageId: ami-xxxxxxxx
```

use parameters, mappings, or Systems Manager parameters where appropriate.

## Separate Environments

```text
dev
test
stage
prod
```

---

# 33. Troubleshooting

## Stack Creation Failed

Check:

```bash
aws cloudformation describe-stack-events \
  --stack-name my-stack
```

Look for:

```text
CREATE_FAILED
UPDATE_FAILED
DELETE_FAILED
ROLLBACK_IN_PROGRESS
ROLLBACK_COMPLETE
```

---

## UPDATE_ROLLBACK_FAILED

First inspect:

```bash
aws cloudformation describe-stack-events \
  --stack-name my-stack
```

Then determine which resource caused the failure before continuing.

---

## Resource Already Exists

Possible causes:

```text
Existing AWS resource
Wrong logical ID
Wrong stack ownership
Resource imported outside CloudFormation
```

---

## IAM AccessDenied

Check:

```bash
aws sts get-caller-identity
```

Then verify the IAM permissions used to create/update the stack.

---

# 34. Learning Roadmap

### Beginner

```text
Day 1
 ├── CloudFormation concepts
 ├── Templates
 ├── Resources
 └── First S3 stack

Day 2
 ├── Parameters
 ├── Outputs
 └── Pseudo parameters

Day 3
 ├── Ref
 ├── GetAtt
 ├── Sub
 └── Join

Day 4
 ├── Conditions
 └── Mappings

Day 5
 ├── IAM
 └── EC2
```

### Intermediate

```text
Week 2

VPC
EC2
Security Groups
ALB
Auto Scaling
RDS
S3
CloudWatch
SNS
```

### Advanced

```text
Week 3

Nested Stacks
Cross-Stack References
Change Sets
Drift Detection
StackSets
Secrets Manager
CloudFormation security
CloudFormation CI/CD
```

### Production

```text
Week 4

3-Tier Architecture
Multi-AZ
Security
Monitoring
Backup
Disaster Recovery
CI/CD
Infrastructure testing
```

---

# 35. Interview Topics

Prepare these CloudFormation questions:

### Fundamentals

* What is AWS CloudFormation?
* What is Infrastructure as Code?
* What is a CloudFormation stack?
* What is a template?
* What are Resources?
* What are Parameters?
* What are Outputs?

### Advanced

* `Ref` vs `GetAtt`
* Parameters vs Mappings
* Conditions
* Nested stacks
* Cross-stack references
* Change Sets
* Drift Detection
* StackSets
* Rollbacks
* DeletionPolicy
* UpdateReplacePolicy
* DependsOn
* CreationPolicy
* UpdatePolicy
* WaitCondition

### Production

* How do you protect an RDS database?
* How do you manage secrets?
* How do you deploy CloudFormation through CI/CD?
* How do you detect configuration drift?
* How do you handle a failed stack update?
* How do you deploy infrastructure across multiple accounts?
* How do you organize large CloudFormation templates?

---

# 36. Final Project

The final repository project should look like:

```text
aws-cloudformation-learning/
│
├── README.md
│
├── templates/
│   ├── network.yaml
│   ├── security.yaml
│   ├── compute.yaml
│   ├── loadbalancer.yaml
│   ├── database.yaml
│   └── monitoring.yaml
│
├── environments/
│   ├── dev.yaml
│   ├── test.yaml
│   └── prod.yaml
│
├── scripts/
│   ├── deploy.sh
│   ├── update.sh
│   └── destroy.sh
│
├── .github/
│   └── workflows/
│       └── cloudformation.yml
│
└── docs/
    ├── architecture.md
    ├── troubleshooting.md
    └── best-practices.md
```

Final deployment:

```text
GitHub
   │
   ▼
GitHub Actions
   │
   ├── cfn-lint
   ├── Validate
   ├── Security Scan
   └── Change Set
          │
          ▼
    CloudFormation
          │
          ▼
       AWS VPC
          │
    ┌─────┴─────┐
    ▼           ▼
   ALB          RDS
    │
    ▼
  EC2 ASG
    │
    ▼
CloudWatch
```

---

# 🎯 Goal

By completing this repository, you should be able to go from:

```text
CloudFormation YAML
        ↓
Validate
        ↓
Create Stack
        ↓
Build VPC
        ↓
Deploy EC2
        ↓
Add ALB
        ↓
Add Auto Scaling
        ↓
Add RDS
        ↓
Add Monitoring
        ↓
Use Change Sets
        ↓
Detect Drift
        ↓
Deploy through CI/CD
        ↓
Production AWS Infrastructure
```

## 🚀 Start Here

Your first lab:

```bash
git clone <your-repository>

cd aws-cloudformation-learning

aws sts get-caller-identity

aws cloudformation validate-template \
  --template-body file://01-basics/hello-world.yaml

aws cloudformation create-stack \
  --stack-name my-first-stack \
  --template-body file://01-basics/hello-world.yaml
```

Then verify:

```bash
aws cloudformation describe-stacks \
  --stack-name my-first-stack
```

AWSTemplateFormatVersion: '2010-09-09'

Description: Application Load Balancer Lab

Parameters:

  VpcId:
    Type: AWS::EC2::VPC::Id

  PublicSubnet1:
    Type: AWS::EC2::Subnet::Id

  PublicSubnet2:
    Type: AWS::EC2::Subnet::Id

Resources:

  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP traffic to ALB
      VpcId: !Ref VpcId

      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: cloudformation-alb
      Scheme: internet-facing
      Type: application

      SecurityGroups:
        - !Ref ALBSecurityGroup

      Subnets:
        - !Ref PublicSubnet1
        - !Ref PublicSubnet2

  WebTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: cloudformation-web-tg
      Port: 80
      Protocol: HTTP
      VpcId: !Ref VpcId
      TargetType: instance

      HealthCheckEnabled: true
      HealthCheckPath: /
      HealthCheckProtocol: HTTP
      HealthCheckPort: traffic-port

  HTTPListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref ApplicationLoadBalancer
      Port: 80
      Protocol: HTTP

      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref WebTargetGroup

Outputs:

  LoadBalancerDNS:
    Description: ALB DNS name
    Value: !GetAtt ApplicationLoadBalancer.DNSName

  TargetGroupArn:
    Description: Target Group ARN
    Value: !Ref WebTargetGroup

**Next milestone:** build the same infrastructure first with CloudFormation, then compare the design with Terraform to understand where each IaC approach fits.
