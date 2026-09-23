# PROJECT DOCUMENTATION

## CloudOps360


**AWS Cloud Operations, Troubleshooting & Automated Cost Optimization Platform**

*A Serverless Platform for Proactive Cloud Support, Resource Health Monitoring, Compliance Checks, and Automated Cost Optimization Recommendations*

**Technology Stack:**
AWS Lambda • Amazon EventBridge • Amazon DynamoDB • Amazon API Gateway • Amazon SNS • Amazon S3 • AWS Config • React 18 + Vite • Python 3.12 (Boto3) • CloudFormation

Document Version: 3.0 (Expanded & Restructured) | Date: July 2026

*Prepared for Cloud Support / Cloud Operations / Associate Cloud Engineer Portfolio*

**[COMPANY / CANDIDATE NAME PLACEHOLDER]**

---

## How to Use This Document

This documentation was rebuilt and substantially expanded from the original draft to bring it to a professional, portfolio-ready and interview-ready standard, targeting 30-50 pages. Three types of colored callout boxes appear throughout. Please resolve all of them before sharing this document externally.

> **📷 SCREENSHOT PLACEHOLDER**
> Example: SCREENSHOT PLACEHOLDER — marks an exact spot where you should paste a screenshot of the AWS Console, dashboard UI, CLI output, or logs. Replace this box with your image once you have it; do not leave placeholders in the final version.

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Example: ACTION REQUIRED — FILL IN BY YOU — marks a spot where real code, exact values, or configuration from your actual implementation must be pasted in (the AI does not have access to your source code or AWS account). Search this document for this box to find every remaining fill-in item.

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> Example: GAP IDENTIFIED IN ORIGINAL DOCUMENT — marks a topic that a complete, professional project document should cover but that was missing or under-explained in the version supplied. Sensible draft content has been added; verify and correct it against your real implementation.

A consolidated checklist of every screenshot, every fill-in item, and every gap is repeated in the final appendices so you can work through them one by one without re-reading everything.

*Note on the Table of Contents below: Word builds this list from the document headings automatically, but it needs one manual refresh. After opening the file, right-click anywhere inside the Table of Contents and choose "Update Field → Update entire table" (on Mac: click the TOC and press the Update button that appears).*

---

## Table of Contents

1. Executive Summary
2. Introduction and Background
3. Problem Statement
4. Project Objectives and Success Criteria
5. System Architecture
6. Data Model (DynamoDB Schema)
7. AWS Services and Technology Stack
8. Detailed Feature Specification
9. Implementation Details
10. Security Architecture and Best Practices
11. Deployment Guide
12. Testing, Validation and Observability
13. Cloud Support Skills Demonstrated
14. Alignment with AWS Well-Architected Framework
15. Challenges Encountered and Troubleshooting Guide
16. Estimated Cost of Running CloudOps360
17. Future Enhancements and Roadmap
18. Conclusion
- Appendix A — Environment Variables
- Appendix B — API Contract Summary
- Appendix C — Sample Finding Object
- Appendix D — Consolidated Screenshot Checklist
- Appendix E — Consolidated Action-Required (Fill-In) Checklist
- Appendix F — Gaps Identified in the Source Document
- Appendix G — Glossary

---

## 1. Executive Summary

CloudOps360 is a serverless AWS platform that automates daily operational health checks, cost-leak detection, compliance evaluation, and troubleshooting guidance for Cloud Support and Cloud Operations teams. The system continuously discovers EC2 instances, EBS volumes, Elastic IPs, S3 buckets, DynamoDB tables, and AWS Config rule compliance status, then surfaces prioritized findings through email alerts and a modern React dashboard.

Built entirely on managed AWS services (Lambda, EventBridge, DynamoDB, API Gateway, SNS, S3, and Config), the platform requires no servers to manage, scales automatically, and follows least-privilege security practices. Each finding includes severity classification, root-cause analysis, and concrete remediation steps (Console navigation + AWS CLI commands), directly supporting faster incident resolution and reduced Mean Time to Resolution (MTTR).

This project serves as a comprehensive portfolio artifact demonstrating practical Cloud Support engineering capabilities: resource discovery, cost awareness, compliance monitoring, automation, observability, IAM design, and API-driven operations dashboards.

### At a Glance

| Attribute | Detail |
|---|---|
| Architecture style | Fully serverless, event-driven |
| Primary language | Python 3.12 (Boto3) for backend Lambdas; React 18 + Vite for frontend |
| Core AWS services | Lambda, EventBridge, DynamoDB, SNS, API Gateway, S3, IAM, CloudWatch, AWS Config |
| Scan cadence | Daily, automated (EventBridge cron) + on-demand via dashboard |
| Data model | Single DynamoDB table, scan_id partition key, timestamp sort key |
| Deployment method | Infrastructure as Code (CloudFormation, YAML) |
| Target roles this demonstrates | Cloud Support Engineer, Associate Cloud Engineer, Cloud Operations / NOC Analyst |

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> The source document did not include quantified outcomes (e.g., estimated MTTR reduction, estimated monthly savings identified in a real test run). Once you have run CloudOps360 against a test account, add 1-2 sentences of concrete numbers to this section — it is the single highest-impact addition you can make.

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Replace with real results, e.g.: "In a test AWS Free Tier account, CloudOps360 identified 3 unattached EBS volumes and 1 unassociated Elastic IP representing approximately $X/month in avoidable spend, and flagged 2 AWS Config non-compliant resources within one scan cycle."

---

## 2. Introduction and Background

Modern organizations running production and non-production workloads on Amazon Web Services face continuous operational pressure. Cloud Support teams must simultaneously monitor resource health, respond to incidents, enforce security and compliance baselines, and control cloud spend. Performing these tasks manually does not scale and frequently leads to delayed detection of idle resources, unattached storage, public exposure risks, and configuration drift.

CloudOps360 was conceived to answer a practical question that every Cloud Support engineer faces: "Can I operate, troubleshoot, secure, monitor, and optimize an AWS environment efficiently?" The platform provides a centralized, automated answer by combining scheduled discovery, structured analysis, actionable guidance, and real-time visibility.

The solution deliberately stays within serverless and managed services so that the focus remains on operational logic rather than infrastructure management. This mirrors the day-to-day reality of many support organizations that prefer operational tooling over heavy custom platforms.

### 2.1 Who This Document Is For

- Cloud Support / Cloud Operations hiring panels reviewing this as a portfolio project.
- Engineers who want to redeploy or extend CloudOps360 in their own AWS account.
- The project author, as a reference for demo talking points and interview preparation.

### 2.2 Document Conventions

