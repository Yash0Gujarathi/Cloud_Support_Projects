# Cloud Audit and Insigt - Command Center
## Collect and Analysis of Logs
https://github.com/user-attachments/assets/bed5933a-416c-41af-aeaf-907d4a1f5bd2

## Architecture
╔══════════════════════════════════════════════════════════════════════════════════════════════╗
║                           AWS ACCOUNT: 282627753593 │ REGION: us-east-1                     ║
║                              CloudGuard Security Scanner Architecture                        ║
╚══════════════════════════════════════════════════════════════════════════════════════════════╝

                                         INTERNET
                                             │
                              ┌──────────────┴──────────────┐
                              │                             │
                    ┌─────────▼──────────┐      ┌──────────▼──────────┐
                    │    CLOUDFRONT      │      │   API GATEWAY        │
                    │  E2WTPUAINAOTNP    │      │  (CloudGuard-API)    │
                    │  d392runfkyxnze    │      │                      │
                    │  .cloudfront.net   │      └──────────┬──────────┘
                    │  OAC: E93V863CQWE5B│                 │
                    └─────────┬──────────┘                 │
                              │ HTTPS (OAC signed)         │ HTTPS
                              │                            │
╔═════════════════════════════╪════════════════════════════╪══════════════════════════════════╗
║  project-vpc  │  vpc-0261a5e1b2611731f  │  10.0.0.0/16  │                                  ║
║               │                                          │                                  ║
║  ┌────────────┴──────────────────────────────────────────┴──────────────────────────────┐  ║
║  │                    ╔══════════════════════════════════╗                               │  ║
║  │  PUBLIC SUBNETS    ║    Internet Gateway               ║                               │  ║
║  │                    ║    project-igw                    ║                               │  ║
║  │                    ║    igw-0f62206a9c8223844          ║                               │  ║
║  │                    ╚══════════════════════════════════╝                               │  ║
║  │  ┌─────────────────────────────┐  ┌──────────────────────────────┐                   │  ║
║  │  │ project-subnet-public1      │  │ project-subnet-public2       │                   │  ║
║  │  │ subnet-00c429d1e383ebf80    │  │ subnet-0786858262c5b9afa     │                   │  ║
║  │  │ 10.0.0.0/20 │ us-east-1a   │  │ 10.0.16.0/20 │ us-east-1b   │                   │  ║
║  │  │                             │  │                              │                   │  ║
║  │  │  [RDS subnet member]        │  │  [RDS subnet member]         │                   │  ║
║  │  └─────────────────────────────┘  └──────────────────────────────┘                   │  ║
║  │                                                                                       │  ║
║  │  PRIVATE SUBNETS                                                                      │  ║
║  │  ┌─────────────────────────────────────────┐  ┌────────────────────────────────────┐ │  ║
║  │  │ project-subnet-private1                 │  │ project-subnet-private2            │ │  ║
║  │  │ subnet-03e1a4eb50c7bc403                │  │ subnet-0df113a2995a0a1ca           │ │  ║
║  │  │ 10.0.128.0/20 │ us-east-1a             │  │ 10.0.144.0/20 │ us-east-1b         │ │  ║
║  │  │                                         │  │                                    │ │  ║
║  │  │  ┌──────────────────────────────────┐   │  │  ┌──────────────────────────────┐  │ │  ║
║  │  │  │  Lambda: CloudGuard-Resource-    │   │  │  │  Lambda: CloudGuard-Resource-│  │ │  ║
║  │  │  │  Scanner                         │   │  │  │  Scanner (Multi-AZ)          │  │ │  ║
║  │  │  │  Runtime: Python 3.14            │   │  │  │  sg-063939ad3450fba50        │  │ │  ║
║  │  │  │  Memory: 512MB │ Timeout: 120s   │   │  │  └──────────────────────────────┘  │ │  ║
║  │  │  │  sg-063939ad3450fba50            │   │  │                                    │ │  ║
║  │  │  └──────────────────────────────────┘   │  │  ┌──────────────────────────────┐  │ │  ║
║  │  │                                         │  │  │  RDS: cloudguard-audit-db    │  │ │  ║
║  │  │  ┌──────────────────────────────────┐   │  │  │  PostgreSQL 18.3             │  │ │  ║
║  │  │  │  RDS: cloudguard-audit-db        │   │  │  │  db.t3.micro │ 20GB gp2     │  │ │  ║
║  │  │  │  PostgreSQL 18.3                 │   │  │  │  sg-054a310af826d3251        │  │ │  ║
║  │  │  │  db.t3.micro │ 20GB gp2 (enc)   │   │  │  └──────────────────────────────┘  │ │  ║
║  │  │  │  Port: 5432                      │   │  │                                    │ │  ║
║  │  │  │  sg-054a310af826d3251            │   │  └────────────────────────────────────┘ │  ║
║  │  │  └──────────────────────────────────┘   │                                         │  ║
║  │  └─────────────────────────────────────────┘                                         │  ║
║  │                                                                                       │  ║
║  │  VPC INTERFACE ENDPOINTS (sg-064b45dbda60872d0)                                      │  ║
║  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │  ║
║  │  │   STS    │ │   SNS    │ │Secrets   │ │   RDS    │ │   IAM    │ │   EC2    │     │  ║
║  │  │Interface │ │Interface │ │Manager   │ │Interface │ │Interface │ │Interface │     │  ║
║  │  │PrivDNS✅ │ │PrivDNS✅ │ │Interface │ │PrivDNS✅ │ │PrivDNS✅ │ │PrivDNS✅ │     │  ║
║  │  └──────────┘ └──────────┘ │PrivDNS✅ │ └──────────┘ └──────────┘ └──────────┘     │  ║
║  │  ┌──────────┐ ┌──────────┐ └──────────┘ ┌──────────┐ ┌──────────┐                  │  ║
║  │  │CW Logs   │ │Monitoring│              │CloudTrail│ │Logs      │                  │  ║
║  │  │Interface │ │Interface │              │Interface │ │Insights  │                  │  ║
║  │  │PrivDNS✅ │ │PrivDNS✅ │              │PrivDNS✅ │ │PrivDNS✅ │                  │  ║
║  │  └──────────┘ └──────────┘              └──────────┘ └──────────┘                  │  ║
║  │                                                                                       │  ║
║  │  VPC GATEWAY ENDPOINTS                                                                │  ║
║  │  ┌──────────────────────────┐  ┌──────────────────────────┐                          │  ║
║  │  │  S3 Gateway Endpoint     │  │  DynamoDB Gateway Endpoint│                          │  ║
║  │  │  vpce-0a6c3c525436ac2d7  │  │  vpce-02fd5d0d151cbd89f  │                          │  ║
║  │  └──────────────────────────┘  └──────────────────────────┘                          │  ║
║  └───────────────────────────────────────────────────────────────────────────────────────┘  ║
╚══════════════════════════════════════════════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════════════════════════════════════════════╗
║                              AWS MANAGED SERVICES (Global / Regional)                        ║
╠══════════════════════════════════════════════════════════════════════════════════════════════╣
║                                                                                              ║
║   STORAGE                    COMPUTE                  MESSAGING          MONITORING          ║
║  ┌────────────────────┐    ┌──────────────────────┐  ┌───────────────┐  ┌────────────────┐  ║
║  │  S3 Buckets        │    │ Lambda: CloudGuard-  │  │  SNS Topic    │  │  CloudWatch    │  ║
║  │                    │    │ API                  │  │  CloudGuard-  │  │  Logs          │  ║
║  │ ├ cloudguard-      │    │ Python 3.14          │  │  Alerts       │  │  /aws/lambda/  │  ║
║  │ │  reports-        │    │ Memory: 128MB        │  │               │  │  CloudGuard-   │  ║
║  │ │  282627753593    │◄───│ Timeout: 3s          │  │  Subscribers: │  │  Resource-     │  ║
║  │ │                  │    │ Role: CloudGuard-    │  │  Email/HTTP   │  │  Scanner       │  ║
║  │ ├ cloudguard-      │    │ Lambda-API-Role      │  └───────┬───────┘  └────────────────┘  ║
║  │ │  flowlogs-       │    └──────────────────────┘          │                              ║
║  │ │  282627753593    │                                       │ Alerts                       ║
║  │ │                  │    DATABASE                           ▼                              ║
║  │ ├ aws-cloudtrail-  │   ┌──────────────────────┐  ┌───────────────┐                       ║
║  │ │  logs-           │   │  DynamoDB Tables     │  │  Secrets Mgr  │                       ║
║  │ │  282627753593    │   │                      │  │               │                       ║
║  │ │                  │   │ ├ CloudGuard-        │  │  rds_secrets  │                       ║
║  │ └ s3-website-      │   │ │  Findings          │  │  -xGeABJ      │                       ║
║  │   hosting-         │   │ ├ CloudGuard-        │  │  (RDS creds)  │                       ║
║  │   282627753593     │   │ │  ResourceInventory │  └───────────────┘                       ║
║  └────────────────────┘   │ └ CloudGuard-Scores  │                                          ║
║         ▲                 └──────────────────────┘                                          ║
║         │ OAC                                                                                ║
║         │                                                                                    ║
╚═════════╪══════════════════════════════════════════════════════════════════════════════════╝
          │
