# Exercise 3: Monitoring and Alert Automation — Security Events

## Overview

In this exercise, you will verify the 5 CloudWatch metric filters and 5 security alarms that were automatically created by the stack. These monitor CloudTrail logs in real time and trigger instant SNS alerts for critical security events. You will test each alert type and use CloudWatch Logs Insights to query audit logs.

**Estimated Duration:** 25 minutes

---

## What Was Auto-Deployed (Security Alerts)

| Filter Name | Pattern Detected | Metric | Alarm |
|---|---|---|---|
| `{StackName}-ConsoleLoginFilter` | `eventName = ConsoleLogin` | ConsoleLoginCount | `{StackName}-ConsoleLoginDetected` |
| `{StackName}-RootLoginFilter` | `userIdentity.type = Root` | RootAccountUsageCount | `{StackName}-RootAccountUsage` |
| `{StackName}-UnauthorizedAPIFilter` | `errorCode = AccessDenied/UnauthorizedAccess/AuthFailure` | UnauthorizedAPICallCount | `{StackName}-UnauthorizedAPICalls` |
| `{StackName}-SecurityGroupChangeFilter` | SG create/modify/delete events | SecurityGroupChangeCount | `{StackName}-SecurityGroupChanged` |
| `{StackName}-ResourceChangeFilter` | EC2/S3/CFT create or delete events | ResourceChangeCount | `{StackName}-ResourceCreatedOrDeleted` |

All metrics are published to namespace: **`CloudTrailMetrics`**

---

## Task 3.1 — Verify Metric Filters

1. Navigate to **CloudWatch** → **Log groups** → click **`{StackName}-CloudTrailLogs`**.

2. Click the **Metric filters** tab.

3. Confirm all 5 filters exist:

   | Filter Name | Metric Namespace | Metric Name |
   |---|---|---|
   | `{StackName}-ConsoleLoginFilter` | CloudTrailMetrics | ConsoleLoginCount |
   | `{StackName}-RootLoginFilter` | CloudTrailMetrics | RootAccountUsageCount |
   | `{StackName}-UnauthorizedAPIFilter` | CloudTrailMetrics | UnauthorizedAPICallCount |
   | `{StackName}-SecurityGroupChangeFilter` | CloudTrailMetrics | SecurityGroupChangeCount |
   | `{StackName}-ResourceChangeFilter` | CloudTrailMetrics | ResourceChangeCount |

4. Click **`{StackName}-ConsoleLoginFilter`** → review the filter pattern:
   ```
   { ($.eventName = "ConsoleLogin") }
   ```

5. Click **`{StackName}-SecurityGroupChangeFilter`** → review the pattern:
   ```
   { ($.eventName = "AuthorizeSecurityGroupIngress") ||
     ($.eventName = "AuthorizeSecurityGroupEgress") ||
     ($.eventName = "RevokeSecurityGroupIngress") ||
     ($.eventName = "RevokeSecurityGroupEgress") ||
     ($.eventName = "CreateSecurityGroup") ||
     ($.eventName = "DeleteSecurityGroup") }
   ```

---

## Task 3.2 — Verify Security Alarms

1. Navigate to **CloudWatch** → **Alarms** → **All alarms**.

2. You should now see **10 total alarms** — 5 from Exercise 1 (EC2 monitoring) + 5 security alarms:

   | Alarm Name | Metric | Threshold |
   |---|---|---|
   | `{StackName}-ConsoleLoginDetected` | ConsoleLoginCount | >= 1 |
   | `{StackName}-RootAccountUsage` | RootAccountUsageCount | >= 1 (1-min period) |
   | `{StackName}-UnauthorizedAPICalls` | UnauthorizedAPICallCount | >= 5 |
   | `{StackName}-SecurityGroupChanged` | SecurityGroupChangeCount | >= 1 |
   | `{StackName}-ResourceCreatedOrDeleted` | ResourceChangeCount | >= 1 |

3. Click **`{StackName}-RootAccountUsage`** → note its evaluation period is **1 minute** (fastest response for the most critical event).

---

## Task 3.3 — Test Console Login Alert

1. Open a **new browser tab** and sign in to the AWS Console again using your lab credentials.