- Section numbers follow the structure used throughout this document and match the Table of Contents.
- Grey boxes mark screenshot placeholders. Orange boxes mark fill-in-the-blank action items. Red boxes mark content gaps that were filled with reasonable draft text.
- Code and configuration are shown in code blocks; replace bracketed [PLACEHOLDER] values with your real values.

---

## 3. Problem Statement

Through observation of common AWS operational patterns and typical support tickets, the following recurring problems were identified:

- EC2 instances continue running with very low CPU utilization, generating avoidable compute cost.
- EBS volumes remain in the 'available' (unattached) state after instance termination or testing, continuing to incur storage charges.
- Elastic IP addresses stay allocated but unassociated, generating hourly charges.
- S3 buckets lack lifecycle policies, leading to accumulation of infrequently accessed data in higher-cost storage classes.
- Security and compliance configurations drift over time (public buckets, missing encryption, overly permissive security groups).
- Support teams lack a single pane of glass that combines cost signals, health status, and compliance posture.
- Troubleshooting remains reactive and knowledge is often tribal rather than captured as reusable guidance.
- Small and medium-sized teams rarely have dedicated FinOps or Cloud Center of Excellence resources, making automated tooling essential.

These issues result in higher monthly bills, longer incident resolution times, audit findings, and increased operational toil for support engineers.

### 3.1 Why This Matters to a Support Organization

CloudOps360 shifts detection of these issues from reactive (discovered during a bill spike or an audit) to proactive (surfaced daily, before they compound). Every finding already contains the diagnostic context a support engineer would otherwise gather manually across multiple console screens, directly reducing MTTR.

---

## 4. Project Objectives and Success Criteria

### 4.1 Primary Objectives

- Automate daily discovery of key AWS resource types (EC2, EBS, Elastic IP, S3, DynamoDB).
- Detect classic cost-leak patterns and attach estimated savings ranges where feasible.
- Integrate AWS Config to surface non-compliant rules alongside operational findings.
- Generate severity-tagged findings that include root cause and step-by-step remediation guidance.
- Deliver near-real-time email alerts via Amazon SNS after each scan.
- Expose scan results through a REST API consumed by a React operations dashboard.
- Apply security best practices: least-privilege IAM, encryption at rest, blocked public access.
- Provide a repeatable CloudFormation foundation for infrastructure provisioning.

### 4.2 Success Criteria

- Scanner successfully detects unattached EBS volumes and running EC2 instances created for testing.
- Findings are persisted in DynamoDB with scan_id, timestamp, severity counts, and full guidance text.
- SNS email is received within seconds of scan completion when issues exist.
- All four API endpoints (`/compliance`, `/resources`, `/reports/latest`, `/scan/trigger`) return valid JSON.
- React dashboard displays live data after switching from mock mode to the deployed API.
- CloudWatch Logs contain clean execution traces without permission or unhandled exception errors.

| Objective | How Success Is Measured |
|---|---|
| Automated daily discovery | EventBridge rule fires on schedule; CloudWatch Logs show a completed Scanner invocation every 24h |
| Cost-leak detection | Unattached EBS volumes / idle EIPs created in a test account are detected within one scan cycle |
| Compliance visibility | Non-compliant AWS Config rules appear in dashboard within one scan cycle |
| Actionable findings | Every finding includes severity, root cause text, and at least one Console or CLI fix step |
| Notifications & dashboard | SNS email received after a scan with findings; dashboard reflects same data via API |
| Security best practice | IAM policy contains no wildcard resource on mutating actions; S3 bucket has Block Public Access = on |

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> The source document listed success criteria as a bullet list but did not tie each one back to a specific project objective. The table above makes that mapping explicit — useful both as a self-check and as strong interview material ("how did you know it worked?").

---

## 5. System Architecture

### 5.1 High-Level Architecture

CloudOps360 adopts a fully serverless, event-driven architecture. There are no long-running servers. Compute is provided exclusively by AWS Lambda; scheduling by EventBridge; persistence by DynamoDB; notifications by SNS; and external access by API Gateway.

*Figure 1 — CloudOps360 high-level architecture: scheduled scan path (top-left), API/dashboard path (right), shared DynamoDB data store (center).*

The architecture separates concerns cleanly:

- **Scanner Lambda** — responsible only for discovery, analysis, persistence, and alerting.
- **API Lambda** — responsible only for reading stored results and serving the dashboard.
- **EventBridge** — owns the schedule and decouples time-based triggering from business logic.
- **DynamoDB** — acts as the single source of truth for scan history.
- **React frontend** — remains a pure consumer of the REST API.

### 5.2 Component Interaction Flow

*Figure 2 — Step-by-step sequence for one automated scan cycle.*

1. EventBridge rule fires on a daily schedule (or is invoked manually via test event).
2. CloudOps360-Scanner Lambda is invoked.
3. Scanner uses Boto3 to call Describe/List APIs on EC2, S3, DynamoDB, and Config.
4. Each interesting resource is transformed into a structured finding object containing severity, root cause, fix steps, and optional savings estimate.
5. A composite item (scan_id + timestamp + findings array + counters) is written to DynamoDB.
6. If findings exist, an SNS message is published to the alerts topic; subscribed email addresses receive the notification.
7. Support engineers open the React dashboard, which calls API Gateway.
8. API Gateway forwards requests to CloudOps360-API Lambda, which reads the latest DynamoDB item and returns dashboard-ready JSON.
9. Optional: User clicks "Trigger Scan" → `POST /scan/trigger` → API Lambda asynchronously invokes Scanner Lambda.

### 5.3 Design Principles

- Serverless first — no EC2 instances or containers for core logic.
- Loose coupling — EventBridge and SNS isolate producers from consumers.
- Least privilege — IAM role grants only the actions actually required.
- Observability by default — every execution writes structured CloudWatch Logs.
- Actionable output — findings are written for humans (support engineers), not only machines.
- Portfolio clarity — architecture is simple enough to explain in an interview in under five minutes.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the deployed AWS architecture as seen in the AWS Console (e.g., the CloudFormation stack's Resources tab, or a Lambda function's Configuration → Triggers view showing the EventBridge trigger).

---

## 6. Data Model (DynamoDB Schema)

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> The source document referenced a DynamoDB item shape only informally (in Appendix C: 'Sample Finding Object') and never defined the actual table's key schema or full item structure. A complete architecture section needs this, since it is one of the first things a reviewer or interviewer will ask about. A reasonable draft schema is provided below, consistent with the partition key (scan_id) and sort key (timestamp) mentioned in Section 8.1 — replace it with your table's real attribute names if they differ.

### 6.1 Table: CloudOps360-Findings (draft — confirm actual table name)

