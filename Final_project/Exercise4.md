# Exercise 4: Final Verification and Architecture Review

## Overview

In this final exercise, you will perform a complete end-to-end verification of all automated monitoring and auditing resources, review the architecture that was deployed by your single CloudFormation template, and complete the final checklist.

**Estimated Duration:** 15 minutes

---

## Task 4.1 — Complete Resource Inventory Check

Navigate to each AWS service and confirm all resources exist:

**CloudFormation:**

| Check | Expected |
|---|---|
| Stack status | CREATE_COMPLETE |
| Total resources created | 35 resources |

**EC2:**
- Instance `{EnvironmentName}-MonitoredInstance` — State: **Running**
- Security group `{EnvironmentName}-SecurityGroup`
- VPC `{EnvironmentName}-VPC` with public subnet and Internet Gateway

**IAM:**
- Role `EC2MonitoringRole` — policies: CloudWatchAgentServerPolicy, SSMManagedInstanceCore, CloudWatchCustomMetrics (inline)
- Role `CloudTrailLogsRole` — inline policy: CloudTrailToCloudWatchLogs

**Systems Manager:**
- Parameter `/AmazonCloudWatch-{StackName}/config` — contains CloudWatch Agent JSON config

**SNS:**
- Topic `{StackName}-AlarmTopic` — Subscription: **Confirmed**

**CloudWatch — Alarms (10 total):**
- 5 EC2 monitoring alarms (CPU, NetworkIn, NetworkOut, Memory, Disk)
- 5 security alarms (ConsoleLogin, RootAccount, UnauthorizedAPI, SGChange, ResourceChange)

**CloudWatch — Log Groups:**
- `{StackName}-CloudTrailLogs` — Retention: 90 days — 5 metric filters attached

**CloudTrail:**
- Trail `{StackName}-Trail` — Status: Logging — Multi-region: Yes — Validation: Enabled

**S3:**
- Bucket `{StackName}-cloudtrail-{AccountId}` — Versioning: On — Lifecycle: Configured

---

## Task 4.2 — Review All CloudFormation Outputs

Go to **CloudFormation** → your stack → **Outputs** tab and record all values:

| Output Key | Description |
|---|---|
| `EC2InstanceId` | Instance ID of the monitored EC2 |
| `EC2PublicIP` | Public IP address |
| `SNSTopicArn` | ARN of the SNS alerting topic |
| `CloudTrailName` | Name of the CloudTrail trail |
| `CloudTrailS3Bucket` | S3 bucket storing audit logs |
| `CloudWatchDashboardURL` | Direct URL to the monitoring dashboard |
| `CloudTrailLogGroupName` | CloudWatch log group for CloudTrail |
| `CloudWatchAgentConfigParam` | SSM parameter with agent configuration |
| `VPCID` | VPC ID created for this lab |

---

## Task 4.3 — Architecture Review

The complete automated architecture deployed by a single CloudFormation template:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   Your Dedicated AWS Account                            │
│                                                                         │
│  ┌─────────────────────┐                                                │
│  │    EC2 Instance      │  ──── CloudWatch Agent ────►  CWAgent         │
│  │  (t3.micro, AL2)     │       (auto-installed via      namespace       │
│  │  IAM Role attached   │        UserData + SSM config)  mem_used_%     │
│  └──────────┬──────────┘                                disk_used_%     │
│             │ built-in metrics                               │           │
│             ▼                                               │           │
│  ┌──────────────────────────────────────────────────────────┘           │
│  │  CloudWatch Alarms (5)          CloudWatch Dashboard                 │
│  │  • HighCPU >= 80%               {StackName}-Dashboard                │
│  │  • HighNetworkIn >= 10MB        • CPU widget                         │
│  │  • HighNetworkOut >= 10MB       • Network widget                     │
│  │  • HighMemory >= 80%            • Memory widget                      │
│  │  • HighDisk >= 85%              • Disk widget                        │
│  └──────────────────┬─────────────────────────────────────┘            │
│                     │ alarm action                                       │
│                     ▼                                                    │
│           ┌──────────────────┐                                          │
│           │   SNS Topic       │ ──► Email Notification                  │
│           │ (alarm + security)│                                          │
│           └──────────────────┘                                          │
│                     ▲                                                    │
│  All API Calls       │                                                   │
│  in all Regions      │                                                   │
│       │              │                                                   │
│       ▼              │                                                   │
│  ┌────────────────────────────┐                                         │
│  │  CloudTrail Trail           │  IsMultiRegionTrail: true               │
│  │  {StackName}-Trail          │  EnableLogFileValidation: true          │
│  │  IncludeGlobalServices:true │  All management events                  │
│  └───────────┬────────────────┘                                         │
│              │                    │                                      │
│              ▼                    ▼                                      │
│  ┌──────────────────┐  ┌──────────────────────────────────────────────┐ │
│  │   S3 Bucket       │  │  CloudWatch Log Group                        │ │
│  │  (archive + hash) │  │  {StackName}-CloudTrailLogs (90-day retain)  │ │
│  │  Versioning ON    │  │                                              │ │
│  │  Lifecycle:30d IA │  │  Metric Filters (5):                         │ │
│  │  Expiry: 365d     │  │  • ConsoleLoginFilter    → ConsoleLoginCount │ │
│  └──────────────────┘  │  • RootLoginFilter       → RootAccountCount  │ │
│                         │  • UnauthorizedAPIFilter → UnauthorizedCount │ │
│                         │  • SGChangeFilter        → SGChangeCount     │ │
│                         │  • ResourceChangeFilter  → ResourceCount     │ │
│                         │              │                                │ │
│                         └──────────────┼────────────────────────────────┘ │
│                                        │                                │
│                                        ▼                                │
│                         CloudWatch Alarms (5 security):                 │
│                         • ConsoleLoginDetected  >= 1                    │
│                         • RootAccountUsage      >= 1 (1-min period)    │
│                         • UnauthorizedAPICalls  >= 5                   │
│                         • SecurityGroupChanged  >= 1                   │
│                         • ResourceCreatedOrDeleted >= 1                │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Task 4.4 — Summary of What Was Automated

