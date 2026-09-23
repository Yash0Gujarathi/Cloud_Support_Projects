# Hell Guard - 
## VPC (Production and Operational) security and monitoring Dashboard 

https://github.com/user-attachments/assets/a20586e4-f2d2-4d96-a8fe-9a25b2157387

# CloudOps Guardian AI
## AI-Driven Self-Healing Cloud Operations, Vulnerability Management & Security Hardening Platform

**Document type:** Project Design + Build Blueprint  
**Architecture style:** Native AWS Console / AWS-managed services  
**Primary focus:** CloudOps + Cloud Security + Event-Driven Automation + AI-assisted remediation  
**GuardDuty:** Excluded from the initial build; can be added later  
**Target environment:** AWS Free Tier / Free Trial optimized, with strict cost controls  
**Recommended implementation region:** `us-east-1` unless a required service/model is unavailable in the chosen Region

---

<!-- PAGE 1 -->

# Page 1 — Executive Summary

## 1.1 Project Vision

CloudOps Guardian AI is an event-driven AWS operations platform that continuously observes private cloud workloads, detects operational failures and software vulnerabilities, analyzes incidents with Amazon Bedrock, recommends security hardening, and executes only controlled remediation actions through AWS Systems Manager.

The platform is intentionally designed around a key enterprise principle:

> **AI recommends and reasons; deterministic AWS controls authorize and execute.**

Bedrock must not be allowed to freely invent and execute shell commands.

The platform therefore separates:

```text
Detection
   ↓
Event Routing
   ↓
AI Analysis
   ↓
Structured Recommendation
   ↓
Safety / Policy Validation
   ↓
Approval Decision
   ↓
SSM Execution
   ↓
Verification
   ↓
Audit + Notification
```

## 1.2 Business Problem

A production AWS environment creates thousands of operational signals:

- EC2 CPU spikes
- Application crashes
- Nginx 5xx errors
- Disk exhaustion
- Vulnerable packages
- Misconfigured Security Groups
- Unauthorized API activity
- Failed deployments
- Configuration drift
- Expired or missing patches

Traditional monitoring produces alerts but still leaves engineers to:

1. Read the alert.
2. Investigate logs.
3. Identify the affected resource.
4. Decide what the problem means.
5. Determine the correct fix.
6. Log into the server.
7. Apply the fix.
8. Verify recovery.
9. Document the incident.

CloudOps Guardian compresses this workflow into an event-driven operational loop.

## 1.3 Project Objectives

The platform must:

- Keep application EC2 instances private.
- Remove routine SSH administration.
- Use Systems Manager as the execution and administration plane.
- Detect vulnerabilities using Amazon Inspector.
- Detect application/infrastructure problems using CloudWatch.
- Audit AWS API activity with CloudTrail.
- Route events using EventBridge.
- Analyze incidents with Amazon Bedrock.
- Produce structured remediation recommendations.
- Enforce an allowlist of safe actions.
- Require approval for high-risk actions.
- Execute approved actions with SSM Run Command.
- Verify remediation.
- Store evidence and reports.
- Provide an operational dashboard.
- Produce AI-generated security-hardening recommendations.
- Minimize AWS cost.

---

<!-- PAGE 2 -->

# Page 2 — Architecture Principles

## 2.1 Core Principles

### Principle 1 — Private by Default

Private EC2 instances should not have:

- Public IP addresses
- Direct inbound SSH
- Unrestricted internet access

Administrative access occurs through Systems Manager.

### Principle 2 — Least Privilege

Every automation component gets only the permissions it needs.

Example:

```text
CloudOps AI Lambda
│
├── bedrock:InvokeModel
├── ssm:SendCommand
├── cloudwatch:GetMetricData
├── logs:GetLogEvents
└── inspector2:GetFindings
```

Do not give the Lambda:

```text
AdministratorAccess
```

### Principle 3 — Event-Driven Operations

Avoid polling whenever an AWS event can trigger an operation.

```text
Finding
  ↓
EventBridge
  ↓
Lambda
```

### Principle 4 — AI Is Not the Root of Trust

Bedrock output is untrusted input.

The safety layer validates:

- action
- target
- confidence
- severity
- allowed command
- approval requirement
- resource ownership
- maintenance window
- rate limit

### Principle 5 — Every Automated Action Is Auditable

Record:

```text
Event ID
Incident ID
Instance ID
Finding
AI recommendation
AI confidence
Policy decision
SSM command ID
Execution result
Timestamp
Operator/automation identity
```

### Principle 6 — Fail Closed

If:

- AI output is malformed
- confidence is too low
- target is unknown
- action is not allowlisted
- resource is not tagged correctly
- permissions are insufficient

then:

```text
DO NOT EXECUTE
```

Instead:

```text
Recommendation → Dashboard → Human Review
```

---

## 2.2 Why GuardDuty Is Excluded

GuardDuty is deliberately **not part of Phase 1**.

The initial security detection stack is:

```text
IAM
CloudTrail
Inspector
CloudWatch
VPC
Security Groups
NACL
SSM
```

GuardDuty can later become an additional event source:

```text
GuardDuty Finding
       ↓
EventBridge
       ↓
AI Engine
       ↓
Incident Response
```

This keeps the initial architecture smaller and easier to operate.

---

<!-- PAGE 3 -->

# Page 3 — High-Level Architecture

## 3.1 Logical Architecture

```text
                         AWS ORGANIZATION
                                │
                 ┌──────────────┴──────────────┐
                 │                             │
          Operations Account             Production Account
                 │                             │
          ┌──────▼───────┐             ┌───────▼────────┐
          │ Operations   │             │ Production VPC │
          │ VPC          │◄───────────►│                │
          │              │ VPC Peering │ Public         │
          │ EventBridge  │              │ Private App    │
          │ Lambda       │              │ Isolated Data  │
          │ Bedrock      │              │                │
          │ CloudWatch   │              │ EC2 / RDS      │
          │ SSM         │              └────────────────┘
          └──────────────┘
                 │
                 ▼
       Centralized Logging / Evidence
                 │
                 ├── CloudWatch Logs
                 ├── S3
                 └── CloudTrail
```