| Attribute | Type | Description |
|---|---|---|
| scan_id (Partition Key) | String | Unique identifier for a scan run |
| timestamp (Sort Key) | String (ISO 8601) | When the scan was executed — enables chronological ordering |
| findings | List\<Map\> | Array of finding objects (see 6.2 and Appendix C) |
| total_issues | Number | Total count of findings in this scan |
| high_severity | Number | Count of High-severity findings in this scan |
| findings_count | Number | Total resources evaluated |
| triggered_by | String | "scheduled" or "manual" |

### 6.2 Finding Object Shape

This mirrors the conceptual structure already defined in Appendix C, shown here as a concrete JSON example for reference while reading the Architecture and Implementation sections:

```json
{
  "resource_type": "EBS",
  "resource_id": "vol-0123456789abcdef0",
  "issue": "Volume is unattached",
  "severity": "Medium",
  "root_cause": "Volume was detached from a terminated instance and never deleted",
  "fix_steps": "Console: EC2 -> Elastic Block Store -> Volumes -> Delete volume. CLI: aws ec2 delete-volume --volume-id vol-0123456789abcdef0",
  "estimated_savings": "$4.00/month"
}
```

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Confirm the exact DynamoDB table name and run: `aws dynamodb describe-table --table-name <your-table-name>` to verify the KeySchema and AttributeDefinitions match what's documented above.

### 6.3 Access Patterns

- `GET /reports/latest` → Query DynamoDB ordered by timestamp descending, limit 1, for the most recent scan item.
- `GET /resources` → Read recent scan_id items and flatten their findings arrays into a single list for the table view.
- `GET /compliance` → Aggregate total_issues, high_severity, and resourcesByService across a recent time window for summary cards and the trend chart.

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Confirm whether `/resources` and `/compliance` use a DynamoDB Scan or Query. If a Scan is used, note this as a known scaling limitation in Section 13 (Challenges) or the Troubleshooting appendix — Scans become slower and more expensive as scan history grows; a GSI (constant partition key + timestamp sort key) would be the fix.

---

## 7. AWS Services and Technology Stack

| Service / Technology | Role in CloudOps360 | Key Benefit |
|---|---|---|
| AWS Lambda | Scanner + API compute | No server management, auto scaling, pay-per-use |
| Amazon EventBridge | Daily schedule + event routing | Reliable time-based triggering |
| Amazon DynamoDB | Scan results store | Serverless NoSQL, on-demand capacity |
| Amazon SNS | Email alerting | Simple pub/sub for support notifications |
| Amazon S3 | Report storage (future) | Durable, encrypted object storage |
| Amazon API Gateway | REST front-end for dashboard | Managed HTTP endpoints + CORS |
| AWS IAM | Execution roles & policies | Fine-grained access control |
| Amazon CloudWatch | Logs, metrics, monitoring | Operational visibility |
| AWS Config | Compliance evaluation | Managed rule compliance status |
| Boto3 (Python) | AWS SDK inside Lambdas | Native AWS API access |
| React 18 + Vite | Operations dashboard UI | Modern, fast frontend |
| CloudFormation | Infrastructure as Code | Repeatable environment creation |

All selected services offer free-tier or low-cost starting points, making the project suitable for individual learning accounts while remaining representative of production patterns.

### 7.1 Why Serverless (Design Rationale)

- Cost: pay-per-invocation instead of an always-on process for a job that runs once a day plus light API traffic.
- Operational simplicity: no OS patching, no capacity planning — matches the small-team / no-dedicated-FinOps-staff scenario described in the Problem Statement.
- Natural fit for event-driven work: a daily scan is inherently a scheduled event, and API requests are inherently discrete invocations.

---

## 8. Detailed Feature Specification

### 8.1 Resource Discovery

The Scanner enumerates the following resource families on every run:

- EC2 instances currently in the running state (instance ID and type recorded).
- EBS volumes whose status is 'available' (unattached).
- Elastic IP addresses that have neither an InstanceId nor a NetworkInterfaceId association.
- All S3 buckets in the account.
- All DynamoDB tables in the account.

Discovery uses standard Describe/List APIs and is intentionally read-only, ensuring the Scanner itself cannot modify production resources.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the Resources table view in the dashboard showing discovered EC2/EBS/S3/DynamoDB items.

### 8.2 Cost Optimization Signals

Certain findings are annotated with estimated monthly savings ranges derived from public AWS pricing heuristics (for example, an unassociated Elastic IP is approximately $3.65 per month). While not a full Cost Explorer replacement, these signals highlight the most common idle-cost patterns that support teams encounter.

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Document the exact pricing constants used for each resource type. Paste the relevant constants or Boto3 Pricing API calls from your Scanner code, for example:

```python
# Example placeholder - replace with your real constants
ELASTIC_IP_HOURLY_RATE = 0.005  # USD, unassociated EIP, us-east-1
EBS_GP3_PRICE_PER_GB_MONTH = 0.08

def estimate_eip_savings():
    return round(ELASTIC_IP_HOURLY_RATE * 730, 2)  # ~$3.65/month
```

### 8.3 Compliance Integration

When AWS Config is enabled in the account, the Scanner queries `DescribeComplianceByConfigRule` and surfaces any rule whose ComplianceType is NON_COMPLIANT. This brings governance findings into the same operational view as cost and health issues.

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> The source document did not name which specific AWS Config rules are evaluated. List the actual managed or custom Config rules enabled in your account, for example: `s3-bucket-public-read-prohibited`, `restricted-ssh`, `encrypted-volumes`, `iam-user-mfa-enabled`. This level of specificity is what separates a real implementation from a generic description in an interview.

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> List your actual AWS Config rule names here, one per line, replacing this box.

### 8.4 Troubleshooting Guidance Engine

Every finding is enriched with four support-oriented fields:

- **Severity** — High, Medium, or Low.
- **Root Cause** — concise explanation of why the condition exists.
- **Fix Steps** — numbered human instructions plus corresponding AWS CLI commands where applicable.
- **Estimated Savings** — optional monetary or risk-oriented note.

This design mirrors the way experienced support engineers document resolution steps for knowledge bases and runbooks.

Severity is assigned using the following draft rule set — confirm or replace with your actual thresholds:

| Severity | Example Trigger Condition |
|---|---|
| High | Public S3 bucket / disabled encryption / non-compliant security-related Config rule |
| Medium | Unattached EBS volume, unassociated Elastic IP, idle running EC2 instance |
| Low | Informational findings, e.g. missing resource tags |

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Replace the severity table above with your project's real severity logic (paste the relevant if/elif block from the Scanner Lambda).

### 8.5 Notifications

After a successful scan that produces findings, an SNS notification is published. The email subject and body contain the scan identifier, total issue count, and high-severity count, directing the recipient to DynamoDB or the dashboard for full details.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the SNS email alert as received in an inbox (subject line + body).

### 8.6 Operations Dashboard

The React dashboard consumes four REST endpoints and presents:

