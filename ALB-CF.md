# AWS CloudFormation — Application Load Balancer Stack

This lab creates a complete **Application Load Balancer stack** using AWS CloudFormation.

## Architecture

```text
                         Internet
                            │
                            │ HTTP :80
                            ▼
                  ┌───────────────────┐
                  │ Application       │
                  │ Load Balancer     │
                  └─────────┬─────────┘
                            │
                       Target Group
                       /           \
                      /             \
                     ▼               ▼
               ┌─────────┐     ┌─────────┐
               │  EC2-1  │     │  EC2-2  │
               │ Nginx   │     │ Nginx   │
               └─────────┘     └─────────┘
                     │               │
                     └───────┬───────┘
                             │
                       Private/Backend
```

---

# 1. Repository Structure

```text
aws-cloudformation-learning/
│
├── README.md
│
├── 09-load-balancer/
│   ├── alb-stack.yaml
│   └── README.md
│
└── scripts/
    ├── deploy-alb.sh
    └── delete-alb.sh
```

---

# 2. CloudFormation Template

Create:

```text
09-load-balancer/alb-stack.yaml
```

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: >
  Complete Application Load Balancer stack with VPC,
  public subnets, EC2 instances, target group and listener.

Parameters:

  VpcCidr:
    Type: String
    Default: 10.0.0.0/16
    Description: VPC CIDR block

  PublicSubnet1Cidr:
    Type: String
    Default: 10.0.1.0/24
    Description: First public subnet CIDR

  PublicSubnet2Cidr:
    Type: String
    Default: 10.0.2.0/24
    Description: Second public subnet CIDR

  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium

  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64

Resources:

  # --------------------------------------------------
  # VPC
  # --------------------------------------------------

  VPC:
    Type: AWS::EC2::VPC

    Properties:
      CidrBlock: !Ref VpcCidr

      EnableDnsSupport: true
      EnableDnsHostnames: true

      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-vpc'

  # --------------------------------------------------
  # Internet Gateway
  # --------------------------------------------------

  InternetGateway:
    Type: AWS::EC2::InternetGateway

    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-igw'

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment

    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  # --------------------------------------------------
  # Public Subnet 1
  # --------------------------------------------------

  PublicSubnet1:
    Type: AWS::EC2::Subnet

    Properties:
      VpcId: !Ref VPC

      CidrBlock: !Ref PublicSubnet1Cidr

      AvailabilityZone:
        Fn::Select:
          - 0
          - Fn::GetAZs: !Ref AWS::Region

      MapPublicIpOnLaunch: true

      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-public-subnet-1'

  # --------------------------------------------------
  # Public Subnet 2
  # --------------------------------------------------

  PublicSubnet2:
    Type: AWS::EC2::Subnet

    Properties:
      VpcId: !Ref VPC

      CidrBlock: !Ref PublicSubnet2Cidr

      AvailabilityZone:
        Fn::Select:
          - 1
          - Fn::GetAZs: !Ref AWS::Region

      MapPublicIpOnLaunch: true

      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-public-subnet-2'

  # --------------------------------------------------
  # Route Table
  # --------------------------------------------------

  PublicRouteTable:
    Type: AWS::EC2::RouteTable

    Properties:
      VpcId: !Ref VPC

      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-public-rt'

  # --------------------------------------------------
  # Internet Route
  # --------------------------------------------------

  DefaultRoute:
    Type: AWS::EC2::Route

    DependsOn: AttachGateway

    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  # --------------------------------------------------
  # Route Table Associations
  # --------------------------------------------------

  PublicSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation

    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet1

  PublicSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation

    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet2

  # --------------------------------------------------
  # ALB Security Group
  # --------------------------------------------------

  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup

    Properties:
      GroupDescription: Allow HTTP traffic to ALB
      VpcId: !Ref VPC

      SecurityGroupIngress:

        - Description: HTTP from Internet
          IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

      SecurityGroupEgress:

        - Description: Allow outbound traffic
          IpProtocol: -1
          CidrIp: 0.0.0.0/0

      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-alb-sg'

  # --------------------------------------------------
  # EC2 Security Group
  # --------------------------------------------------

  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup

    Properties:
      GroupDescription: Allow HTTP traffic from ALB
      VpcId: !Ref VPC

      SecurityGroupIngress:

        - Description: HTTP from ALB
          IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          SourceSecurityGroupId: !Ref ALBSecurityGroup

      SecurityGroupEgress:

        - Description: Allow outbound traffic
          IpProtocol: -1
          CidrIp: 0.0.0.0/0

      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-web-sg'

  # --------------------------------------------------
  # EC2 Instance 1
  # --------------------------------------------------

  WebServer1:
    Type: AWS::EC2::Instance

    Properties:

      ImageId: !Ref LatestAmiId

      InstanceType: !Ref InstanceType

      SubnetId: !Ref PublicSubnet1

      SecurityGroupIds:
        - !Ref WebSecurityGroup

      UserData:
        Fn::Base64: |
          #!/bin/bash

          dnf update -y

          dnf install -y nginx

          systemctl enable nginx
          systemctl start nginx

          echo "<h1>CloudFormation ALB - Server 1</h1>" > /usr/share/nginx/html/index.html

      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-server-1'

  # --------------------------------------------------
  # EC2 Instance 2
  # --------------------------------------------------

  WebServer2:
    Type: AWS::EC2::Instance

    Properties:

      ImageId: !Ref LatestAmiId

      InstanceType: !Ref InstanceType

      SubnetId: !Ref PublicSubnet2

      SecurityGroupIds:
        - !Ref WebSecurityGroup

      UserData:
        Fn::Base64: |
          #!/bin/bash

          dnf update -y

          dnf install -y nginx

          systemctl enable nginx
          systemctl start nginx

          echo "<h1>CloudFormation ALB - Server 2</h1>" > /usr/share/nginx/html/index.html

      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-server-2'

  # --------------------------------------------------
  # Application Load Balancer
  # --------------------------------------------------

  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer

    Properties:

      Name: !Sub '${AWS::StackName}-alb'

      Scheme: internet-facing

      Type: application

      SecurityGroups:
        - !Ref ALBSecurityGroup

      Subnets:
        - !Ref PublicSubnet1
        - !Ref PublicSubnet2

      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-alb'

  # --------------------------------------------------
  # Target Group
  # --------------------------------------------------

  WebTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup

    Properties:

      Name: !Sub '${AWS::StackName}-tg'

      VpcId: !Ref VPC

      Protocol: HTTP

      Port: 80

      TargetType: instance

      Targets:

        - Id: !Ref WebServer1
          Port: 80

        - Id: !Ref WebServer2
          Port: 80

      HealthCheckEnabled: true

      HealthCheckProtocol: HTTP

      HealthCheckPath: /

      HealthCheckPort: traffic-port

      HealthCheckIntervalSeconds: 30

      HealthyThresholdCount: 2

      UnhealthyThresholdCount: 3

      Matcher:
        HttpCode: 200

      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-target-group'

  # --------------------------------------------------
  # ALB Listener
  # --------------------------------------------------

  HTTPListener:
    Type: AWS::ElasticLoadBalancingV2::Listener

    Properties:

      LoadBalancerArn: !Ref ApplicationLoadBalancer

      Port: 80

      Protocol: HTTP

      DefaultActions:

        - Type: forward
          TargetGroupArn: !Ref WebTargetGroup

