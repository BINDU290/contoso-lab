# Exercise 2: CloudTrail Automation — Audit Logging

## Overview

In this exercise, you will verify the CloudTrail configuration that was automatically deployed by the stack. CloudTrail is already logging all API activity across all regions, storing logs in S3, and streaming them in real-time to CloudWatch Logs — all set up automatically by CloudFormation.

**Estimated Duration:** 20 minutes

---

## What Was Auto-Deployed (CloudTrail)

| Resource | Name | Configuration |
|---|---|---|
| CloudTrail Trail | `{StackName}-Trail` | Multi-region=true, Log validation=true, Global services=true |
| S3 Bucket | `{StackName}-cloudtrail-{AccountId}` | Versioning ON, lifecycle: 30-day IA, 365-day expiry |
| S3 Bucket Policy | Auto-applied | Allows CloudTrail GetBucketAcl + PutObject |
| CloudWatch Log Group | `{StackName}-CloudTrailLogs` | 90-day retention |
| IAM Role | `CloudTrailLogsRole` | Inline policy: logs:CreateLogStream + logs:PutLogEvents |

---

## Task 2.1 — Verify CloudTrail Trail

1. Navigate to **CloudTrail** → **Trails**.

2. Click **`{StackName}-Trail`**.

3. Verify all settings:

   | Setting | Expected Value |
   |---|---|
   | Trail status | **Logging** (green indicator) |
   | Multi-region trail | **Yes** |
   | Log file validation | **Enabled** |
   | Include global service events | **Yes** |
   | Management events | **All** (Read + Write) |
   | S3 bucket | `{StackName}-cloudtrail-{AccountId}` |
   | CloudWatch Logs log group | `{StackName}-CloudTrailLogs` |

4. Note the **Trail ARN** from the CloudFormation Outputs tab → confirm it matches.

---

## Task 2.2 — Verify S3 Log Bucket

1. Navigate to **S3** → find the bucket whose name starts with your stack name and contains `cloudtrail`.

2. Also check CloudFormation Outputs → **CloudTrailS3Bucket** for the exact bucket name.

3. Open the bucket → navigate:
   ```
   AWSLogs/
   └── {AccountId}/
       └── CloudTrail/
           └── us-east-1/
               └── {Year}/{Month}/{Day}/
   ```

4. After 5–15 minutes you will see `.json.gz` log files here.

5. Download one → open it (use 7-Zip or `gunzip` on Linux) → it contains JSON records of every API call made.

**Verify Lifecycle Rules (cost optimization):**

1. In the S3 bucket → click **Management** tab → **Lifecycle rules**.

2. Confirm the rule `ArchiveLogs`:
   - Transition to Standard-IA after **30 days**
   - Expiration after **365 days**

---

## Task 2.3 — Verify CloudWatch Logs Integration

1. Navigate to **CloudWatch** → **Log groups**.

2. Find the log group named `{StackName}-CloudTrailLogs` (also shown in Outputs: **CloudTrailLogGroupName**).

3. Click it → confirm:
   - **Retention**: 90 days (set by the `LogRetentionDays` parameter)
   - **Log streams**: One or more streams appearing (named by region + account)

4. Click a log stream → you will see raw CloudTrail event records being streamed in real time.

5. Expand one record → identify:
   - `eventName` — what API call was made
   - `userIdentity` — who made it
   - `sourceIPAddress` — from where
   - `awsRegion` — in which region
   - `eventTime` — when

---

## Task 2.4 — Verify Log File Validation

1. In **CloudTrail** → **Trails** → click your trail.

2. Confirm **Log file validation**: **Enabled**.

3. Navigate to your S3 bucket → look for a folder named **CloudTrail-Digest**.

4. This folder contains hash digest files that prove log integrity — if any log file is tampered with, the digest validation will catch it.

> **Why this matters:** Log file validation is a compliance requirement. It ensures that CloudTrail logs are tamper-proof and can be trusted as evidence of activity.

---

## Task 2.5 — Browse CloudTrail Event History

1. Navigate to **CloudTrail** → **Event history**.

2. Review the list of recent API calls in your account.

3. Use the **Lookup attributes** filter:
   - Set **Event name** = `RunInstances`
   - Find the event where CloudFormation launched your EC2 instance
   - Click it → review the full JSON → note the `userIdentity` shows `cloudformation.amazonaws.com` as the invoker

4. Try another filter:
   - Set **Event name** = `CreateTrail`
   - Find the event that created your CloudTrail trail
   - This proves CloudTrail captures its own creation event

---

## Task 2.6 — Verify IAM Role for Log Delivery

1. Navigate to **IAM** → **Roles** → search for `CloudTrailLogsRole`.

2. Click the role → click **Permissions** tab.

3. Expand the inline policy `CloudTrailToCloudWatchLogs`.

4. Confirm the policy allows:
   - `logs:CreateLogStream`
   - `logs:PutLogEvents`
   - On the resource: your CloudTrail log group ARN

---

## Validation Checkpoint 2

- [ ] Trail `{StackName}-Trail` shows **Logging** status
- [ ] Multi-region trail and log file validation are both **Enabled**
- [ ] S3 bucket exists with `AWSLogs/{AccountId}/CloudTrail/` folder structure
- [ ] S3 lifecycle rule configured: 30-day IA transition, 365-day expiry
- [ ] CloudWatch Log Group has **90-day retention**
- [ ] Log streams are populating in CloudWatch
- [ ] Digest folder `CloudTrail-Digest` exists in S3
- [ ] CloudTrail Event History shows recent API calls
- [ ] IAM Role has `logs:CreateLogStream` and `logs:PutLogEvents` permissions

---

**Proceed to Exercise 3 →**