## 3.2 Main Data Flow

```text
Private EC2
   │
   ├── CloudWatch Agent
   │        └── Logs + Metrics
   │
   ├── SSM Agent
   │        └── Management channel
   │
   └── Inspector
            └── Vulnerability findings

CloudWatch / Inspector / CloudTrail
             │
             ▼
       EventBridge
             │
             ▼
       Lambda AI Engine
             │
       ┌─────┴─────┐
       ▼           ▼
   Context      Bedrock
   Collection    Analysis
       │           │
       └─────┬─────┘
             ▼
      Structured JSON
             │
             ▼
      Safety Validator
             │
      ┌──────┴──────┐
      ▼             ▼
   Auto Fix     Human Approval
      │             │
      └──────┬──────┘
             ▼
          SSM
             │
             ▼
        Private EC2
             │
             ▼
         Verification
             │
             ▼
       CloudWatch/S3/SNS
```

## 3.3 Architecture Image

The project architecture is represented in:

- `architecture_overview.png`
- `architecture_ai_workflow.png`
- `architecture_private_network.png`

The generated AWS-style architecture image included with this document can be used as the visual reference while implementing the project.

---

<!-- PAGE 4 -->

# Page 4 — Multi-Account / Multi-VPC Strategy

## 4.1 Recommended Logical Accounts

For a serious portfolio implementation:

```text
AWS Organization
│
├── Management Account
│
├── Operations Account
│   └── Operations VPC
│
└── Production Account
    └── Production VPC
```

For a Free Tier student account where multiple accounts are impractical:

```text
Single AWS Account
│
├── Operations VPC
└── Production VPC
```

Start with the second design and document how it would scale to Organizations.

## 4.2 Operations VPC

Purpose:

- Automation
- Event routing
- AI orchestration
- Administrative tooling
- Central dashboards

Components:

```text
Operations VPC
│
├── EventBridge
├── Lambda
├── Bedrock API integration
├── CloudWatch
├── SNS
└── S3
```

AWS managed services do not all live "inside" the VPC. The VPC contains the private network and endpoints where needed.

## 4.3 Production VPC

Purpose:

- Application workloads
- Database workloads
- Private compute

```text
Production VPC
│
├── Public Subnets
│   └── ALB
│
├── Private App Subnets
│   └── EC2
│
└── Isolated Data Subnets
    └── RDS
```

## 4.4 VPC Peering

For the student build:

```text
Operations VPC
       │
       │ VPC Peering
       │
Production VPC
```

Transit Gateway is intentionally excluded because it adds unnecessary cost for this scale.

---

<!-- PAGE 5 -->

# Page 5 — Three-Tier Production Network

## 5.1 CIDR Plan

Example:

```text
Production VPC
10.20.0.0/16

AZ-A
├── Public-A      10.20.1.0/24
├── App-A         10.20.11.0/24
└── Data-A        10.20.21.0/24

AZ-B
├── Public-B      10.20.2.0/24
├── App-B         10.20.12.0/24
└── Data-B        10.20.22.0/24
```

Operations VPC:

```text
10.10.0.0/16
```

Do not overlap VPC CIDRs if VPC Peering is planned.

## 5.2 Public Tier

Contains:

- Application Load Balancer
- NAT Gateway only if the budget permits

The public tier should not contain application servers.

## 5.3 Private App Tier

Contains:

- EC2
- Optional ECS workloads
- SSM Agent
- CloudWatch Agent

Characteristics:

```text
No public IP
No inbound SSH
No direct inbound internet
```

## 5.4 Isolated Data Tier

Contains:

- RDS

Characteristics:

```text
No Internet Gateway route
No NAT route
Only application security group access
```

## 5.5 Desired Traffic

```text
Internet
   │
   ▼
ALB
   │
   ▼
Private EC2
   │
   ▼
RDS
```

Not:

```text
Internet → EC2
Internet → RDS
EC2 → Internet unrestricted
```

---

<!-- PAGE 6 -->

# Page 6 — Private Management Network

## 6.1 SSM Architecture

The EC2 instance uses the SSM Agent.

```text
PRIVATE EC2
│
│ HTTPS 443 outbound
│
├──────────────► SSM Interface Endpoint
│
├──────────────► SSMMessages Endpoint
│
└──────────────► EC2Messages Endpoint
```

AWS documentation states that Systems Manager can use interface VPC endpoints to keep traffic between managed nodes, Systems Manager, and EC2 on the AWS network. SSM Agent initiates the connection, so inbound firewall access to the instance is not required for Systems Manager.

For modern SSM Agent versions, `ssmmessages` is preferred when available. `ec2messages` remains relevant in Regions/architectures where it is supported.

## 6.2 Endpoint Set

For a strict private design, plan for:

```text
com.amazonaws.<region>.ssm
com.amazonaws.<region>.ssmmessages
com.amazonaws.<region>.ec2
com.amazonaws.<region>.logs
com.amazonaws.<region>.s3
```

Not every endpoint is required for every feature.

## 6.3 S3 Gateway Endpoint

S3 Gateway Endpoint is particularly useful because it can provide S3 connectivity without NAT.

```text
Private EC2
     │
     ▼
S3 Gateway Endpoint
     │
     ▼
S3
```

Gateway endpoints for S3 and DynamoDB have no additional endpoint charge.

## 6.4 Why This Matters

This produces:

```text
Internet
   X
   │
Private EC2
   │
   ├── SSM
   ├── CloudWatch Logs
   └── S3
        │
        ▼
AWS private networking
```

---