- Compliance summary cards (total resources, compliant / non-compliant counts, rate).
- Alerts panel sorted by severity.
- Resources table with issue and remediation columns.
- Latest report metadata.
- Manual "Trigger Scan" action.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the full React dashboard home / compliance overview screen.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the Alerts panel sorted by severity.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the Resources table with an active filter applied.

---

## 9. Implementation Details

### 9.1 Scanner Lambda (Core Engine)

Runtime: Python 3.12. The handler performs sequential discovery blocks wrapped in try/except so that failure of one resource type does not abort the entire scan. Findings are accumulated in a list and written as a single DynamoDB item keyed by scan_id (partition) and timestamp (sort). Environment variables supply the DynamoDB table name, SNS topic ARN, and report bucket name.

**Key design choices**

- Idempotent writes — each run creates a new scan_id, preserving history.
- Defensive coding — individual service failures are logged and skipped.
- Human-readable output — findings are ready for both API consumers and email.

**High-level pseudocode**

```python
def lambda_handler(event, context):
    findings = []
    for scan_fn in [scan_ec2, scan_ebs, scan_eips, scan_s3, scan_dynamodb, scan_config]:
        try:
            findings += scan_fn()
        except Exception as e:
            logger.error(f"{scan_fn.__name__} failed: {e}")

    scan_item = build_scan_item(findings)  # scan_id, timestamp, counters
    table.put_item(Item=scan_item)

    if findings:
        publish_sns_alert(scan_item)

    return {"statusCode": 200, "body": json.dumps({"scan_id": scan_item["scan_id"]})}
```

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Replace the pseudocode above with (or place alongside) the real function signatures from your Lambda source, e.g. paste the actual `scan_ebs()` function body. If the file is large, paste the most interesting/representative function rather than the entire file.

### 9.2 API Lambda and REST Endpoints

The API Lambda implements path-based routing for four operations. It reads the most recent DynamoDB item (scan ordered by timestamp) and transforms the raw findings into the exact JSON shapes expected by the React frontend. The `/scan/trigger` endpoint uses an asynchronous Lambda invoke (`InvocationType=Event`) so the HTTP response returns immediately while the scan runs in the background.

| Method | Path | Description |
|---|---|---|
| GET | /compliance | Returns summary, alerts array, resourcesByService map, and 7-day trend |
| GET | /resources | Returns flat list of findings for the resources table |
| GET | /reports/latest | Returns latest scanId, timestamp, and issue counters |
| POST | /scan/trigger | Asynchronously invokes Scanner; returns queued status |

**Example request/response — GET /compliance (draft, confirm actual shape)**

```json
// Response 200 OK
{
  "lastScan": "2026-07-27T02:00:00Z",
  "totalResources": 42,
  "compliant": 37,
  "nonCompliant": 5,
  "complianceRate": 0.88,
  "alerts": [ { "severity": "High", "issue": "Public S3 bucket", "..." } ],
  "resourcesByService": { "EC2": 10, "EBS": 8, "S3": 6 },
  "trend": [ { "date": "2026-07-25", "total": 4 }, { "date": "2026-07-26", "total": 6 } ]
}
```

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Paste the real JSON response for each of the four endpoints (use the browser Network tab or a curl call against your deployed API Gateway URL). This is also the source of truth for Appendix B.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of a successful API call in Postman/curl/browser DevTools for at least one endpoint, showing status 200 and the JSON body.

### 9.3 Frontend Dashboard

Built with React 18, Vite, and Tailwind CSS. During development the application can run entirely on mock data (`USE_MOCKS = true`). Switching to live mode requires only setting `VITE_API_BASE_URL` to the API Gateway invoke URL and flipping the mock flag. Axios interceptors attach an optional Bearer token, preparing the application for future Cognito integration.

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Confirm the exact tech versions used (React 18.x, Vite x.x, Tailwind x.x) by pasting the relevant lines from package.json.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the deployed/running frontend loading real data from the API (not mock data).

### 9.4 Infrastructure as Code

A CloudFormation template provisions the IAM role, DynamoDB table, S3 bucket (with encryption and public-access block), SNS topic, both Lambda functions, EventBridge rule, API Gateway REST API with the four resources/methods, and necessary Lambda permissions. After stack creation the full Python source is deployed into the Lambda functions via the console or CLI `update-function-code`.

**Template resource skeleton (draft — confirm against your real template)**

```yaml
Resources:
  ScannerFunctionRole: { Type: AWS::IAM::Role }
  ScannerFunction: { Type: AWS::Lambda::Function }
  ApiFunction: { Type: AWS::Lambda::Function }
  FindingsTable: { Type: AWS::DynamoDB::Table }
  ReportsBucket: { Type: AWS::S3::Bucket }
  AlertsTopic: { Type: AWS::SNS::Topic }
  DailyScanRule: { Type: AWS::Events::Rule }
  CloudOps360Api: { Type: AWS::ApiGateway::RestApi }
```

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Paste your actual CloudFormation template's Resources: section (or a link to the file in your repository) in place of the skeleton above.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the CloudFormation stack's Events tab showing CREATE_COMPLETE for all resources.

---

## 10. Security Architecture and Best Practices

### 10.1 Identity and Access Management

A single execution role is shared by both Lambdas. The role trusts only the `lambda.amazonaws.com` service principal. Attached permissions are scoped to:

- `ec2:Describe*` — read-only discovery.
- `s3:List*` and limited Get — bucket enumeration.
- `dynamodb:PutItem`, `Scan`, `GetItem`, `ListTables` — persistence and read.
- `sns:Publish` — alerting.
- `config:DescribeComplianceByConfigRule` — compliance.
- `lambda:InvokeFunction` — only for the Scanner function (API → Scanner).
- `logs:*` — CloudWatch Logs (via AWSLambdaBasicExecutionRole).

No AdministratorAccess or broad wildcards beyond the necessary Describe/List patterns are retained in the final policy.

