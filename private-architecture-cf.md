
                              INTERNET
                                  │
                                  │ HTTP :80 / HTTPS :443
                                  ▼
                    ┌──────────────────────────┐
                    │   Internet Gateway       │
                    └────────────┬─────────────┘
                                 │
                                 ▼
              ┌─────────────────────────────────────┐
              │             PUBLIC SUBNETS           │
              │                                     │
              │     ┌─────────────────────────┐     │
              │     │ Application Load        │     │
              │     │ Balancer (ALB)          │     │
              │     └────────────┬────────────┘     │
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 │
                          Target Group
                                 │
                ┌────────────────┴────────────────┐
                │                                 │
                ▼                                 ▼
       ┌─────────────────┐              ┌─────────────────┐
       │ PRIVATE SUBNET 1│              │ PRIVATE SUBNET 2│
       │                 │              │                 │
       │  ┌───────────┐  │              │  ┌───────────┐  │
       │  │  EC2      │  │              │  │  EC2      │  │
       │  │ Backend   │  │              │  │ Backend   │  │
       │  │ :80       │  │              │  │ :80       │  │
       │  └───────────┘  │              │  └───────────┘  │
       │                 │              │                 │
       └────────┬────────┘              └────────┬────────┘
                │                                │
                └───────────────┬────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │ PRIVATE DB      │
                       │ SUBNETS         │
                       │                 │
                       │      RDS        │
                       └─────────────────┘



Network Layout:
VPC: 10.0.0.0/16
│
├── Public Subnet AZ-1
│   └── ALB
│
├── Public Subnet AZ-2
│   └── ALB
│
├── Private App Subnet AZ-1
│   └── EC2 / Application
│
├── Private App Subnet AZ-2
│   └── EC2 / Application
│
├── Private DB Subnet AZ-1
│   └── RDS
│
└── Private DB Subnet AZ-2
    └── RDS


Security group flow:

Internet
   │
   │ 80/443
   ▼
┌──────────────┐
│ ALB-SG       │
└──────┬───────┘
       │
       │ 80
       ▼
┌──────────────┐
│ Backend-SG   │
│ Source:      │
│ ALB-SG only  │
└──────┬───────┘
       │
       │ DB port
       ▼
┌──────────────┐
│ Database-SG  │
│ Source:      │
│ Backend-SG   │
└──────────────┘


or

                         INTERNET
                             │
                             ▼
                    ┌────────────────┐
                    │ Internet       │
                    │ Gateway        │
                    └───────┬────────┘
                            │
                 ┌──────────┴──────────┐
                 │     PUBLIC SUBNETS   │
                 │                     │
                 │   ┌─────────────┐   │
                 │   │     ALB     │   │
                 │   │  :80 / :443 │   │
                 │   └──────┬──────┘   │
                 └──────────┼──────────┘
                            │
                       Target Group
                            │
             ┌──────────────┴──────────────┐
             │                             │
             ▼                             ▼
      ┌──────────────┐              ┌──────────────┐
      │ PRIVATE APP  │              │ PRIVATE APP  │
      │ SUBNET AZ-1  │              │ SUBNET AZ-2  │
      │              │              │              │
      │    EC2-1     │              │    EC2-2     │
      │   Nginx/App  │              │   Nginx/App  │
      │  NO PUBLIC IP│              │  NO PUBLIC IP│
      └──────┬───────┘              └──────┬───────┘
             │                             │
             └──────────────┬──────────────┘
                            │
                            ▼
                   ┌─────────────────┐
                   │ PRIVATE DB      │
                   │ SUBNETS         │
                   │                 │
                   │      RDS        │
                   └─────────────────┘

Private EC2 ──► NAT Gateway ──► Internet Gateway
                 outbound only

CloudFormation stack structure

cf-alb-private-backend
│
├── VPC
│
├── Internet Gateway
│
├── Public Subnet AZ-1
│   └── ALB
│
├── Public Subnet AZ-2
│   └── ALB
│
├── NAT Gateway AZ-1
│
├── NAT Gateway AZ-2
│
├── Private App Subnet AZ-1
│   └── EC2
│
├── Private App Subnet AZ-2
│   └── EC2
│
├── ALB Security Group
│
├── Backend Security Group
│
├── Target Group
│
├── ALB Listener
│
└── Outputs

And the security model should be:

Internet
   │
   │ 80/443
   ▼
 ALB-SG
   │
   │ 80
   ▼
Backend-SG
   │
   │ DB port
   ▼
Database-SG

Note:
No 0.0.0.0/0 rule should exist on the backend EC2 security group.
                 