<!-- PAGE 7 -->

# Page 7 — Security Groups and NACL Architecture

## 7.1 Security Group Model

Security Groups are stateful.

Recommended groups:

```text
SG-ALB
SG-APP
SG-DB
SG-SSM-ENDPOINT
SG-VPC-ENDPOINT
```

## 7.2 ALB SG

Inbound:

```text
TCP 443 ← Internet
```

Optionally:

```text
TCP 80 ← Internet
```

Outbound:

```text
TCP 80/443 → SG-APP
```

## 7.3 APP SG

Inbound:

```text
TCP 80/443 ← SG-ALB
```

SSM-related endpoint communication:

```text
TCP 443 → SG-SSM-ENDPOINT
```

Outbound should be minimized according to actual requirements.

## 7.4 DB SG

Inbound:

```text
TCP 3306 ← SG-APP
```

or:

```text
TCP 5432 ← SG-APP
```

No internet access.

## 7.5 NACL Matrix

NACLs are stateless.

Conceptual matrix:

| Tier | Inbound | Outbound |
|---|---|---|
| Public | ALB listener traffic | ALB → App / required return |
| App | ALB + endpoint traffic | DB + endpoints + required return |
| Data | App → DB | Required return traffic |
| Operations | Required VPC peering flows | Required management flows |

When implementing custom NACLs, explicitly account for ephemeral return ports. Do not copy a generic ephemeral-port rule without testing the real flow.

---

<!-- PAGE 8 -->

# Page 8 — Identity and Access Management

## 8.1 IAM Architecture

```text
Human
 │
 ▼
IAM Identity Center
 │
 ├── CloudOps Operator
 ├── Security Reviewer
 └── ReadOnly Auditor
```

Automation:

```text
Lambda Execution Role
        │
        ├── Bedrock Invoke
        ├── SSM SendCommand
        ├── CloudWatch Read
        ├── Inspector Read
        └── S3 Write
```

## 8.2 EC2 Role

The EC2 instance profile should provide:

```text
SSM managed-node permissions
CloudWatch Agent permissions
S3 access only if required
```

Do not attach administrator permissions.

## 8.3 Lambda Role

The AI Lambda needs permissions to:

```text
bedrock:InvokeModel
ssm:SendCommand
ssm:GetCommandInvocation
logs:GetLogEvents
cloudwatch:GetMetricData
inspector2:GetFindings
sns:Publish
s3:PutObject
```

Scope resources where possible.

## 8.4 Separation of Duties

```text
AI
 │
 └── Recommendation

Policy Validator
 │
 └── Authorization decision

SSM
 │
 └── Execution

CloudTrail
 │
 └── Audit
```

No single component should have unrestricted authority.

---

<!-- PAGE 9 -->

# Page 9 — Observability Layer

## 9.1 CloudWatch Agent

Install the CloudWatch Agent on EC2.

Collect:

### Metrics

- CPU
- Memory
- Disk
- Disk I/O
- Network
- Processes

### Logs

Example:

```text
/var/log/messages
/var/log/syslog
/var/log/nginx/access.log
/var/log/nginx/error.log
/application/logs/*.log
```

## 9.2 Log Groups

Example:

```text
/cloudops/ec2/system
/cloudops/ec2/nginx
/cloudops/application
/cloudops/remediation
/cloudops/security
```

## 9.3 Metric Filters

Example:

```text
Pattern:
"502"

Metric:
Nginx5xxCount
```

Another:

```text
"upstream timed out"
```

Another:

```text
"worker process exited"
```

## 9.4 Alarms

Example:

```text
CPU > 85% for 5 minutes
```

or:

```text
Nginx5xxCount > 10 in 5 minutes
```

or:

```text
DiskUsedPercent > 90%
```

CloudWatch alarm state changes can be routed through EventBridge.

---

<!-- PAGE 10 -->

# Page 10 — CloudTrail Audit Architecture

## 10.1 Purpose

CloudTrail answers:

```text
Who?
What?
When?
From where?
Which AWS API?
Which resource?
```

## 10.2 Initial Configuration

Use one multi-Region trail for management events.

Prefer:

```text
Management Events
├── Read
└── Write
```

Keep data events disabled initially unless the project specifically needs them because data events can incur charges.

## 10.3 Audit Flow

```text
AWS API Call
     │
     ▼
CloudTrail
     │
     ├── S3
     │
     └── EventBridge
```

## 10.4 Automated Remediation Audit

```text
EventBridge
    ↓
Lambda
    ↓
SSM SendCommand
    ↓
CloudTrail
    ↓
S3
```

Store a separate remediation record:

```json
{
  "incident_id": "INC-001",
  "resource": "i-xxxxxxxx",
  "action": "RESTART_NGINX",
  "ai_confidence": 0.94,
  "policy_decision": "APPROVED",
  "ssm_command_id": "xxxxxxxx",
  "timestamp": "..."
}
```

---

<!-- PAGE 11 -->

# Page 11 — Vulnerability Management with Inspector

## 11.1 Role of Inspector

Amazon Inspector is the vulnerability-management layer.

It identifies software vulnerabilities and provides findings that can be used by automation.

Example:

```text
EC2
 │
 ▼
Inspector
 │
 ▼
CVE Finding
 │
├── Severity
├── Package
├── Installed version
├── Fixed version
└── Resource
```

## 11.2 Free Trial Strategy

Amazon Inspector provides a 15-day free trial for each scan type when activated. The trial does not apply to CIS scanning.

Therefore:

```text
Phase 1
Activate Inspector

Phase 2
Test findings

Phase 3
Build EventBridge rules

Phase 4
Test remediation

Phase 5
Review projected cost

Phase 6
Disable or continue intentionally
```

Do not leave a paid vulnerability scanner running accidentally after the trial.

## 11.3 Example Finding