╔═════════╪══════════════════════════════════════════════════════════════════════════════════╗
║         │           DATA FLOW — CloudGuard Scanner Execution                               ║
╠═════════╪══════════════════════════════════════════════════════════════════════════════════╣
║                                                                                             ║
║  EventBridge/Trigger                                                                        ║
║       │                                                                                     ║
║       ▼                                                                                     ║
║  Lambda: CloudGuard-Resource-Scanner                                                        ║
║       │                                                                                     ║
║       ├──[1]── Secrets Manager ──► Fetch RDS credentials (rds_secrets-xGeABJ)              ║
║       │                                                                                     ║
║       ├──[2]── IAM endpoint ──────► Scan IAM keys, roles, policies                         ║
║       │                                                                                     ║
║       ├──[3]── EC2 endpoint ──────► Scan EBS, EIPs, Snapshots, Security Groups             ║
║       │                                                                                     ║
║       ├──[4]── S3 Gateway ────────► Scan bucket lifecycle, public access                   ║
║       │                                                                                     ║
║       ├──[5]── RDS endpoint ──────► Scan public access, encryption                         ║
║       │                                                                                     ║
║       ├──[6]── DynamoDB Gateway ──► Write findings to CloudGuard-Findings                  ║
║       │                            Write inventory to CloudGuard-ResourceInventory          ║
║       │                            Write scores to CloudGuard-Scores                        ║
║       │                                                                                     ║
║       ├──[7]── RDS PostgreSQL ────► Write audit report to cloudguard_audit DB              ║
║       │        (Port 5432)                                                                  ║
║       │                                                                                     ║
║       ├──[8]── S3 Gateway ────────► Upload report to cloudguard-reports-282627753593       ║
║       │                                                                                     ║
║       ├──[9]── SNS Topic ─────────► Publish alerts to CloudGuard-Alerts                    ║
║       │                                                                                     ║
║       └──[10]─ CloudWatch Metrics ► Publish Security Score & Findings count                ║
║                (PutMetricData)                                                              ║
║                                                                                             ║
║  SCAN RESULTS:  Findings: 5 │ Security: 65/100 │ Optimization: 68/100 │ Saving: $20/month  ║
╚═════════════════════════════════════════════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════════════════════════════════════════════╗
║                              SECURITY GROUPS REFERENCE                                       ║
╠══════════════════════════════════════════════════════════════════════════════════════════════╣
║                                                                                              ║
║  sg-063939ad3450fba50 (cloudguard-lambda-sg)                                                 ║
║  ├── Inbound:  TCP 443 ← sg-064b45dbda60872d0 (endpoint sg)                                 ║
║  └── Outbound: All traffic → 0.0.0.0/0                                                      ║
║                                                                                              ║
║  sg-064b45dbda60872d0 (endpoint)                                                             ║
║  ├── Inbound:  TCP 443 ← sg-063939ad3450fba50 (lambda sg)                                   ║
║  └── Outbound: All traffic → 0.0.0.0/0                                                      ║
║                                                                                              ║
║  sg-054a310af826d3251 (cloudguard-rds-sg)                                                    ║
║  ├── Inbound:  TCP 5432 ← sg-063939ad3450fba50 (lambda sg) ✅                               ║
║  ├── Inbound:  TCP 5432 ← 152.58.31.166/32                                                  ║
║  └── Outbound: All traffic → 0.0.0.0/0                                                      ║
║                                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════════════════════════════════════════════╗
║                              IAM ROLES REFERENCE                                             ║
╠══════════════════════════════════════════════════════════════════════════════════════════════╣
║                                                                                              ║
║  CloudGuard-Lambda-Scanner-Role                                                              ║
║  ├── AWSLambdaBasicExecutionRole        ├── AmazonS3FullAccess                              ║
║  ├── AWSLambdaVPCAccessExecutionRole    ├── AmazonRDSFullAccess                             ║
║  ├── AmazonEC2ReadOnlyAccess            ├── IAMReadOnlyAccess                               ║
║  ├── AmazonDynamoDBFullAccess           ├── CloudWatchReadOnlyAccess                        ║
║  ├── AWSCloudTrail_ReadOnlyAccess       ├── cloudwatch-putmetric-policy (inline) ✅         ║
║  ├── rdspolicy (inline)                 └── Lambda-VPC-Execution-Policy (inline)            ║
║  └── s3_policy (inline)                                                                      ║
║                                                                                              ║
║  CloudGuard-Lambda-API-Role                                                                  ║
║  └── (API Lambda execution role)                                                             ║
║                                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════════════════════╝