| Requirement | Automated By | Status |
|---|---|---|
| CloudWatch Dashboard creation | CFT: MonitoringDashboard resource | ✅ |
| EC2 metrics monitoring (CPU, Network) | CFT: CPUUtilizationAlarm, NetworkInAlarm, NetworkOutAlarm | ✅ |
| CloudWatch Alarm based on CPU threshold | CFT: CPUUtilizationAlarm (param: CPUAlarmThreshold) | ✅ |
| SNS Topic creation | CFT: MonitoringSnsTopic | ✅ |
| SNS email subscription | CFT: SnsEmailSubscription (param: AlarmEmailAddress) | ✅ |
| Alarms attached to EC2 instance | CFT: Dimension InstanceId = MonitoredEC2Instance | ✅ |
| IAM Role for EC2 monitoring | CFT: EC2MonitoringRole | ✅ |
| CloudWatch Agent installation | CFT: UserData script in MonitoredEC2Instance | ✅ |
| Agent configuration via SSM | CFT: CloudWatchAgentConfig SSM Parameter | ✅ |
| Custom metrics (Memory, Disk) | CFT: Agent config + MemoryAlarm, DiskUsageAlarm | ✅ |
| Dashboard for custom metrics | CFT: MonitoringDashboard (includes CWAgent metrics) | ✅ |
| CloudTrail Trail creation | CFT: MonitoringTrail | ✅ |
| Multi-region logging | CFT: IsMultiRegionTrail: true | ✅ |
| S3 bucket for CloudTrail logs | CFT: CloudTrailBucket + BucketPolicy | ✅ |
| Log file validation | CFT: EnableLogFileValidation: true | ✅ |
| CloudTrail → CloudWatch integration | CFT: CloudWatchLogsLogGroupArn + RoleArn | ✅ |
| Log retention configuration | CFT: CloudTrailLogGroup RetentionInDays (param) | ✅ |
| Console login alerts | CFT: ConsoleLoginFilter + ConsoleLoginAlarm | ✅ |
| Root user login alerts | CFT: RootLoginFilter + RootLoginAlarm | ✅ |
| Unauthorized API call alerts | CFT: UnauthorizedAPIFilter + UnauthorizedAPIAlarm | ✅ |
| Security group change alerts | CFT: SecurityGroupChangeFilter + SGChangeAlarm | ✅ |
| Resource creation/deletion alerts | CFT: ResourceChangeFilter + ResourceChangeAlarm | ✅ |
| CloudTrail metric filters | CFT: 5 x AWS::Logs::MetricFilter resources | ✅ |
| CloudWatch alarms on CloudTrail events | CFT: 5 security CloudWatch Alarms | ✅ |

---

## Final Validation Checklist

**CloudWatch Automation:**
- [ ] IAM Role `EC2MonitoringRole` created with correct policies
- [ ] SSM Parameter with CW Agent config exists
- [ ] EC2 instance deployed with CloudWatch Agent running
- [ ] SNS email subscription confirmed
- [ ] 5 monitoring alarms active (CPU, Network x2, Memory, Disk)
- [ ] Namespace `CWAgent` collecting Memory and Disk custom metrics
- [ ] Dashboard `{StackName}-Dashboard` shows all 5 metric widgets
- [ ] Stress test triggered CPU alarm and email received

**CloudTrail Automation:**
- [ ] Trail `{StackName}-Trail` is in Logging status
- [ ] Multi-region = Yes, Log file validation = Enabled
- [ ] S3 bucket receiving `.json.gz` log files
- [ ] S3 lifecycle rule: 30-day IA transition, 365-day expiry
- [ ] CloudWatch Log Group has 90-day retention
- [ ] IAM Role `CloudTrailLogsRole` has correct log delivery permissions

**Alert Automation:**
- [ ] 5 metric filters on log group in namespace `CloudTrailMetrics`
- [ ] 10 total alarms visible (5 monitoring + 5 security)
- [ ] Console login alert triggered and email received
- [ ] Security group change alert triggered and email received
- [ ] All 5 Logs Insights queries executed and saved

---

## Congratulations!

You have successfully verified the complete automation of AWS monitoring and auditing. A single CloudFormation template deployed and configured **35 AWS resources** automatically, implementing:

- Real-time EC2 performance monitoring with custom Memory and Disk metrics
- Multi-region CloudTrail auditing with log integrity validation
- Automated security alerting for 5 critical event types
- CloudWatch Logs Insights for advanced log analysis

---