```text
Severity: HIGH
Package: nginx
Installed: vulnerable version
Fixed: newer version
Instance: i-xxxxxxxx
```

This becomes:

```text
Inspector
    ↓
EventBridge
    ↓
AI Engine
```

## 11.4 Patch Manager

Systems Manager Patch Manager can be used to:

- define patch baselines
- scan instances
- apply patches
- schedule maintenance

For the project, separate:

```text
Detection
=
Inspector

Patch Execution
=
SSM Patch Manager
```

---

<!-- PAGE 12 -->

# Page 12 — EventBridge Event Router

## 12.1 Event Sources

Initial sources:

```text
CloudWatch Alarm
Inspector Finding
CloudTrail API Event
SSM Automation Event
AWS Health Event
```

## 12.2 EventBridge Rules

Example conceptual rules:

```text
RULE-EC2-HIGH-CPU
RULE-NGINX-CRASH
RULE-INSPECTOR-HIGH
RULE-INSPECTOR-CRITICAL
RULE-SECURITY-GROUP-CHANGE
RULE-IAM-POLICY-CHANGE
RULE-SSM-FAILURE
```

## 12.3 Routing

```text
                         EventBridge
                              │
       ┌──────────────────────┼─────────────────────┐
       │                      │                     │
       ▼                      ▼                     ▼
   CloudOps AI            SNS Alert             S3 Archive
      Lambda
       │
       ▼
    Bedrock
       │
       ▼
   Policy Engine
       │
       ▼
      SSM
```

## 12.4 Event Filtering

Do not send every event to Bedrock.

Filter first.

Example:

```text
Severity = HIGH/CRITICAL
AND
Resource type = EC2
AND
Finding state = ACTIVE
```

This reduces:

- AI cost
- Lambda invocations
- false positives
- operational noise

---

<!-- PAGE 13 -->

# Page 13 — AI Engine Design

## 13.1 Components

```text
AI Engine
│
├── Lambda
│
├── Context Collector
│
├── Amazon Bedrock
│
├── JSON Parser
│
├── Safety Validator
│
├── Action Allowlist
│
└── Incident Logger
```

## 13.2 Bedrock Responsibilities

### Diagnosis

"What caused this?"

### Recommendation

"What should be done?"

### Hardening

"How can the environment be made safer?"

### Explanation

"Why is this recommendation appropriate?"

## 13.3 Bedrock Must Not

Bedrock must not:

- directly execute shell commands
- create IAM administrators
- delete RDS
- delete S3
- disable CloudTrail
- disable Inspector
- disable security controls
- arbitrarily modify routes
- arbitrarily modify NACLs

## 13.4 Context Sent to Bedrock

Use only relevant context:

```text
Incident
Resource metadata
CloudWatch log excerpt
CloudWatch metrics
Inspector finding
OS/package information
Network exposure
Security Group summary
Recent related events
```

Never send secrets.

---

<!-- PAGE 14 -->

# Page 14 — AI Remediation Workflow

## 14.1 Complete Workflow

```text
1. Event occurs
        ↓
2. EventBridge receives event
        ↓
3. Lambda extracts resource ID
        ↓
4. Lambda collects context
        ↓
5. Context is normalized
        ↓
6. Bedrock analyzes
        ↓
7. Bedrock returns JSON
        ↓
8. JSON schema validation
        ↓
9. Action allowlist validation
        ↓
10. Risk classification
        ↓
11. Approval decision
        ↓
12. SSM Run Command
        ↓
13. Verify result
        ↓
14. Record evidence
        ↓
15. Notify
```

## 14.2 Example

Input:

```text
CPU: 96%
Nginx 502: 43
Nginx error:
upstream timed out
worker exited
```

AI:

```json
{
  "incident_type": "application_failure",
  "severity": "HIGH",
  "confidence": 0.94,
  "diagnosis": "Nginx worker instability",
  "recommended_action": "RESTART_NGINX",
  "requires_human_approval": false
}
```

Policy:

```text
RESTART_NGINX
Allowed?
YES

Confidence >= 0.85?
YES

Target is tagged cloudops-managed?
YES

Execute?
YES
```

SSM:

```text
sudo systemctl restart nginx
```

Verification:

```text
systemctl is-active nginx
HTTP health check
CloudWatch 5xx metric
```

---

<!-- PAGE 15 -->

# Page 15 — AI Security Hardening Advisor

## 15.1 Two AI Modes

### Mode 1 — Incident Remediation

```text
Something is broken
        ↓
Fix it
```

### Mode 2 — Security Advisor

```text
Nothing is necessarily broken
        ↓
Find ways to improve security
```

## 15.2 Example Hardening Input

```text
EC2:
private = false

Security Group:
22/tcp → 0.0.0.0/0

SSM:
enabled

Inspector:
2 high findings

CloudTrail:
enabled

EBS:
encrypted
```

## 15.3 AI Recommendation

```text
Risk: HIGH

1. Remove public SSH exposure.
2. Use Session Manager.
3. Restrict ingress to ALB/application sources.
4. Patch vulnerable packages.
5. Restrict outbound access.
6. Review unnecessary listening ports.
```

## 15.4 Recommendation Score

Use deterministic scoring around the AI output.

Example:

```text
Critical = 100
High     = 70
Medium   = 40
Low      = 20
```

Then add factors:

```text
Internet exposure
Exploitability
Asset criticality
Vulnerability severity
Confidence
Existing controls
```

The score is a project-defined operational score, not an AWS security score.

---

<!-- PAGE 16 -->

# Page 16 — Safety and Policy Engine

## 16.1 Why It Exists

AI can make mistakes.

Therefore:

```text
Bedrock
  ↓
Policy Engine
  ↓
SSM
```

not:

```text
Bedrock
  ↓
Shell
```

## 16.2 Low-Risk Allowlist

