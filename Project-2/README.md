# Cloud Audit and Insigth - Command Center
## Motive: Collect and Analyse the logs

https://github.com/user-attachments/assets/bed5933a-416c-41af-aeaf-907d4a1f5bd2

## Architecture
# ☁️ CloudGuard Security Scanner

A serverless AWS security scanning platform that inventories cloud resources, evaluates security and configuration posture, stores findings and scores, generates reports, and exposes results through a web dashboard and API.

## 🎥 Project Demo

Watch the complete project demonstration:

<video src="./Screen%20Recording%202026-07-26%20164931.mp4" controls width="900"></video>

> **Note:** If GitHub does not render the repository-hosted video, drag the MP4 directly into the GitHub README editor and use the generated GitHub attachment URL.

## 🏗️ Architecture

    ╔══════════════════════════════════════════════════════════════════════════════════════════════════╗
    ║                                                                                                  ║
    ║        ██████╗██╗      ██████╗ ██╗   ██╗██████╗  ██████╗ ██╗   ██╗ █████╗ ██████╗ ██████╗      ║
    ║       ██╔════╝██║     ██╔═══██╗██║   ██║██╔══██╗██╔════╝ ██║   ██║██╔══██╗██╔══██╗██╔══██╗     ║
    ║       ██║     ██║     ██║   ██║██║   ██║██║  ██║██║  ███╗██║   ██║███████║██████╔╝██║  ██║     ║
    ║       ██║     ██║     ██║   ██║██║   ██║██║  ██║██║   ██║██║   ██║██╔══██║██╔══██╗██║  ██║     ║
    ║       ╚██████╗███████╗╚██████╔╝╚██████╔╝██████╔╝╚██████╔╝╚██████╔╝██║  ██║██║  ██║██████╔╝     ║
    ║        ╚═════╝╚══════╝ ╚═════╝  ╚═════╝ ╚═════╝  ╚═════╝  ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝╚═════╝     ║
    ║                                                                                                  ║
    ║                    Security Scanner Platform  |  Account: 282627753593  |  us-east-1            ║
    ╚══════════════════════════════════════════════════════════════════════════════════════════════════╝
    
                                        ┌─────────────────────────────┐
                                        │         🌍 INTERNET          │
                                        └──────────┬──────────┬────────┘
                                                   │          │
                                  ┌────────────────┘          └─────────────────┐
                                  │  Static Website                  API Calls  │
                                  ▼                                              ▼
                  ┌───────────────────────────┐              ┌───────────────────────────────┐
                  │       ☁️  CLOUDFRONT       │              │    🔀 API GATEWAY (HTTP v2)    │
                  │                           │              │                               │
                  │  ID : E2WTPUAINAOTNP      │              │  ID  : p7ekrfelgi             │
                  │  URL: d392runfkyxnze      │              │  URL : p7ekrfelgi.execute-api │
                  │       .cloudfront.net     │              │        .us-east-1.amazonaws   │
                  │                           │              │        .com                   │
                  │  OAC : E93V863CQWE5B      │              │                               │
                  │  TLS : redirect-to-https  │              │  Routes:                      │
                  │  Root: index.html         │              │  ┌─ GET  /                    │
                  │  Price: PriceClass_All    │              │  ├─ GET  /findings            │
                  └──────────────┬────────────┘              │  ├─ GET  /scores              │
                                 │                           │  ├─ GET  /summary             │
                        OAC Signed│Request                   │  ├─ POST /scan                │
                        (SigV4 + HTTPS)                      │  └─ GET  /{proxy+}            │
                                 │                           └──────────────┬────────────────┘
                                 │                                          │ AWS_PROXY
                                 ▼                                          ▼
                  ┌───────────────────────────┐              ┌───────────────────────────────┐
                  │  🪣 S3: Website Hosting    │              │  λ Lambda: CloudGuard-API     │
                  │                           │              │                               │
                  │  s3-website-hosting-      │              │  Runtime : Python 3.14        │
                  │  282627753593-us-east-1-an│              │  Memory  : 128 MB             │
                  │                           │              │  Timeout : 3s / 30s (APIGW)   │
                  │  📄 index.html (15.5 KB)  │              │  VPC     : ❌ Public Lambda   │
                  │  Website Hosting: ON      │              │  Role    : CloudGuard-        │
                  │  Bucket Policy : OAC only │              │            Lambda-API-Role    │
                  │  Public Access : OFF      │              │                               │
                  └───────────────────────────┘              │  Reads  → DynamoDB Findings  │
                                                             │  Reads  → DynamoDB Scores    │
                                                             │  Invokes→ Scanner Lambda     │
                                                             └──────────────┬────────────────┘
                                                                            │ Invoke
                                                                            │ (POST /scan)
                                                                            ▼
    ╔══════════════════════════════════════════════════════════════════════════════════════════════════╗
    ║  🏗️  project-vpc  |  vpc-0261a5e1b2611731f  |  CIDR: 10.0.0.0/16  |  us-east-1                 ║
    ║                                                                                                  ║
    ║  ┌────────────────────────────────────────────────────────────────────────────────────────────┐ ║
    ║  │  🌐 Internet Gateway: project-igw  (igw-0f62206a9c8223844)  — Attached to project-vpc     │ ║
    ║  └────────────────────────────────────────────────────────────────────────────────────────────┘ ║
    ║                                                                                                  ║
    ║  ╔══════════════════════════════════════════════════════════════════════════════════════════╗   ║
    ║  ║  🟢 PUBLIC SUBNETS                                                                       ║   ║
    ║  ╠═════════════════════════════════════════╦════════════════════════════════════════════════╣   ║
    ║  ║  📌 project-subnet-public1              ║  📌 project-subnet-public2                    ║   ║
    ║  ║  subnet-00c429d1e383ebf80               ║  subnet-0786858262c5b9afa                     ║   ║
    ║  ║  CIDR : 10.0.0.0/20                     ║  CIDR : 10.0.16.0/20                          ║   ║
    ║  ║  AZ   : us-east-1a                      ║  AZ   : us-east-1b                            ║   ║
    ║  ║  Route: 0.0.0.0/0 → IGW ✅             ║  Route: 0.0.0.0/0 → IGW ✅                   ║   ║
    ║  ║                                         ║                                               ║   ║
    ║  ║  ⚠️  No NAT Gateway                     ║  ⚠️  No NAT Gateway                           ║   ║
    ║  ║  [RDS DB Subnet Group Member]           ║  [RDS DB Subnet Group Member]                 ║   ║
    ║  ╚═════════════════════════════════════════╩════════════════════════════════════════════════╝   ║
    ║                                                                                                  ║
    ║  ╔══════════════════════════════════════════════════════════════════════════════════════════╗   ║
    ║  ║  🔴 PRIVATE SUBNETS                                                                      ║   ║
    ║  ╠═════════════════════════════════════════╦════════════════════════════════════════════════╣   ║
    ║  ║  📌 project-subnet-private1             ║  📌 project-subnet-private2                   ║   ║
    ║  ║  subnet-03e1a4eb50c7bc403               ║  subnet-0df113a2995a0a1ca                     ║   ║
    ║  ║  CIDR : 10.0.128.0/20                   ║  CIDR : 10.0.144.0/20                         ║   ║
    ║  ║  AZ   : us-east-1a                      ║  AZ   : us-east-1b                            ║   ║
    ║  ║  Route: 10.0.0.0/16 → local             ║  Route: 10.0.0.0/16 → local                  ║   ║
    ║  ║  Route: S3  prefix  → S3  Gateway       ║  Route: S3  prefix  → S3  Gateway             ║   ║
    ║  ║  Route: DDB prefix  → DDB Gateway       ║  Route: DDB prefix  → DDB Gateway             ║   ║
    ║  ║                                         ║                                               ║   ║
    ║  ║  ┌─────────────────────────────────┐    ║  ┌─────────────────────────────────┐          ║   ║
    ║  ║  │  λ CloudGuard-Resource-Scanner  │    ║  │  λ CloudGuard-Resource-Scanner  │          ║   ║
    ║  ║  │  Runtime : Python 3.14          │    ║  │  (Multi-AZ Network Interface)   │          ║   ║
    ║  ║  │  Memory  : 512 MB               │    ║  │  sg-063939ad3450fba50           │          ║   ║
    ║  ║  │  Timeout : 120s                 │    ║  └─────────────────────────────────┘          ║   ║
    ║  ║  │  Role    : Scanner-Role         │    ║                                               ║   ║
    ║  ║  │  sg-063939ad3450fba50           │    ║  ┌─────────────────────────────────┐          ║   ║
    ║  ║  └─────────────────────────────────┘    ║  │  🗄️  RDS: cloudguard-audit-db   │          ║   ║
    ║  ║                                         ║  │  Engine  : PostgreSQL 18.3      │          ║   ║
    ║  ║  ┌─────────────────────────────────┐    ║  │  Class   : db.t3.micro          │          ║   ║
    ║  ║  │  🗄️  RDS: cloudguard-audit-db   │    ║  │  Storage : 20 GB gp2 🔐 Enc    │          ║   ║
    ║  ║  │  Engine  : PostgreSQL 18.3      │    ║  │  Port    : 5432                 │          ║   ║
    ║  ║  │  Class   : db.t3.micro          │    ║  │  sg-054a310af826d3251           │          ║   ║
    ║  ║  │  Storage : 20 GB gp2 🔐 Enc    │    ║  └─────────────────────────────────┘          ║   ║
    ║  ║  │  Port    : 5432                 │    ║                                               ║   ║
    ║  ║  │  sg-054a310af826d3251           │    ║                                               ║   ║
    ║  ║  └─────────────────────────────────┘    ║                                               ║   ║
    ║  ╚═════════════════════════════════════════╩════════════════════════════════════════════════╝   ║
    ║                                                                                                  ║
    ║  ╔══════════════════════════════════════════════════════════════════════════════════════════╗   ║
    ║  ║  🔗 VPC ENDPOINTS                                                                        ║   ║
    ║  ╠══════════════════════════════════════════════════════════════════════════════════════════╣   ║
    ║  ║                                                                                          ║   ║
    ║  ║  GATEWAY ENDPOINTS  (Route Table — No cost)                                             ║   ║
    ║  ║  ┌──────────────────────────────────┐   ┌──────────────────────────────────┐           ║   ║
    ║  ║  │  🪣 S3 Gateway                   │   │  📊 DynamoDB Gateway             │           ║   ║
    ║  ║  │  vpce-0a6c3c525436ac2d7          │   │  vpce-02fd5d0d151cbd89f          │           ║   ║
    ║  ║  │  Both private route tables ✅    │   │  Both private route tables ✅    │           ║   ║
    ║  ║  └──────────────────────────────────┘   └──────────────────────────────────┘           ║   ║
    ║  ║                                                                                          ║   ║
    ║  ║  INTERFACE ENDPOINTS  (DNS based — sg-064b45dbda60872d0 — Private DNS ✅)              ║   ║
    ║  ║                                                                                          ║   ║
    ║  ║  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐            ║   ║
    ║  ║  │  🔑 STS   │  │  👤 IAM   │  │  🔔 SNS   │  │ 🔐 Secrets│  │  🗄️  RDS  │            ║   ║
    ║  ║  │  vpce-    │  │  vpce-    │  │  vpce-    │  │  Manager  │  │  vpce-    │            ║   ║
    ║  ║  │  0c731ece │  │  094cea0e │  │  004d0ebc │  │  vpce-    │  │  0bb6198  │            ║   ║
    ║  ║  │  PrivDNS✅│  │  PrivDNS✅│  │  PrivDNS✅│  │  061b4a78 │  │  PrivDNS✅│            ║   ║
    ║  ║  └───────────┘  └───────────┘  └───────────┘  │  PrivDNS✅│  └───────────┘            ║   ║
    ║  ║                                                └───────────┘                           ║   ║
    ║  ║  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐                           ║   ║
    ║  ║  │  💻 EC2   │  │ 📋 CW     │  │ 📈 CW     │  │ 🔍 Cloud  │                           ║   ║
    ║  ║  │  vpce-    │  │  Logs     │  │  Monitoring│  │  Trail    │                           ║   ║
    ║  ║  │  01adedb6 │  │  vpce-    │  │  vpce-    │  │  vpce-    │                           ║   ║
    ║  ║  │  PrivDNS✅│  │  0cd8124d │  │  0add3277 │  │  0ecac455 │                           ║   ║
    ║  ║  └───────────┘  │  PrivDNS✅│  │  PrivDNS✅│  │  PrivDNS✅│                           ║   ║
    ║  ║                 └───────────┘  └───────────┘  └───────────┘                           ║   ║
    ║  ╚══════════════════════════════════════════════════════════════════════════════════════════╝   ║
    ╚══════════════════════════════════════════════════════════════════════════════════════════════════╝