2. Wait approximately **5 minutes** for CloudTrail to record and process the event.

3. Navigate to **CloudWatch** → **Alarms** → **`{StackName}-ConsoleLoginDetected`**.

4. The alarm transitions from **OK** → **In alarm**.

5. Check your email for:
   > Subject: **ALARM: "{StackName}-ConsoleLoginDetected"**

---

## Task 3.4 — Test Security Group Change Alert

1. Navigate to **EC2** → **Security Groups** → find **`{EnvironmentName}-SecurityGroup`** (the one tagged with your lab environment name).

2. Click **Edit inbound rules** → **Add rule**:
   - Type: `Custom TCP`
   - Port: `9999`
   - Source: `Anywhere (0.0.0.0/0)`

3. Click **Save rules**.

4. Wait approximately **5 minutes**.

5. Go to **CloudWatch** → **Alarms** → **`{StackName}-SecurityGroupChanged`** → state changes to **In alarm**.

6. Check your email for the security group change notification.

7. Go back and remove the test rule (port 9999) to restore the original state.

---

## Task 3.5 — CloudWatch Logs Insights Queries

1. Navigate to **CloudWatch** → **Logs Insights**.

2. In the **Select log group(s)** dropdown, select: **`{StackName}-CloudTrailLogs`**

3. Run each query below — click **Run query** after each:

**Query 1 — Find all Console Logins:**
```
fields @timestamp, userIdentity.arn, sourceIPAddress, responseElements.ConsoleLogin
| filter @message like /ConsoleLogin/
| sort @timestamp desc
| limit 20
```

**Query 2 — Find all Unauthorized API Calls:**
```
fields @timestamp, userIdentity.arn, eventName, errorCode, sourceIPAddress, awsRegion
| filter errorCode in ["AccessDenied", "UnauthorizedAccess", "AuthFailure"]
| sort @timestamp desc
| limit 50
```

**Query 3 — Find Security Group Changes:**
```
fields @timestamp, userIdentity.arn, eventName, awsRegion, sourceIPAddress
| filter eventName in [
    "AuthorizeSecurityGroupIngress",
    "RevokeSecurityGroupIngress",
    "AuthorizeSecurityGroupEgress",
    "RevokeSecurityGroupEgress",
    "CreateSecurityGroup",
    "DeleteSecurityGroup"
  ]
| sort @timestamp desc
| limit 20
```

**Query 4 — Find Resource Creation and Deletion Events:**
```
fields @timestamp, userIdentity.arn, eventName, awsRegion, sourceIPAddress
| filter eventName in [
    "RunInstances", "TerminateInstances",
    "CreateBucket", "DeleteBucket",
    "CreateStack", "DeleteStack"
  ]
| sort @timestamp desc
| limit 20
```

**Query 5 — Count events by type in the last hour:**
```
fields eventName
| stats count() as EventCount by eventName
| sort EventCount desc
| limit 20
```

4. For each query, click **Save** → enter a meaningful name → click **Save query**.

---

## Task 3.6 — Verify Security Metrics in CloudWatch

1. Navigate to **CloudWatch** → **Metrics** → **All metrics**.

2. Find namespace **CloudTrailMetrics**.

3. Click it → confirm all 5 security metrics are listed:
   - `ConsoleLoginCount`
   - `RootAccountUsageCount`
   - `UnauthorizedAPICallCount`
   - `SecurityGroupChangeCount`
   - `ResourceChangeCount`

4. Select `ConsoleLoginCount` → click **Add to graph** → confirm you see a data point from your Task 3.3 test.

---

## Validation Checkpoint 3

- [ ] 5 metric filters visible on log group `{StackName}-CloudTrailLogs`
- [ ] 10 total alarms visible (5 monitoring + 5 security)
- [ ] Namespace **CloudTrailMetrics** contains all 5 security metrics
- [ ] Console login test triggered `{StackName}-ConsoleLoginDetected` alarm
- [ ] Security group change test triggered `{StackName}-SecurityGroupChanged` alarm
- [ ] Received email notifications for both alarms
- [ ] All 5 Logs Insights queries executed successfully
- [ ] Queries saved in CloudWatch Logs Insights

---

**Proceed to Exercise 4 →**