```text
RESTART_NGINX
RESTART_HTTPD
RESTART_DOCKER
CLEAR_CACHE
ROTATE_LOGS
```

## 16.3 Medium-Risk

```text
PATCH_PACKAGE
RESTART_EC2
CHANGE_APPLICATION_CONFIG
```

These should normally require:

- maintenance window
- tag validation
- rollback strategy

## 16.4 High-Risk

```text
DELETE_RESOURCE
MODIFY_IAM
MODIFY_NACL
MODIFY_ROUTE_TABLE
CHANGE_RDS
DISABLE_SECURITY_CONTROL
```

These require human approval.

## 16.5 Resource Tag Guardrail

Only remediate:

```text
Environment=Lab
CloudOpsManaged=true
```

Example:

```text
CloudOpsManaged=true
RemediationMode=Auto
```

A production resource without this tag should not be automatically modified.

---

<!-- PAGE 17 -->

# Page 17 — Systems Manager Execution Plane

## 17.1 Run Command

Run Command executes controlled commands without SSH.

Example:

```text
AWS-RunShellScript
```

Command:

```bash
sudo systemctl restart nginx
```

Systems Manager Run Command is available for EC2 at no additional Systems Manager charge.

## 17.2 Session Manager

Used for:

- manual troubleshooting
- incident investigation
- port-less terminal access

```text
Operator
   ↓
Systems Manager Session Manager
   ↓
SSM Agent
   ↓
Private EC2
```

## 17.3 Patch Manager

Used for:

```text
Inspector finding
      ↓
Patch decision
      ↓
Patch Manager
      ↓
Package update
```

## 17.4 Automation

Use Automation when remediation becomes multi-step:

```text
Create snapshot
     ↓
Patch
     ↓
Restart
     ↓
Health check
     ↓
Report
```

## 17.5 Fleet Manager / Inventory

Use for:

- instance inventory
- software inventory
- operating-system details
- operational visibility

---

<!-- PAGE 18 -->

# Page 18 — Verification and Closed-Loop Operations

A remediation is not complete when SSM says:

```text
Command = Success
```

The application must be healthy.

## 18.1 Verification Layers

### Layer 1 — Command

```text
SSM command status
```

### Layer 2 — Process

```bash
systemctl is-active nginx
```

### Layer 3 — Application

```text
HTTP health endpoint
```

### Layer 4 — Metrics

```text
CPU
5xx
latency
error rate
```

### Layer 5 — Security

```text
Inspector finding
still active?
```

## 18.2 Closed Loop

```text
DETECT
  ↓
ANALYZE
  ↓
REMEDIATE
  ↓
VERIFY
  ↓
HEALTHY?
 ├── YES → CLOSE
 └── NO  → ESCALATE
```

## 18.3 Circuit Breaker

If the same remediation happens repeatedly:

```text
Restart Nginx
Restart Nginx
Restart Nginx
Restart Nginx
```

stop automation.

Example rule:

```text
Maximum 3 automated remediations
within 15 minutes.
```

Then:

```text
Circuit Breaker
     ↓
Disable auto-remediation
     ↓
Human notification
```

---

<!-- PAGE 19 -->

# Page 19 — Centralized Logging and Evidence

## 19.1 Logging Architecture

```text
EC2
 │
 ▼
CloudWatch Logs
 │
 ├── Operational logs
 ├── Application logs
 ├── Remediation logs
 └── Security logs
        │
        ▼
       S3
```

CloudTrail:

```text
AWS API
  ↓
CloudTrail
  ↓
S3
```

## 19.2 Evidence Object

Example:

```text
s3://cloudops-evidence/
│
├── incidents/
│   └── INC-0001/
│       ├── event.json
│       ├── context.json
│       ├── ai-response.json
│       ├── policy-decision.json
│       ├── ssm-result.json
│       └── verification.json
│
└── reports/
    └── daily/
```

## 19.3 Retention

For the student project:

```text
CloudWatch:
7–14 days

S3:
30–90 days
```

Use S3 lifecycle rules to reduce long-term storage.

---

<!-- PAGE 20 -->

# Page 20 — Notifications and Operational Dashboard

## 20.1 SNS

Notifications should contain:

```text
Incident ID
Severity
Resource
Diagnosis
Action
AI confidence
Policy decision
SSM command ID
Verification result
```

Example:

```text
[CLOUDOPS GUARDIAN]

Incident: INC-0007
Severity: HIGH

Resource:
i-xxxxxxxx

Diagnosis:
Nginx worker instability

Action:
RESTART_NGINX

AI Confidence:
94%

Policy:
APPROVED

SSM:
Success

Verification:
HEALTHY
```

## 20.2 CloudWatch Dashboard

Create these widgets.

### Infrastructure

```text
EC2 CPU
Memory
Disk
Network
Status checks
```

### Application

```text
HTTP 5xx
Latency
Request count
Nginx errors
```

### Security

```text
Critical Inspector findings
High Inspector findings
Security Group changes
IAM changes
```

### AI Operations

```text
Incidents analyzed
Auto-remediations
Recommendations
Human approvals
Rejected actions
Failed remediations
```

### Reliability

```text
Mean time to detect
Mean time to remediate
Verification failures
Circuit breaker activations
```

---

<!-- PAGE 21 -->

# Page 21 — End-to-End Vulnerability Flow

## 21.1 Detailed Packet / Event Flow