# --------------------------------------------------
# Outputs
# --------------------------------------------------

Outputs:

  StackName:
    Description: CloudFormation Stack Name
    Value: !Ref AWS::StackName

  VpcId:
    Description: VPC ID
    Value: !Ref VPC

  ALBName:
    Description: Application Load Balancer name
    Value: !Ref ApplicationLoadBalancer

  ALBDNSName:
    Description: Application Load Balancer DNS name
    Value: !GetAtt ApplicationLoadBalancer.DNSName

  TargetGroup:
    Description: Target Group ARN
    Value: !Ref WebTargetGroup

  Server1:
    Description: EC2 Server 1
    Value: !Ref WebServer1

  Server2:
    Description: EC2 Server 2
    Value: !Ref WebServer2

  ApplicationURL:
    Description: Application URL
    Value: !Sub 'http://${ApplicationLoadBalancer.DNSName}'
```

---

# 3. Validate the CloudFormation Template

Run:

```bash
aws cloudformation validate-template \
  --template-body file://09-load-balancer/alb-stack.yaml
```

If successful, CloudFormation accepts the template syntax.

---

# 4. Create the CloudFormation Stack

```bash
aws cloudformation create-stack \
  --stack-name cf-alb-lab \
  --template-body file://09-load-balancer/alb-stack.yaml
```

Notice that we are creating **one CloudFormation stack**.

```text
cf-alb-lab
    │
    ├── VPC
    ├── Subnets
    ├── IGW
    ├── Route Table
    ├── Security Groups
    ├── EC2 #1
    ├── EC2 #2
    ├── ALB
    ├── Target Group
    └── Listener
```

---

# 5. Monitor the Stack

```bash
aws cloudformation describe-stacks \
  --stack-name cf-alb-lab
```

Watch events:

```bash
aws cloudformation describe-stack-events \
  --stack-name cf-alb-lab
```

Or:

```bash
aws cloudformation wait stack-create-complete \
  --stack-name cf-alb-lab
```

---

# 6. Get the ALB URL

```bash
aws cloudformation describe-stacks \
  --stack-name cf-alb-lab \
  --query 'Stacks[0].Outputs'
```

You should see:

```text
ALBDNSName
ApplicationURL
Server1
Server2
TargetGroup
VpcId
```

Get only the URL:

```bash
aws cloudformation describe-stacks \
  --stack-name cf-alb-lab \
  --query 'Stacks[0].Outputs[?OutputKey==`ApplicationURL`].OutputValue' \
  --output text