**Draft IAM policy JSON (confirm against real policy)**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["ec2:Describe*", "config:DescribeComplianceByConfigRule"], "Resource": "*" },
    { "Effect": "Allow", "Action": ["dynamodb:PutItem", "dynamodb:Scan", "dynamodb:GetItem"],
      "Resource": "arn:aws:dynamodb:[REGION]:[ACCOUNT-ID]:table/[TABLE-NAME]" },
    { "Effect": "Allow", "Action": ["sns:Publish"],
      "Resource": "arn:aws:sns:[REGION]:[ACCOUNT-ID]:[TOPIC-NAME]" },
    { "Effect": "Allow", "Action": ["lambda:InvokeFunction"],
      "Resource": "arn:aws:lambda:[REGION]:[ACCOUNT-ID]:function:CloudOps360-Scanner" }
  ]
}
```

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Paste your actual IAM policy JSON (IAM console → Roles → execution role → Permissions).

### 10.2 Data Protection

- S3 bucket has Block Public Access fully enabled and default SSE-S3 encryption.
- DynamoDB encryption at rest is enabled by default.
- No secrets are hard-coded; configuration travels through environment variables.

### 10.3 Network and Exposure

For portfolio simplicity the API Gateway endpoints are currently open (AuthorizationType NONE). In a production hardening phase, Amazon Cognito user pools or API keys would be introduced. CORS headers are explicitly returned by the API Lambda to allow the React origin.

### 10.4 Monitoring and Audit

CloudWatch Logs capture every invocation. CloudTrail (account-level) records API activity. IAM Access Analyzer can be enabled to surface any unintended external access findings on the S3 bucket or other resources.

### 10.5 Threat Model Notes (new)

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> The source document listed security controls but did not discuss residual risk if a control were missing or bypassed — useful in an interview to show risk-based thinking. A short threat model is added below.

| Risk | Current Mitigation | Residual Risk / Next Step |
|---|---|---|
| Open API Gateway endpoints | Portfolio simplicity; CORS restricted to known origin | Add Cognito User Pool authorizer or API key + usage plan before any production use |
| Over-broad Describe* wildcard | Scoped to read-only Describe/List actions only | Consider AWS-managed ReadOnlyAccess-style boundary + SCP if deployed org-wide |
| S3 report bucket exposure | Block Public Access + SSE-S3 encryption enabled | Add bucket policy explicitly denying non-TLS requests (aws:SecureTransport) |
| No authentication on manual trigger endpoint | Low impact (re-running a read-only scan) | Rate-limit via API Gateway usage plan to prevent abuse/cost spikes |

---

## 11. Deployment Guide

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> The source document described the CloudFormation template's contents (Section 9.4) but did not include a step-by-step deployment procedure. This is one of the most important sections for a portfolio project because it lets a reviewer, or your future self, actually stand the project up. A draft procedure is provided below — walk through it against your real setup and correct any step that differs.

### 11.1 Prerequisites

- An AWS account with administrator or sufficiently scoped IAM permissions.
- AWS CLI v2 installed and configured (`aws configure`) with a valid access key/secret or SSO profile.
- Node.js (for the React frontend build) and Python 3.12 (for local Lambda testing, optional).
- Git, to clone the project repository.

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Confirm minimum Node.js and Python versions actually used, and list any additional CLI tools (e.g. SAM CLI) if you used them for local testing.

### 11.2 Step-by-Step Deployment

**Step 1 — Clone the repository**

```bash
git clone [YOUR-REPOSITORY-URL]
cd cloudops360
```

**Step 2 — Review and edit CloudFormation parameters**

Open the CloudFormation template and confirm/adjust parameters such as table name, bucket name (must be globally unique), notification email, and schedule expression.

```yaml
Parameters:
  NotificationEmail:
    Type: String
    Default: [YOUR-EMAIL@example.com]
  ScanSchedule:
    Type: String
    Default: "cron(0 2 * * ? *)"  # 02:00 UTC daily
```

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Paste the real Parameters: block from your template.

**Step 3 — Deploy the stack**

```bash
aws cloudformation deploy \
  --template-file infrastructure/cloudops360.yaml \
  --stack-name cloudops360 \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides NotificationEmail=[YOUR-EMAIL@example.com]
```

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the terminal output showing successful stack deployment (or the CloudFormation console showing CREATE_COMPLETE).

**Step 4 — Confirm the SNS subscription**

SNS will send a confirmation email to the address in NotificationEmail. You must click "Confirm subscription" before alert emails will be delivered.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the SNS subscription confirmation email or the confirmed subscription in the SNS console.

**Step 5 — Deploy Lambda function code**

If the CloudFormation template provisions the functions with placeholder code, package and update the real code:

```bash
cd backend/scanner
zip -r scanner.zip .
aws lambda update-function-code --function-name CloudOps360-Scanner --zip-file fileb://scanner.zip