## ☁️ AWS Managed Services

    ║  ☁️  AWS MANAGED SERVICES                                                                        ║
    ╠═══════════════════════╦══════════════════════╦═════════════════════╦════════════════════════════╣
    ║  🪣 S3 BUCKETS         ║  📊 DYNAMODB TABLES  ║  🔔 SNS             ║  🔐 SECRETS MANAGER        ║
    ║                       ║                      ║                     ║                            ║
    ║  cloudguard-reports   ║  CloudGuard-Findings  ║  CloudGuard-Alerts  ║  rds_secrets-xGeABJ        ║
    ║  -282627753593        ║  PK: finding_id       ║  ARN: us-east-1:    ║  (RDS credentials)         ║
    ║  (scan PDF reports)   ║  SK: timestamp        ║  282627753593:      ║                            ║
    ║                       ║  PITR: ✅ 35 days    ║  CloudGuard-Alerts  ║                            ║
    ║  cloudguard-flowlogs  ║                      ║                     ║                            ║
    ║  -282627753593        ║  CloudGuard-          ║                     ║                            ║
    ║  (VPC flow logs)      ║  ResourceInventory    ║                     ║                            ║
    ║                       ║  PK: resource_id      ║                     ║                            ║
    ║  aws-cloudtrail-logs  ║  SK: resource_type    ║                     ║                            ║
    ║  -282627753593        ║                      ║                     ║                            ║
    ║  (audit trail logs)   ║  CloudGuard-Scores    ║                     ║                            ║
    ║                       ║  PK: score_date       ║                     ║                            ║
    ║  s3-website-hosting   ║                      ║                     ║                            ║
    ║  -282627753593        ║  All tables:          ║                     ║                            ║
    ║  (CloudFront origin)  ║  PAY_PER_REQUEST      ║                     ║                            ║
    ╠═══════════════════════╩══════════════════════╩═════════════════════╩════════════════════════════╣
    ║  📋 CLOUDWATCH LOGS                                                                              ║
    ║  /aws/lambda/CloudGuard-API              (6.1 KB stored)                                         ║
    ║  /aws/lambda/CloudGuard-Resource-Scanner (7.5 KB stored  |  1 Metric Filter)                    ║
    ╚══════════════════════════════════════════════════════════════════════════════════════════════════╝
    