```

Example:

```text
http://cf-alb-lab-alb-123456789.ap-south-1.elb.amazonaws.com
```

---

# 7. Test the Application

```bash
curl http://<ALB-DNS-NAME>
```

You should receive something similar to:

```html
<h1>CloudFormation ALB - Server 1</h1>
```

Run it again:

```bash
curl http://<ALB-DNS-NAME>
```

You may receive:

```html
<h1>CloudFormation ALB - Server 2</h1>
```

The ALB distributes requests between the healthy targets.

---

# 8. Check Target Health

First get the Target Group ARN:

```bash
aws cloudformation describe-stacks \
  --stack-name cf-alb-lab \
  --query 'Stacks[0].Outputs[?OutputKey==`TargetGroup`].OutputValue' \
  --output text
```

Then:

```bash
aws elbv2 describe-target-health \
  --target-group-arn <TARGET-GROUP-ARN>
```

Expected:

```text
Target 1 → healthy
Target 2 → healthy
```

---

# 9. Understand the Traffic Flow

```text
Browser
   │
   │ HTTP :80
   ▼
ALB Security Group
   │
   ▼
Application Load Balancer
   │
   ▼
Listener :80
   │
   ▼
Target Group
   │
   ├───────────────┐
   ▼               ▼
EC2 Server 1    EC2 Server 2
   │               │
   ▼               ▼
 Nginx           Nginx
```

The important security design is:

```text
Internet
   │
   ▼
ALB
 │
 └── ALB Security Group
          │
          ▼
     Web Security Group
          │
          ▼
       EC2 :80
```

The EC2 instances **do not allow HTTP directly from the Internet**.

They only allow HTTP from the ALB security group.

---

# 10. View Resources Created by the Stack

```bash
aws cloudformation list-stack-resources \
  --stack-name cf-alb-lab
```

You should see resources such as:

```text
AWS::EC2::VPC
AWS::EC2::InternetGateway
AWS::EC2::Subnet
AWS::EC2::RouteTable
AWS::EC2::SecurityGroup
AWS::EC2::Instance
AWS::ElasticLoadBalancingV2::LoadBalancer
AWS::ElasticLoadBalancingV2::TargetGroup
AWS::ElasticLoadBalancingV2::Listener
```

This is the key CloudFormation concept:

```text
Template
   │
   ▼
CloudFormation Stack
   │
   ├── Resource
   ├── Resource
   ├── Resource
   └── Resource
```

---

# 11. Update the Stack

Change Server 1:

```yaml
echo "<h1>CloudFormation ALB - SERVER 1 UPDATED</h1>" > /usr/share/nginx/html/index.html
```

Then update:

```bash
aws cloudformation update-stack \
  --stack-name cf-alb-lab \
  --template-body file://09-load-balancer/alb-stack.yaml
```

For production, use a **Change Set** before executing infrastructure changes.

---

# 12. Create a Change Set

```bash
aws cloudformation create-change-set \
  --stack-name cf-alb-lab \
  --change-set-name alb-update \
  --template-body file://09-load-balancer/alb-stack.yaml
```

Review:

```bash
aws cloudformation describe-change-set \
  --stack-name cf-alb-lab \
  --change-set-name alb-update
```

Then execute:

```bash
aws cloudformation execute-change-set \
  --stack-name cf-alb-lab \
  --change-set-name alb-update
```

---

# 13. Delete the Entire Lab

Because all resources belong to the stack:

```bash
aws cloudformation delete-stack \
  --stack-name cf-alb-lab
```

Wait:

```bash
aws cloudformation wait stack-delete-complete \
  --stack-name cf-alb-lab
```

The stack manages the lifecycle of its resources.

---

# 14. Important CloudFormation Concept

The difference is:

### Manual AWS CLI approach

```text
Create VPC
   ↓
Create Subnet
   ↓
Create EC2
   ↓
Create Target Group
   ↓
Register EC2
   ↓
Create ALB
   ↓
Create Listener
```

You manually manage every resource.

### CloudFormation approach

```text
              alb-stack.yaml
                    │
                    ▼
             CloudFormation
                    │
                    ▼
              cf-alb-lab
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
      VPC           EC2          ALB
       │             │            │
    Subnets      Targets      Listener
                                  │
                                  ▼
                            Target Group
```

**The YAML template becomes the source of truth for the infrastructure.**

---

# 15. Next Level

After this single-stack ALB lab, evolve it into:

```text
                    CloudFormation
                          │
              ┌───────────┴───────────┐
              │                       │
        Network Stack            Application Stack
              │                       │
       ┌──────┴──────┐                │
       │             │                │
      VPC         Subnets             │
                                      │
                              ┌───────┴───────┐
                              │               │
                             ALB             ASG
                              │               │
                              │          EC2 EC2 EC2
                              │               │
                              └───────┬───────┘
                                      │
                                     RDS
```

Then add:

```text
ALB
 ├── HTTP → HTTPS redirect
 ├── HTTPS :443
 ├── ACM Certificate
 ├── Listener Rules
 ├── Path-based routing
 ├── Host-based routing
 ├── Health Checks
 ├── WAF
 └── Access Logs
```

This gives you a proper **CloudFormation ALB → Auto Scaling → RDS production-style lab**, rather than just creating an ALB resource by itself.