```text
                 PRIVATE EC2
                     │
             SSM Agent / OS
                     │
                     │
              Inspector scan
                     │
                     ▼
            Inspector Finding
                     │
                     │ Event
                     ▼
              Amazon EventBridge
                     │
                     │ matched rule
                     ▼
             Lambda AI Engine
                     │
       ┌─────────────┼──────────────┐
       │             │              │
       ▼             ▼              ▼
 Inspector       CloudWatch      CloudTrail
 finding         logs/metrics    recent APIs
       │             │              │
       └─────────────┼──────────────┘
                     │
                     ▼
               Context Builder
                     │
                     ▼
                Amazon Bedrock
                     │
                     ▼
          Structured Recommendation
                     │
                     ▼
              Safety Validator
                     │
            ┌────────┴────────┐
            │                 │
         Approved           Rejected
            │                 │
            ▼                 ▼
           SSM            SNS / Dashboard
            │
            │ SendCommand
            ▼
         SSM Agent
            │
            ▼
        Private EC2
            │
            ▼
        Patch / Fix
            │
            ▼
        Verification
            │
            ▼
      CloudWatch / Inspector
            │
            ▼
         CLOSE / ESCALATE
```

## 21.2 Network Flow vs Control-Plane Flow

Important distinction:

### Network data path

```text
Client
 ↓
ALB
 ↓
Private EC2
 ↓
RDS
```

### Management path

```text
SSM Agent
 ↓
VPC Endpoint
 ↓
Systems Manager
```

### AI control path

```text
EventBridge
 ↓
Lambda
 ↓
Bedrock
 ↓
Lambda
 ↓
SSM API
 ↓
SSM Agent
 ↓
EC2
```

These are different flows and should not be mixed together in the architecture diagram.

---

<!-- PAGE 22 -->

# Page 22 — CloudWatch Incident Flow

## 22.1 Example: Nginx 502 Storm

Application produces:

```text
502 Bad Gateway
```

CloudWatch Agent sends logs:

```text
/var/log/nginx/error.log
```

Metric filter:

```text
"502"
```

Metric:

```text
Nginx5xxCount
```

Alarm:

```text
Nginx5xxCount >= 10
```

Alarm state:

```text
OK → ALARM
```

EventBridge receives the alarm state change.

Lambda:

```text
1. Identify instance
2. Get recent logs
3. Get CPU/memory
4. Get process information
5. Ask Bedrock
```

Bedrock:

```json
{
  "diagnosis": "Nginx worker instability",
  "action": "RESTART_NGINX",
  "confidence": 0.94,
  "requires_human_approval": false
}
```

Policy:

```text
Allowed
```

SSM:

```bash
sudo systemctl restart nginx
```

Verification:

```bash
systemctl is-active nginx
```

Then:

```text
CloudWatch 502 rate decreases
       ↓
Incident closed
```

---

<!-- PAGE 23 -->

# Page 23 — CloudTrail Security Change Flow

## 23.1 Example: Dangerous Security Group Change

An operator creates:

```text
TCP 22
Source: 0.0.0.0/0
```

CloudTrail records:

```text
AuthorizeSecurityGroupIngress
```

EventBridge rule matches the API event.

Lambda retrieves:

```text
Security Group
Source CIDR
Port
User/Role
Timestamp
Resource tags
```

AI analysis:

```text
Risk:
HIGH

Recommendation:
Remove public SSH rule.

Hardening:
Use Session Manager instead.
```

Policy:

```text
Security Group modification
=
HIGH RISK
```

Therefore:

```text
NO AUTO EXECUTION
```

The dashboard shows:

```text
HARDENING RECOMMENDATION

Remove:
0.0.0.0/0 → TCP 22

Replace with:
SSM Session Manager
```

This demonstrates AI-assisted security without giving the model unrestricted network-control authority.

---

# Page 24 — Cost Strategy

## 24.1 Cost Philosophy

The architecture is designed to maximize Free Tier / free-trial usage, but **not every service is permanently free**.

Particularly important:

- Bedrock is token-priced.
- Inspector becomes billable after its trial.
- Interface VPC endpoints are billable.
- NAT Gateway is billable.
- CloudWatch Logs can become billable.
- S3 storage/request charges can apply.
- EC2 public IPv4 addresses can incur charges.
- RDS can incur charges outside eligible Free Tier usage.

## 24.2 Cost-Optimized Initial Stack

| Service | Initial Role | Cost Strategy |
|---|---|---|
| IAM | Identity | No service charge |
| IAM Identity Center | SSO | Use for centralized access |
| VPC | Network | No VPC hourly charge |
| Security Groups | Firewall | No separate service charge |
| NACL | Subnet firewall | No separate service charge |
| EC2 | Application target | Keep instance count minimal |
| SSM | Management | Use EC2-managed-node capabilities |
| CloudWatch | Monitoring | Keep log volume low |
| CloudTrail | Audit | Use management events initially |
| EventBridge | Event routing | Filter aggressively |
| Lambda | AI orchestration | Low invocation volume |
| S3 | Evidence | Lifecycle old data |
| Inspector | Vulnerability | Use 15-day trial deliberately |
| Bedrock | AI | Use small prompts and low-volume calls |
| SNS | Notification | Email-only for lab |
| RDS | Optional data tier | Add only when needed |

## 24.3 Avoid Initially

```text
NAT Gateway
Transit Gateway
Network Firewall
CloudTrail Lake
High-volume data events
Large RDS
Multiple EC2 instances
Large CloudWatch log ingestion
```

The goal is to demonstrate architecture without generating unnecessary charges.

---

<!-- PAGE 25 -->

# Page 25 — Bedrock Cost-Control Design

Amazon Bedrock is the one component that must be treated differently from the permanent Free Tier services.

Bedrock pricing depends on the model, provider, service tier, and tokens consumed.

## 25.1 Reduce AI Calls

Do not:

```text
Every log line → Bedrock
```

Do:

```text
Logs
 ↓
Metric Filter
 ↓
Alarm
 ↓
EventBridge
 ↓
Only significant incidents → Bedrock
```

## 25.2 Reduce Prompt Size

Send:

```text
last 20–50 relevant log lines
```

not:

```text
entire 500 MB log
```

## 25.3 Context Compression

Lambda should summarize:

```text
CPU = 94%
Memory = 87%
Disk = 72%

502 count = 37

Relevant errors:
1. upstream timeout
2. worker exited
3. connection refused
```