## 🔄 Scanner Execution Flow

    ║  🔄 SCANNER EXECUTION FLOW                                                                       ║
    ╠══════════════════════════════════════════════════════════════════════════════════════════════════╣
    ║                                                                                                  ║
    ║   POST /scan (API Gateway)  ──or──  EventBridge Schedule                                        ║
    ║              │                                                                                   ║
    ║              ▼                                                                                   ║
    ║   ┌──────────────────────────────────────────────────────────────────────────────────────────┐  ║
    ║   │                         λ  CloudGuard-Resource-Scanner                                   │  ║
    ║   │                         Python 3.14  |  512 MB  |  120s  |  Private Subnet               │  ║
    ║   └───┬──────────┬──────────┬──────────┬──────────┬──────────┬──────────┬────────────────────┘  ║
    ║       │          │          │          │          │          │          │                        ║
    ║  [1] Fetch  [2] Scan    [3] Scan   [4] Scan   [5] Scan   [6] Scan   [7] Scan                   ║
    ║  Secrets    IAM/STS     EC2        S3         RDS        CloudTrail  CloudWatch                 ║
    ║  Manager    (keys,      (EBS,      (lifecycle (public    (events,    (metrics)                  ║
    ║  (RDS       roles,      EIPs,      public     access,    trails)                                ║
    ║  creds)     policies)   SGs,       access)    encrypt)                                          ║
    ║       │          │          │          │          │          │          │                        ║
    ║       └──────────┴──────────┴────┬─────┴──────────┘          └──────────┘                      ║
    ║                                  │                                                              ║
    ║                    ┌─────────────┼──────────────────────────────────┐                          ║
    ║                    │             │                                  │                          ║
    ║                    ▼             ▼                                  ▼                          ║
    ║          ┌──────────────┐  ┌──────────────┐               ┌──────────────────┐                ║
    ║          │  📊 DynamoDB  │  │  🗄️  RDS      │               │  📈 CloudWatch   │                ║
    ║          │              │  │  PostgreSQL  │               │  Metrics         │                ║
    ║          │  Findings    │  │  Port: 5432  │               │                  │                ║
    ║          │  Inventory   │  │  cloudguard  │               │  Security: 65/100│                ║
    ║          │  Scores      │  │  _audit DB   │               │  Optim:   68/100 │                ║
    ║          └──────┬───────┘  └──────────────┘               │  Findings: 5     │                ║
    ║                 │                                          └──────────────────┘                ║
    ║          ┌──────┴───────┐                                                                      ║
    ║          │              │                                                                      ║
    ║          ▼              ▼                                                                      ║
    ║  ┌──────────────┐  ┌──────────────┐                                                           ║
    ║  │  🪣 S3        │  │  🔔 SNS      │                                                           ║
    ║  │  cloudguard  │  │  CloudGuard  │                                                           ║
    ║  │  -reports    │  │  -Alerts     │                                                           ║
    ║  │  (PDF report)│  │  (email/http)│                                                           ║
    ║  └──────────────┘  └──────────────┘                                                           ║
    ║                                                                                                  ║
    ║  ✅ LAST SCAN:  Findings: 5  |  Security: 65/100  |  Optimization: 68/100  |  Saving: $20/mo   ║
    ╚══════════════════════════════════════════════════════════════════════════════════════════════════╝
    

## 🔒 Security Groups

    ║  🔒 SECURITY GROUPS                                                                              ║
    ╠══════════════════════════════════════════════════════════════════════════════════════════════════╣
    ║                                                                                                  ║
    ║  ┌────────────────────────────────────────────────────────────────────────────────────────────┐ ║
    ║  │  sg-063939ad3450fba50  ·  cloudguard-lambda-sg  ·  Lambda Security Group                   │ ║
    ║  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │ ║
    ║  │  │  INBOUND   TCP 443  ←  sg-064b45dbda60872d0  (endpoint sg)                          │   │ ║
    ║  │  │  OUTBOUND  ALL      →  0.0.0.0/0                                                    │   │ ║
    ║  │  └─────────────────────────────────────────────────────────────────────────────────────┘   │ ║
    ║  └────────────────────────────────────────────────────────────────────────────────────────────┘ ║
    ║                                                                                                  ║
    ║  ┌────────────────────────────────────────────────────────────────────────────────────────────┐ ║
    ║  │  sg-064b45dbda60872d0  ·  endpoint  ·  VPC Endpoint Security Group                         │ ║
    ║  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │ ║
    ║  │  │  INBOUND   TCP 443  ←  sg-063939ad3450fba50  (lambda sg)                            │   │ ║
    ║  │  │  OUTBOUND  ALL      →  0.0.0.0/0                                                    │   │ ║
    ║  │  └─────────────────────────────────────────────────────────────────────────────────────┘   │ ║
    ║  └────────────────────────────────────────────────────────────────────────────────────────────┘ ║
    ║                                                                                                  ║
    ║  ┌────────────────────────────────────────────────────────────────────────────────────────────┐ ║
    ║  │  sg-054a310af826d3251  ·  cloudguard-rds-sg  ·  RDS Security Group                         │ ║
    ║  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │ ║
    ║  │  │  INBOUND   TCP 5432  ←  sg-063939ad3450fba50  (lambda sg)  ✅                       │   │ ║
    ║  │  │  INBOUND   TCP 5432  ←  152.58.31.166/32  (admin IP)                                │   │ ║
    ║  │  │  OUTBOUND  ALL       →  0.0.0.0/0                                                   │   │ ║
    ║  │  └─────────────────────────────────────────────────────────────────────────────────────┘   │ ║
    ║  └────────────────────────────────────────────────────────────────────────────────────────────┘ ║
    ╚══════════════════════════════════════════════════════════════════════════════════════════════════╝
    

## 👤 IAM Roles

    ║  👤 IAM ROLES                                                                                    ║
    ╠═════════════════════════════════════════════╦════════════════════════════════════════════════════╣
    ║  CloudGuard-Lambda-Scanner-Role             ║  CloudGuard-Lambda-API-Role                       ║
    ║                                             ║                                                   ║
    ║  Managed Policies:                          ║  Permissions:                                     ║
    ║  ├─ AWSLambdaBasicExecutionRole             ║  ├─ dynamodb:Scan                                 ║
    ║  ├─ AWSLambdaVPCAccessExecutionRole         ║  ├─ dynamodb:GetItem                              ║
    ║  ├─ AmazonEC2ReadOnlyAccess                 ║  ├─ dynamodb:Query                                ║
    ║  ├─ AmazonDynamoDBFullAccess                ║  │   on Findings + Scores tables                  ║
    ║  ├─ AmazonS3FullAccess                      ║  ├─ lambda:InvokeFunction                         ║
    ║  ├─ AmazonRDSFullAccess                     ║  │   on CloudGuard-Resource-Scanner               ║
    ║  ├─ IAMReadOnlyAccess                       ║  └─ logs:CreateLogGroup                           ║
    ║  ├─ CloudWatchReadOnlyAccess                ║      logs:CreateLogStream                         ║
    ║  └─ AWSCloudTrail_ReadOnlyAccess            ║      logs:PutLogEvents                            ║
    ║                                             ║                                                   ║
    ║  Inline Policies:                           ║                                                   ║
    ║  ├─ rdspolicy                               ║                                                   ║
    ║  │   rds-data:ExecuteStatement              ║                                                   ║
    ║  │   secretsmanager:GetSecretValue          ║                                                   ║
    ║  ├─ s3_policy                               ║                                                   ║
    ║  │   s3:PutObject / GetObject               ║                                                   ║
    ║  │   on cloudguard-reports bucket           ║                                                   ║
    ║  ├─ IAM_audit                               ║                                                   ║
    ║  │   iam:Get* / iam:List*                   ║                                                   ║
    ║  │   iam:GenerateCredentialReport           ║                                                   ║
    ║  ├─ Lambda-VPC-Execution-Policy             ║                                                   ║
    ║  │   ec2:CreateNetworkInterface             ║                                                   ║
    ║  │   ec2:DescribeNetworkInterfaces          ║                                                   ║
    ║  │   ec2:DeleteNetworkInterface             ║                                                   ║
    ║  └─ cloudwatch-putmetric-policy ✅ FIXED    ║                                                   ║
    ║      cloudwatch:PutMetricData               ║                                                   ║
    ╚═════════════════════════════════════════════╩════════════════════════════════════════════════════╝
    

## 🐛 Troubleshooting Log

    ║  🐛 TROUBLESHOOTING LOG — Issues Resolved in This Session                                        ║
    ╠════╦══════════════════════════════╦═══════════════════════════════════════════╦══════════════════╣
    ║ #  ║  Issue                       ║  Root Cause                               ║  Status          ║
    ╠════╬══════════════════════════════╬═══════════════════════════════════════════╬══════════════════╣
    ║ 1  ║  IAM Connect Timeout         ║  No STS VPC endpoint in private subnet    ║  ✅ Fixed        ║
    ╠════╬══════════════════════════════╬═══════════════════════════════════════════╬══════════════════╣
    ║ 2  ║  Lambda Timeout 300s         ║  Secrets Manager endpoint had wrong SG    ║  ✅ Fixed        ║
    ║    ║                              ║  (sg-063939ad3450fba50 instead of         ║                  ║
    ║    ║                              ║   sg-064b45dbda60872d0)                   ║                  ║
    ╠════╬══════════════════════════════╬═══════════════════════════════════════════╬══════════════════╣
    ║ 3  ║  AccessDenied                ║  cloudwatch:PutMetricData missing from    ║  ✅ Fixed        ║
    ║    ║  PutMetricData               ║  IAM role — only ReadOnly was attached    ║                  ║
    ╠════╬══════════════════════════════╬═══════════════════════════════════════════╬══════════════════╣
    ║ 4  ║  CloudFront 301              ║  HTTP used instead of HTTPS               ║  ✅ Fixed        ║
    ║    ║  Moved Permanently           ║  (redirect-to-https is correct behavior)  ║                  ║
    ╠════╬══════════════════════════════╬═══════════════════════════════════════════╬══════════════════╣
    ║ 5  ║  CloudFront 403              ║  Origin = Website endpoint (unsigned)     ║  ✅ Fixed        ║
    ║    ║  AccessDenied                ║  Bucket policy = OAC signed only          ║                  ║
    ║    ║                              ║  Fix: Switch to REST endpoint + OAC       ║                  ║
    ╚════╩══════════════════════════════╩═══════════════════════════════════════════╩══════════════════╝

## 🛠️ Technology Stack

- **AWS:** VPC, Lambda, API Gateway, CloudFront, S3, RDS PostgreSQL, DynamoDB
- **Security:** IAM, Security Groups, VPC Endpoints, Secrets Manager, CloudTrail
- **Monitoring:** CloudWatch Logs and Metrics
- **Messaging:** SNS
- **Automation:** EventBridge
- **Runtime:** Python 3.14

## 📌 Key Capabilities

- Automated AWS resource and security scanning
- IAM, EC2, S3, RDS, CloudTrail, and CloudWatch checks
- Private-subnet execution using VPC endpoints
- Findings, resource inventory, and security score persistence
- Automated PDF report storage in Amazon S3
- API-driven and scheduled scan execution
- CloudWatch security metrics and SNS notifications
- Multi-AZ scanner deployment