cd ../api
zip -r api.zip .
aws lambda update-function-code --function-name CloudOps360-API --zip-file fileb://api.zip
```

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Confirm the actual folder structure and packaging commands used (e.g., did you include a requirements.txt / vendored dependencies layer?).

**Step 6 — Configure and build the frontend**

```bash
cd frontend
cp .env.example .env
# set VITE_API_BASE_URL to your API Gateway invoke URL
# set USE_MOCKS=false
npm install
npm run build
```

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the deployed/running frontend loading real data from the API (not mock data).

**Step 7 — Run a manual scan and verify end-to-end**

- Trigger a scan from the dashboard (or invoke the Scanner Lambda directly from the console/CLI).
- Confirm a new item appears in DynamoDB.
- Confirm the SNS email is received (if findings > 0).
- Confirm the dashboard reflects the new scan.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the DynamoDB console showing a scan item with findings.

### 11.3 Teardown / Cost Control

To avoid ongoing charges after a demo, delete the stack:

```bash
aws cloudformation delete-stack --stack-name cloudops360
```

- Verify the S3 report bucket is empty first — CloudFormation will not delete a non-empty bucket by default.
- Manually confirm removal of any resources created outside the stack (e.g., test EC2 instances used for validation in Section 12).

---

## 12. Testing, Validation and Observability

### 12.1 Functional Test Cases Executed

- Created an unattached gp3 EBS volume and confirmed it appeared as a High-severity finding.
- Launched a t2.micro EC2 instance and verified the running-instance finding.
- Allocated an Elastic IP without association and validated detection.
- Confirmed DynamoDB received a new item containing scan_id, timestamp, findings array, total_issues, and high_severity.
- Verified SNS email delivery with correct subject and summary counts.
- Exercised `GET /compliance`, `GET /resources`, `GET /reports/latest` from API Gateway test console and from the React app.
- Invoked `POST /scan/trigger` and observed a new scan appear in DynamoDB within approximately 30 seconds.
- Inspected CloudWatch Log groups for both Lambdas; no AccessDenied or unhandled exceptions remained after permission fixes.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the test EBS volume in the EC2 console (state: available) alongside the matching finding in the dashboard.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of CloudWatch Logs Insights output confirming a successful Scanner run (start/end, no errors).

### 12.2 Test Coverage Summary (new)

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> The source document listed manual test cases but did not summarize coverage or note automated testing status. The table below organizes this and highlights a real gap: no automated/unit tests were described. Consider adding at least a few Pytest unit tests for the severity-assignment logic before finalizing the project — it is a common interview question ("how did you test this?").

| Layer | Tested? | Method |
|---|---|---|
| Resource discovery (Boto3 calls) | Yes | Manual — created real test resources |
| Severity / rule logic | Partially | Manual observation only — no unit tests |
| DynamoDB write/read | Yes | Manual — console inspection |
| SNS delivery | Yes | Manual — inbox check |
| API Gateway endpoints | Yes | Manual — console + dashboard |
| Frontend rendering | Yes | Manual — visual check |
| IAM least-privilege boundary | Not confirmed | TODO — run IAM Access Analyzer / Policy Simulator |

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> If you add automated tests (e.g., pytest with moto for mocking AWS calls), document them here and in a short "How to run tests" subsection.

### 12.3 Observability Practices

CloudWatch Logs Insights queries were used to isolate errors quickly, for example filtering on `ERROR|Exception|AccessDenied`. Log streams were examined after every code change. This mirrors the primary investigation technique used by professional Cloud Support engineers.

```
fields @timestamp, @message
| filter @message like /ERROR/ or @message like /Exception/ or @message like /AccessDenied/
| sort @timestamp desc
| limit 50
```

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of a CloudWatch Logs Insights query and its results.

---

## 13. Cloud Support Skills Demonstrated

| Skill Domain | Concrete Evidence in CloudOps360 |
|---|---|
| Resource Discovery | Boto3 Describe/List calls across EC2, EBS, EIP, S3, DynamoDB |
| Cost Awareness | Detection of unattached volumes, unassociated EIPs, idle signals |
| Compliance Monitoring | AWS Config non-compliant rule collection and display |
| Troubleshooting Mindset | Severity + root cause + Console/CLI remediation steps |
| Automation | EventBridge schedule + manual API trigger |
| Observability | Structured CloudWatch Logs + Logs Insights usage |
| IAM & Security | Least-privilege role, resource policies, encryption, public-access blocks |
| API Design | Clean REST contract consumed by a real frontend |
| Incident Simulation | Ability to create resources and immediately observe detection |
| Infrastructure as Code | CloudFormation template covering core services |
| Customer Communication | Human-readable findings suitable for support emails |

---

## 14. Alignment with AWS Well-Architected Framework

Although a formal Well-Architected Review was not performed, the design intentionally maps to several pillars, summarized visually below and detailed in the subsections that follow.

*Figure 3 — CloudOps360's alignment with the six AWS Well-Architected Framework pillars.*

**Operational Excellence**
- Automated daily scans reduce manual checking.
- Findings are versioned by scan_id, enabling historical comparison.
- Dashboard and SNS provide multiple operator interfaces.

**Security**
- Least-privilege IAM, encryption at rest, blocked public access.
- No long-lived credentials in code.

**Reliability**
- Serverless components with built-in multi-AZ characteristics.
- Defensive try/except blocks prevent total scan failure.

**Performance Efficiency**
- On-demand DynamoDB and short-lived Lambda executions.
- Asynchronous invoke for scan trigger keeps API latency low.

**Cost Optimization**
- The platform itself is low-cost (free-tier friendly).
- Its primary purpose is to surface cost-saving opportunities for the wider estate.

**Sustainability**
- Serverless execution model avoids idle compute.
- Encourages cleanup of unused resources, reducing overall footprint.

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> The source document covered the six pillars in text only. A visual summary (Figure 3) has been added so this section can also serve as a single interview-ready slide if needed.

---

## 15. Challenges Encountered and Troubleshooting Guide

This section starts with the five real challenges recorded during development, then expands into a full component-by-component troubleshooting runbook covering the most common failure modes for a project of this shape (EventBridge + Lambda + DynamoDB + SNS + API Gateway + React) — useful both as a debugging reference and as strong interview material.

### 15.1 Challenges Encountered During Development (original findings)

| Challenge | Root Cause | Resolution |
|---|---|---|
| API Gateway returned path not found | Path matching too strict (endswith) | Relaxed matching + printed full event for debugging |
| POST /scan/trigger returned 500 | Missing lambda:InvokeFunction permission | Added inline policy + environment variable for function name |
| EventBridge produced no CloudWatch Logs | Missing resource-based permission on Lambda | Added events.amazonaws.com permission with SourceArn |
| Dashboard showed only mocks | USE_MOCKS still true + wrong base URL | Updated .env and endpoints.js flag |
| IAM role too broad initially | Speed of development | Replaced with scoped custom policy before finalization |

Each challenge reinforced classic support skills: reading CloudWatch Logs, interpreting IAM evaluation, validating resource-based policies, and iterative testing.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of the real path-matching error from CloudWatch Logs, plus the corrected event handling code, to document the first challenge with full fidelity.

### 15.2 Scanner Lambda / EventBridge — Additional Failure Modes

| Symptom | Likely Root Cause | Fix |
|---|---|---|
| Lambda times out mid-scan | Default 3-second timeout too low for multiple Boto3 Describe calls across services | Increase Lambda timeout to 30-60s in function configuration; consider parallelizing service calls |
| Lambda runs but finds zero resources | Wrong AWS region configured, or IAM role missing Describe/List permission for a service | Confirm boto3 client region matches where test resources exist; review CloudWatch Logs for AccessDenied errors |
| Duplicate scan_id / overwritten items | scan_id generated from a low-resolution timestamp string causing collisions on rapid manual triggers | Use uuid4() for scan_id instead of a timestamp string, or add a random suffix |
| Cold start latency spikes | Larger dependency package or custom layer | Avoid bundling boto3 explicitly (already in the runtime); keep the deployment package small |

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> These four rows are new additions — they were not present in the original Challenges table but are common failure modes for a scan-based Lambda. Keep this table even if you haven't hit these yet; anticipating failure modes is exactly what Cloud Support interviews probe for.

### 15.3 DynamoDB — Additional Failure Modes (new)

| Symptom | Likely Root Cause | Fix |
|---|---|---|
| ValidationException on PutItem | Item shape doesn't match expected types (e.g., a float sent where DynamoDB requires Decimal) | Convert all Python floats to decimal.Decimal before writing; DynamoDB's boto3 resource API rejects native float |
| ProvisionedThroughputExceededException | Table using Provisioned capacity mode with too-low RCU/WCU under load or a manual-trigger burst | Switch table billing mode to On-Demand (PAY_PER_REQUEST), which fits this workload's spiky, low-volume pattern |
| /resources endpoint slow as history grows | Full table Scan used instead of a Query against an indexed access pattern | Add a Global Secondary Index and Query instead of Scan (see Section 6.3) |
| Old items never expire, table grows unbounded | No TTL attribute configured | Add a DynamoDB TTL attribute (e.g., expires_at) on historical scan items and enable TTL on the table |

### 15.4 SNS Notification — Additional Failure Modes (new)

| Symptom | Likely Root Cause | Fix |
|---|---|---|
| No email received after a scan with findings | SNS subscription still in "Pending confirmation" state | Check SNS console → Subscriptions; resend confirmation if needed; check spam folder |
| AuthorizationError on sns:Publish | Lambda execution role missing sns:Publish on the specific topic ARN | Add sns:Publish action scoped to the topic ARN in the Scanner Lambda's IAM policy |
| Emails delayed by several minutes | Normal SNS best-effort delivery latency, or downstream spam filtering | Not usually a code issue; verify via SNS delivery status logging if consistently delayed |

### 15.5 API Gateway / API Lambda — Additional Failure Modes (new)

| Symptom | Likely Root Cause | Fix |
|---|---|---|
| 403 {"message":"Missing Authentication Token"} | Requested path/method not deployed, or wrong stage in the invoke URL | Confirm the exact path and HTTP method are deployed in API Gateway and that the stage is included in the URL |
| CORS errors in browser console | API Gateway response missing Access-Control-Allow-Origin header, or OPTIONS preflight not configured | Enable CORS on each resource; return CORS headers from Lambda proxy integration responses, including on error paths |
| 502 Bad Gateway from API Gateway | Lambda returned a malformed response (missing statusCode or body as non-string) | Ensure Lambda always returns {statusCode, headers, body: JSON.stringify(...)} even on error paths |
| POST /scan/trigger returns before scan finishes but dashboard shows stale data | Expected — trigger is asynchronous by design | Add a polling or "last updated" indicator in the frontend so users know a scan is in progress |

### 15.6 Frontend / Dashboard — Additional Failure Modes (new)

| Symptom | Likely Root Cause | Fix |
|---|---|---|
| Dashboard shows mock data instead of live data | VITE_API_BASE_URL not set, or build performed before .env was configured | Set VITE_API_BASE_URL in .env, then rebuild (npm run build) — Vite inlines env vars at build time, not runtime |
| Blank white screen after deployment | SPA routing not configured on static hosting (e.g., S3/CloudFront) causing 404 on refresh | Configure S3/CloudFront error document to serve index.html for 404s, or use hash-based routing |
| Network errors only in production, not local dev | CORS misconfiguration only manifests once frontend and API are on different origins | See CORS fix in Section 15.5; verify allowed origin matches the deployed frontend URL exactly |

### 15.7 General Debugging Checklist

- Reproduce the issue and immediately check CloudWatch Logs for the relevant Lambda — most issues surface as a clear stack trace.
- Confirm the AWS region is consistent across the CLI profile, Lambda environment, and where test resources were created.
- For IAM errors, read the AccessDenied message literally — it names the exact action and resource that was denied.
- For API errors, check API Gateway's own execution logs (separate from the Lambda's logs) if enabled — they show whether the request even reached the Lambda.
- When in doubt, isolate: invoke the Lambda directly from the console with a test event before blaming the layer above it (API Gateway, frontend).

---

## 16. Estimated Cost of Running CloudOps360

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> A cost estimate was entirely absent from the source document, which is a notable gap for a project whose stated purpose includes cost optimization — a reviewer will reasonably ask "what does running this cost?" The table below is a rough, defensible estimate for a light, single-account portfolio deployment (us-east-1 pricing, order-of-magnitude only). Update it with AWS Pricing Calculator output for precision.

| Service | Usage Assumption | Approx. Monthly Cost |
|---|---|---|
| Lambda (Scanner + API) | ~35 invocations/month, <1s avg duration, 256MB | Effectively $0 (within Free Tier: 1M requests + 400,000 GB-s/month) |
| EventBridge | 1 rule, 30 invocations/month | $0 (rules and low-volume events are free / negligible) |
| DynamoDB | On-Demand mode, <1,000 items, light read/write | $0–$1 (within or near Free Tier) |
| SNS | ~30 emails/month | $0 (first 1,000 email notifications/month free) |
| API Gateway | REST API, low request volume for a demo | $0–$1 (Free Tier covers 1M calls for 12 months on new accounts) |
| S3 (report bucket) | Minimal storage, no objects yet (future use) | <$0.10 |
| CloudWatch Logs | Standard log retention, low volume | <$0.50 |
| **Total (approximate)** | | **$0–$3 / month for a portfolio-scale deployment** |

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Run the actual AWS Pricing Calculator (or check your Cost Explorer after a month of running the stack) and replace the estimate above with real figures. Screenshot Cost Explorer if the numbers are flattering — it's strong portfolio evidence.

> **📷 SCREENSHOT PLACEHOLDER**
> Paste a screenshot of AWS Cost Explorer or AWS Billing showing actual monthly cost for the CloudOps360 stack, if available.

---

## 17. Future Enhancements and Roadmap

1. CloudWatch metric-based idle detection — average CPU ≤ 10% over 7 or 14 days before flagging an instance.
2. S3 lifecycle and Intelligent-Tiering recommendations.
3. PDF and CSV report generation stored in S3 with pre-signed download URLs.
4. Amazon Cognito authentication for the dashboard.
5. Multi-account support using AssumeRole and AWS Organizations.
6. Integration with AWS Compute Optimizer and Trusted Advisor APIs.
7. Richer historical trend storage and optional Amazon QuickSight dashboards.
8. Automated remediation actions (with approval gates) for low-risk findings such as releasing unassociated Elastic IPs.

### 17.1 Suggested Additions (new)

- Automated unit tests (pytest + moto) for the rule-evaluation logic, addressing the coverage gap noted in Section 12.2.
- A DynamoDB GSI + Query-based access pattern to replace full-table Scans as history grows (Section 6.3 / 15.3).
- API authentication (Cognito or API keys) before any use beyond a personal portfolio account (Section 10.5).

> **⚠ GAP IDENTIFIED IN ORIGINAL DOCUMENT**
> The sub-list above is new and cross-references gaps identified elsewhere in this document, so future work is grounded in specific findings rather than a generic wish list.

---

## 18. Conclusion

CloudOps360 successfully delivers a working, end-to-end serverless platform that automates resource discovery, cost-leak detection, compliance visibility, and support-oriented troubleshooting guidance. The combination of scheduled scanning, structured findings, SNS alerting, and a React operations dashboard provides a realistic simulation of the tooling a Cloud Support team would value.

Beyond the running software, the project demonstrates a broad set of practical skills: Boto3 proficiency, IAM policy design, EventBridge scheduling, API Gateway configuration, DynamoDB data modeling, CloudWatch observability, CloudFormation basics, and frontend integration. These skills map directly to the expectations of Cloud Support, Cloud Operations, and Associate Cloud Engineer roles.

The platform is intentionally simple enough to explain and extend, yet complete enough to serve as a credible portfolio centerpiece. With the documented roadmap, it can evolve into a richer multi-account operations suite while remaining grounded in AWS Well-Architected principles.

---

## Appendix A — Environment Variables

| Lambda Function | Variable Name | Purpose |
|---|---|---|
| CloudOps360-Scanner | DYNAMODB_TABLE | Name of the scan-results DynamoDB table |
| CloudOps360-Scanner | SNS_TOPIC_ARN | ARN of the SNS topic used for email alerts |
| CloudOps360-Scanner | REPORT_BUCKET | S3 bucket reserved for future report artifacts |
| CloudOps360-API | DYNAMODB_TABLE | Same table — used for read operations |
| CloudOps360-API | SCANNER_FUNCTION_NAME | Name of the Scanner function to invoke on trigger |

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Add any environment variables missing from this list (e.g., AWS_REGION overrides, LOG_LEVEL, CORS_ALLOWED_ORIGIN) by checking each Lambda's Configuration → Environment variables tab.

## Appendix B — API Contract Summary

All successful responses use HTTP 200 and include standard CORS headers (`Access-Control-Allow-Origin: *`). Error responses use 4xx/5xx with a JSON body containing an error message.

**GET /compliance** — Returns lastScan, totalResources, compliant, nonCompliant, complianceRate, alerts[], resourcesByService{}, and trend[].

**GET /resources** — Returns `{ resources: [ { id, type, region, status, lastChecked, issue, remediation, severity } ] }`.

**GET /reports/latest** — Returns scanId, timestamp, total_issues, high_severity, findings_count, and optional history.

**POST /scan/trigger** — Returns `{ status: "queued", scanId: "...", message: "..." }` and asynchronously starts a new scan.

> **🔧 ACTION REQUIRED — FILL IN BY YOU**
> Replace each response shape with real captured JSON (see Section 9.2), and add example error responses (4xx/5xx) for completeness.

## Appendix C — Sample Finding Object

Each element inside the findings array stored in DynamoDB follows this conceptual structure:

```
resource_type : "EBS" | "EC2" | "ElasticIP" | "S3" | "DynamoDB" | "Config"
resource_id   : AWS resource identifier (e.g., vol-0123456789abcdef0)
issue         : Human-readable short description
severity      : "High" | "Medium" | "Low"
root_cause    : Brief explanation of why the issue exists
fix_steps     : Console steps and/or CLI commands for remediation
estimated_savings : Optional monetary or risk note
```

This consistent schema allows both the API layer and future report generators to process findings uniformly.

## Appendix D — Consolidated Screenshot Checklist

Every screenshot placeholder in this document, listed in one place for convenience:

1. Section 5.3 — Deployed architecture in AWS Console (CloudFormation Resources tab or Lambda trigger config)
2. Section 8.1 — Resources table view in the dashboard
3. Section 8.5 — SNS email alert (subject + body)
4. Section 8.6 — Full React dashboard home / compliance overview screen
5. Section 8.6 — Alerts panel sorted by severity
6. Section 8.6 — Resources table with an active filter applied
7. Section 9.2 — Successful API call in Postman/curl/DevTools (status 200 + JSON body)
8. Section 9.3 — Deployed frontend loading real (non-mock) data
9. Section 9.4 — CloudFormation stack Events tab (CREATE_COMPLETE)
10. Section 11.2 Step 3 — Terminal or console output showing successful stack deployment
11. Section 11.2 Step 4 — SNS subscription confirmation email or confirmed subscription
12. Section 11.2 Step 6 — Deployed frontend loading real (non-mock) data
13. Section 11.2 Step 7 — DynamoDB console showing a scan item with findings
14. Section 12.1 — Test EBS volume in EC2 console alongside matching dashboard finding
15. Section 12.1 — CloudWatch Logs Insights output confirming a successful Scanner run
16. Section 12.3 — CloudWatch Logs Insights query and results
17. Section 15.1 — Real path-matching error from CloudWatch Logs plus its fix
18. Section 16 — AWS Cost Explorer / Billing showing actual monthly cost

## Appendix E — Consolidated Action-Required (Fill-In) Checklist

Every orange "ACTION REQUIRED" box in this document, listed in one place for convenience:

1. Executive Summary — add real quantified results from a test scan
2. Section 6.2 — confirm real DynamoDB table name, key schema, attribute names
3. Section 6.3 — confirm Scan vs Query usage on read endpoints
4. Section 8.2 — document the real savings-estimation formula/constants
5. Section 8.3 — list the real AWS Config rule names evaluated
6. Section 8.4 — paste the real severity-assignment logic
7. Section 9.1 — paste a real representative function from the Scanner Lambda
8. Section 9.2 — paste real JSON responses for all four endpoints
9. Section 9.3 — confirm exact frontend dependency versions
10. Section 9.4 — paste the real CloudFormation Resources: section or repo link
11. Section 10.1 — paste the real Scanner Lambda IAM policy JSON
12. Section 11.1 — confirm minimum tool versions and any extra CLI tools used
13. Section 11.2 Step 2 — paste the real CloudFormation Parameters: block
14. Section 11.2 Step 5 — confirm real folder structure / packaging commands
15. Section 12.2 — document any automated tests added
16. Section 16 — replace cost estimate with real AWS Pricing Calculator or Cost Explorer figures
17. Appendix A — add any missing environment variables
18. Appendix B — replace response shapes with real captured JSON, add error responses

## Appendix F — Gaps Identified in the Source Document

For transparency, every red "GAP IDENTIFIED" callout in this document, listed together:

1. No quantified outcomes in the Executive Summary
2. Success criteria not explicitly mapped back to each project objective
3. No formal DynamoDB schema, key structure, or item shape defined anywhere
4. No list of specific AWS Config rules evaluated
5. No step-by-step deployment guide despite a documented CloudFormation template
6. No threat model or discussion of residual risk for accepted security tradeoffs
7. No test coverage summary or mention of automated testing
8. The Challenges table covered only 5 items; no systematic troubleshooting guide organized by component
9. No cost estimate, despite the project's stated purpose including cost optimization
10. Well-Architected Framework alignment was text-only with no visual summary
11. No forward-looking suggestions tied back to specific gaps found during this review

## Appendix G — Glossary

| Term | Definition |
|---|---|
| MTTR | Mean Time to Resolution — average time to resolve an incident from detection to fix |
| FinOps | Cloud financial operations — practices for managing and optimizing cloud cost |
| Least privilege | Granting only the minimum permissions required to perform a task |
| GSI | Global Secondary Index — an alternate query pattern on a DynamoDB table |
| Cold start | Latency incurred when a serverless function is invoked without a warm execution environment |
| IaC | Infrastructure as Code — defining cloud infrastructure declaratively (e.g., CloudFormation) |
| SNS | Simple Notification Service — AWS pub/sub messaging service used here for email alerts |
| Well-Architected Framework | AWS's set of six pillars (Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability) for evaluating cloud architectures |