Then Bedrock receives a small context.

## 25.4 Model Strategy

Do not hard-code the architecture around one model name.

Use:

```text
MODEL_ID environment variable
```

Then choose a low-cost supported model available in your Region/account.

If a Claude model such as Claude 3.5 Haiku is available to your account, it can be used as the low-cost reasoning model; otherwise select a currently supported low-cost Bedrock model.

---

# Page 26 — Failure Handling

## 26.1 Bedrock Failure

```text
Bedrock unavailable
      ↓
No remediation
      ↓
SNS alert
      ↓
Human investigation
```

## 26.2 Lambda Failure

EventBridge should retry according to the chosen target/retry configuration.

Use a dead-letter strategy where appropriate.

## 26.3 SSM Failure

```text
SSM command failed
      ↓
Verification
      ↓
Retry only if safe
      ↓
Otherwise escalate
```

## 26.4 AI JSON Failure

```text
Invalid JSON
     ↓
Reject
     ↓
Do not execute
```

## 26.5 Wrong Target

```text
Instance not found
       ↓
Reject
```

## 26.6 Repeated Failure

```text
Same incident
   ↓
3 attempts
   ↓
Circuit breaker
   ↓
Human approval
```

---

# Page 27 — Testing Plan

## Test 1 — High CPU

Create CPU stress.

Expected:

```text
CloudWatch
 ↓
Alarm
 ↓
EventBridge
 ↓
Lambda
```

## Test 2 — Nginx Failure

```bash
sudo systemctl stop nginx
```

Expected:

```text
CloudWatch
 ↓
EventBridge
 ↓
Bedrock
 ↓
RESTART_NGINX
 ↓
SSM
 ↓
nginx active
```

## Test 3 — Inspector Finding

Use a deliberately vulnerable package in a controlled lab.

Expected:

```text
Inspector
 ↓
Finding
 ↓
EventBridge
 ↓
Lambda
 ↓
Bedrock
 ↓
Patch recommendation
```

Do not intentionally introduce vulnerabilities into an important production resource.

## Test 4 — Security Group Change

Add:

```text
TCP 22
0.0.0.0/0
```

Expected:

```text
CloudTrail
 ↓
EventBridge
 ↓
AI
 ↓
Hardening recommendation
 ↓
Human approval
```

## Test 5 — Malformed AI Output

Force the test Lambda to provide invalid JSON.

Expected:

```text
Parser failure
 ↓
Reject
 ↓
No SSM execution
```

## Test 6 — Unauthorized Action

Make the test Bedrock response:

```json
{
  "recommended_action": "DELETE_RDS"
}
```

Expected:

```text
ALLOWLIST FAILURE
 ↓
REJECT
```

---

# Page 28 — Deployment Roadmap

## Phase 1 — Network

Build:

```text
VPC
Subnets
Route Tables
Security Groups
NACL
VPC Peering
```

## Phase 2 — Private Compute

Build:

```text
EC2
IAM Role
SSM Agent
SSM Endpoints
CloudWatch Agent
```

## Phase 3 — Monitoring

Build:

```text
CloudWatch Logs
Metrics
Metric Filters
Alarms
Dashboard
```

## Phase 4 — Audit

Build:

```text
CloudTrail
S3 evidence bucket
```

## Phase 5 — Vulnerability

Activate:

```text
Inspector
Patch Manager
```

## Phase 6 — Event Automation

Build:

```text
EventBridge
Lambda
SNS
```

## Phase 7 — AI

Build:

```text
Bedrock
Context Builder
JSON schema
Policy engine
```

## Phase 8 — Remediation

Build:

```text
SSM Run Command
Verification
Circuit breaker
```

## Phase 9 — Security Advisor

Build:

```text
Hardening recommendations
Risk scoring
Dashboard
```

## Phase 10 — Multi-VPC / Operations

Finalize:

```text
Operations VPC
Production VPC
VPC Peering
Centralized dashboard
```

---

# Page 29 — Operational Runbook

## Incident: Nginx Failure

### Detection

```text
CloudWatch Alarm
```

### Investigation

```text
CloudWatch Logs
```

### AI Diagnosis

```text
Bedrock
```

### Authorization

```text
Policy Engine
```

### Execution

```text
SSM Run Command
```

### Verification

```text
systemctl + HTTP + CloudWatch
```

### Closure

```text
S3 + CloudWatch + SNS
```

---

## Incident: Vulnerable Package

```text
Inspector
 ↓
Finding
 ↓
EventBridge
 ↓
Lambda
 ↓
Bedrock
 ↓
Patch recommendation
 ↓
Approval/Policy
 ↓
SSM Patch Manager
 ↓
Verification
 ↓
Finding review
```

---

## Incident: Public SSH Exposure

```text
CloudTrail
 ↓
EventBridge
 ↓
Lambda
 ↓
Bedrock
 ↓
Security recommendation
 ↓
High-risk classification
 ↓
Human approval
 ↓
Security Group change
 ↓
CloudTrail audit
```

---

# Page 30 — Final Architecture and Portfolio Value

## 30.1 Final Architecture

```text
                         CLOUDOPS GUARDIAN AI
                                  │
        ┌─────────────────────────┼──────────────────────────┐
        │                         │                          │
   OPERATIONS                 PRODUCTION                 AUDIT
        │                         │                          │
 EventBridge                ALB / EC2 / RDS             CloudTrail
 Lambda AI                  Private Network              S3
 Bedrock                    SSM Agent                    Logs
 CloudWatch
 SSM
        │
        └─────────────────────────┬──────────────────────────┘
                                  │
                                  ▼
                            EVENT ENGINE
                                  │
                                  ▼
                              AI ENGINE
                                  │
                     ┌────────────┴────────────┐
                     │                         │
               REMEDIATION                 HARDENING
                     │                         │
                     ▼                         ▼
                 POLICY                     REPORT
                     │
              ┌──────┴──────┐
              │             │
            AUTO          APPROVAL
              │             │
              └──────┬──────┘
                     ▼
                   SSM
                     │
                     ▼
                PRIVATE EC2
                     │
                     ▼
                 VERIFY
                     │
              ┌──────┴──────┐
              ▼             ▼
          HEALTHY       FAILED
              │             │
              ▼             ▼
            CLOSE        ESCALATE
```

