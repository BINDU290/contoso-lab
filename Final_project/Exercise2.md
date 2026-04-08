# Exercise 2: CloudTrail Automation — Audit Logging

## Overview

In this exercise, you will verify the CloudTrail configuration that was automatically deployed by the stack. CloudTrail is already logging all API activity across all regions, storing logs in S3, and streaming them in real-time to CloudWatch Logs — all set up automatically by CloudFormation.

**Estimated Duration:** 15 minutes

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
   | CloudWatch Logs log group | `{StackName}-CloudTrailLogs` |
   <img width="1920" height="832" alt="Screenshot (248)" src="https://github.com/user-attachments/assets/5cc2316c-8ee3-4a28-9df2-2d3f47237313" />

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
   <img width="1920" height="849" alt="Screenshot (249)" src="https://github.com/user-attachments/assets/4a37e6aa-a178-4071-aef7-e4a1409a9204" />

4. After 5–15 minutes you will see `.json.gz` log files here.

5. It contains JSON records of every API call made.

**Verify Lifecycle Rules (cost optimization):**

1. In the S3 bucket → click **Management** tab → **Lifecycle rules**.

2. Confirm the rule `DeleteOldLogs` or `ArchiveLogs`:

<img width="1920" height="855" alt="Screenshot (250)" src="https://github.com/user-attachments/assets/8c426daa-6287-41c0-8682-70962ef6421b" />

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
   
<img width="1920" height="860" alt="Screenshot (251)" src="https://github.com/user-attachments/assets/6d6329c3-c25d-44d5-88ea-512c5ac73163" />

---

## Task 2.4 — Verify Log File Validation

1. In **CloudTrail** → **Trails** → click your trail.

2. Confirm **Log file validation**: **Enabled**.

<img width="1920" height="832" alt="Screenshot (248)" src="https://github.com/user-attachments/assets/0184fda8-132f-4fbd-8ff8-d1d2ae65887a" />

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
   <img width="1920" height="880" alt="Screenshot (252)" src="https://github.com/user-attachments/assets/75a9ad58-fe02-46a4-b3a2-e745293cabe7" />

   <img width="1920" height="891" alt="Screenshot (253)" src="https://github.com/user-attachments/assets/434b508d-e346-4f00-a73d-ae325b89ebd3" />

4. Try another filter:
   - Set **Event name** = `CreateTrail`
   - Find the event that created your CloudTrail trail
   - This proves CloudTrail captures its own creation event
   
<img width="1920" height="851" alt="Screenshot (254)" src="https://github.com/user-attachments/assets/3f33b531-130f-42ea-a946-fef8a3ee6f47" />

<img width="1920" height="884" alt="Screenshot (255)" src="https://github.com/user-attachments/assets/aa860443-f4f0-430a-aff4-4a68b71eef5b" />

---

## Task 2.6 — Verify IAM Role for Log Delivery

1. Navigate to **IAM** → **Roles** → search for `CloudTrailLogsRole`.

2. Click the role → click **Permissions** tab.

3. Expand the inline policy `CloudTrailToCloudWatchLogs`.

<img width="1920" height="857" alt="Screenshot (256)" src="https://github.com/user-attachments/assets/11dd2b0a-97a0-4002-a415-66d68ff356be" />

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