## 30.2 What This Project Demonstrates

This single project demonstrates:

### Cloud Networking

- VPC design
- Multi-VPC architecture
- VPC Peering
- Three-tier subnet design
- Route tables
- Security Groups
- NACLs
- VPC endpoints
- Private EC2 networking

### CloudOps

- CloudWatch
- Alarms
- Logs
- Metric filters
- Event-driven operations
- Incident management
- Self-healing
- Operational dashboards

### Security

- IAM
- IAM Identity Center
- Least privilege
- CloudTrail
- Inspector
- Security Groups
- NACLs
- Encryption
- Private administration

### Systems Operations

- Systems Manager
- Session Manager
- Run Command
- Patch Manager
- Automation
- Fleet Manager
- Inventory

### AI Engineering

- Amazon Bedrock
- Context engineering
- Structured JSON output
- AI diagnosis
- AI recommendations
- Security-hardening advisor
- Confidence scoring
- AI safety boundaries

### Enterprise Engineering

- Separation of duties
- Auditability
- Approval workflows
- Circuit breakers
- Failure handling
- Evidence collection
- Cost controls

---

# Appendix A — Recommended AI Response Schema

```json
{
  "incident_id": "INC-0001",
  "incident_type": "application_failure",
  "severity": "HIGH",
  "confidence": 0.94,
  "diagnosis": "Nginx worker process instability",
  "recommended_action": "RESTART_NGINX",
  "target": {
    "instance_id": "i-xxxxxxxx",
    "service": "nginx"
  },
  "reason": "Repeated worker exits and upstream timeout errors were detected.",
  "security_hardening": [
    "Remove direct SSH exposure",
    "Use Systems Manager Session Manager",
    "Restrict application ingress to ALB security group"
  ],
  "requires_human_approval": false,
  "rollback_strategy": "Restart service only; no persistent configuration changes."
}
```

---

# Appendix B — Approved Action Registry

```text
ACTION                         RISK        AUTO?
-------------------------------------------------
RESTART_NGINX                  LOW        YES
RESTART_HTTPD                  LOW        YES
RESTART_DOCKER                 LOW        YES
CLEAR_CACHE                    LOW        YES
ROTATE_LOGS                    LOW        YES

PATCH_PACKAGE                  MEDIUM     CONDITIONAL
RESTART_EC2                    MEDIUM     CONDITIONAL
CHANGE_APP_CONFIG              MEDIUM     CONDITIONAL

MODIFY_SECURITY_GROUP          HIGH       NO
MODIFY_NACL                    HIGH       NO
MODIFY_ROUTE_TABLE             HIGH       NO
MODIFY_IAM                     CRITICAL   NO
DELETE_RDS                     CRITICAL   NO
DELETE_S3                      CRITICAL   NO
DISABLE_CLOUDTRAIL             CRITICAL   NO
DISABLE_INSPECTOR              CRITICAL   NO
```

---

# Appendix C — Required Resource Tags

Use tags as an automation safety boundary.

```text
Project=CloudOpsGuardian
Environment=Lab
CloudOpsManaged=true
RemediationMode=Auto
Owner=CloudOps
DataClassification=NonSensitive
```

Only resources with:

```text
CloudOpsManaged=true
```

should be eligible for automated remediation.

---

# Appendix D — Quick Revision

```text
USER
 │
 ▼
AWS / Application
 │
 ▼
CloudWatch / Inspector / CloudTrail
 │
 ▼
EventBridge
 │
 ▼
Lambda AI Engine
 │
 ├── Collect Context
 │
 ├── Bedrock
 │
 ├── Parse JSON
 │
 └── Safety Validator
 │
 ├───────────────┐
 ▼               ▼
AUTO            HUMAN
 │               │
 └───────┬───────┘
         ▼
        SSM
         │
         ▼
    Private EC2
         │
         ▼
    Verification
         │
    ┌────┴────┐
    ▼         ▼
 HEALTHY    FAILED
    │         │
    ▼         ▼
 CLOSE      ESCALATE
```

---

# Appendix E — AWS Official Reference Notes

Use the current AWS documentation before deployment because pricing, model availability, service limits, and endpoint requirements can change.

Key references to consult:

- AWS Systems Manager — VPC endpoints and Session Manager prerequisites
- AWS Systems Manager — Run Command
- AWS Systems Manager — Pricing
- Amazon Inspector — Pricing and free trial
- Amazon EventBridge — Pricing and event patterns
- AWS CloudTrail — Pricing and trail configuration
- Amazon Bedrock — Current model pricing and service tiers
- Amazon VPC — VPC and endpoint documentation
- AWS Lambda — Pricing

---

# Final Project Definition

**CloudOps Guardian AI** is not an AI chatbot connected to AWS.

It is an **event-driven cloud operations control system**.

```text
OBSERVE
   ↓
DETECT
   ↓
CORRELATE
   ↓
REASON
   ↓
RECOMMEND
   ↓
VALIDATE
   ↓
AUTHORIZE
   ↓
REMEDIATE
   ↓
VERIFY
   ↓
AUDIT
   ↓
IMPROVE
```

The central engineering rule is:

> **Bedrock provides intelligence; AWS controls provide authority.**

That separation is what makes the architecture suitable for a serious CloudOps/Security portfolio project rather than a simple generative-AI demonstration.









