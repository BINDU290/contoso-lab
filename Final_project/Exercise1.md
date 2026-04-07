# Exercise 1: CloudWatch Automation — Monitoring Setup

## Overview

In this exercise, you will verify all CloudWatch resources that were automatically deployed by the CloudFormation stack when your lab launched. The stack has already provisioned the EC2 instance, IAM roles, CloudWatch Agent, SNS topic, alarms, and dashboard — with zero manual configuration.

**Estimated Duration:** 15 minutes

---

## What Was Auto-Deployed (CloudWatch)

| Resource | Name in Your Environment | Purpose |
|---|---|---|
| IAM Role | `monitoringlab-MonitoredInstance` | EC2 role with CloudWatchAgentServerPolicy + SSMManagedInstanceCore |
| SSM Parameter | `/AmazonCloudWatch-{StackName}/config` | CloudWatch Agent configuration stored automatically |
| EC2 Instance | `monitoringlab-MonitoredInstance` | Amazon Linux 2, t3.micro with CW Agent auto-started |
| SNS Topic | `{StackName}-AlarmTopic` | Sends email alerts for all alarms |
| Alarm | `{StackName}-HighCPUUtilization` | Triggers when CPU >= 80% |
| Alarm | `{StackName}-HighNetworkIn` | Triggers when NetworkIn >= 10MB |
| Alarm | `{StackName}-HighNetworkOut` | Triggers when NetworkOut >= 10MB |
| Alarm | `{StackName}-HighMemoryUsage` | Triggers when Memory >= 80% (custom metric) |
| Alarm | `{StackName}-HighDiskUsage` | Triggers when Disk >= 85% (custom metric) |
| Dashboard | `{StackName}-Dashboard` | Visual monitoring for all EC2 and custom metrics |

> **Note:** `{StackName}` is the name of your CloudFormation stack — visible in the CloudFormation console.

---

## Task 1.1 — Confirm Stack Deployment

1. Sign in to the **AWS Console** using credentials from your **Environment Details** panel.

2. Navigate to **CloudFormation** → **Stacks**.

3. Find the stack — confirm its status is **CREATE_COMPLETE**.

4. Click the stack → click the **Outputs** tab.

5. Record these values — you will use them throughout the lab:

   | Output Key | What It Contains |
   |---|---|
   | `EC2InstanceId` | Instance ID of your monitored EC2 |
   | `EC2PublicIP` | Public IP of the EC2 instance |
   | `SNSTopicArn` | ARN of the SNS alarm topic |
   | `CloudWatchDashboardURL` | Direct URL to your dashboard |
   | `CloudWatchAgentConfigParam` | SSM parameter name holding CW Agent config |

---

## Task 1.2 — Confirm SNS Email Subscription

1. Check the inbox of the email address configured in your lab parameters.

2. Find the email with subject: **AWS Notification - Subscription Confirmation**.

3. Click **Confirm subscription**.

> **Important:** If you do not confirm, you will receive no alarm emails throughout this lab.

---

## Task 1.3 — Verify the CloudWatch Dashboard

1. Click the **CloudWatchDashboardURL** from the Outputs tab, or navigate to:
   **CloudWatch** → **Dashboards** → click **`{StackName}-Dashboard`**

2. Verify the following widgets are present:

   | Widget | Metric Source | Namespace |
   |---|---|---|
   | CPU Utilization | EC2 built-in metric | `AWS/EC2` |
   | Network In | EC2 built-in metric | `AWS/EC2` |
   | Network Out | EC2 built-in metric | `AWS/EC2` |
   | Memory Used % | CloudWatch Agent custom | `CWAgent` |
   | Disk Used % | CloudWatch Agent custom | `CWAgent` |

> **Note:** Memory and Disk widgets may show "Insufficient Data" for the first 3–5 minutes while the CloudWatch Agent warms up.

---

## Task 1.4 — Verify CloudWatch Alarms

1. Navigate to **CloudWatch** → **Alarms** → **All alarms**.

2. Confirm all 5 EC2 monitoring alarms exist:

   | Alarm Name | Metric | Namespace | Threshold |
   |---|---|---|---|
   | `{StackName}-HighCPUUtilization` | CPUUtilization | AWS/EC2 | >= 80% |
   | `{StackName}-HighNetworkIn` | NetworkIn | AWS/EC2 | >= 10,000,000 bytes |
   | `{StackName}-HighNetworkOut` | NetworkOut | AWS/EC2 | >= 10,000,000 bytes |
   | `{StackName}-HighMemoryUsage` | mem_used_percent | CWAgent | >= 80% |
   | `{StackName}-HighDiskUsage` | disk_used_percent | CWAgent | >= 85% |

3. Click **`{StackName}-HighCPUUtilization`** → review:
   - **Actions** section shows the SNS topic ARN
   - **Details** shows evaluation period: 2 consecutive periods of 5 minutes

---

## Task 1.5 — Verify IAM Role and CloudWatch Agent Config

**Verify the IAM Role:**

1. Navigate to **IAM** → **Roles** → search for `EC2MonitoringRole` or `monitoringlab`.

2. Click the role → confirm these policies are attached:
   - `CloudWatchAgentServerPolicy` (managed)
   - `AmazonSSMManagedInstanceCore` (managed)
   - `CloudWatchCustomMetrics` (inline — allows PutMetricData)

**Verify the SSM Parameter (CloudWatch Agent Config):**

1. Navigate to **Systems Manager** → **Parameter Store**.

2. Find the parameter: `/AmazonCloudWatch-{StackName}/config`

3. Click it → click **Value** tab → you will see the JSON configuration that tells the agent to collect:
   - `mem_used_percent`, `mem_available`, `mem_total` every 60 seconds
   - `disk_used_percent`, `inodes_free` on the root volume every 60 seconds

**Verify the Agent is Running on EC2:**

1. Navigate to **EC2** → **Instances** → select your instance → **Connect** → **EC2 Instance Connect** → **Connect**.

2. Run:
   ```bash
   sudo systemctl status amazon-cloudwatch-agent
   ```
   Expected: `active (running)`

3. Check the setup log created by UserData:
   ```bash
   cat /var/log/setup.log
   ```
   Expected output: `CloudWatch Agent setup complete`

4. Verify the agent config was fetched from SSM:
   ```bash
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status
   ```
   Expected: `"status": "running"`

---

## Task 1.6 — Verify Custom Metrics (Memory and Disk)

1. Navigate to **CloudWatch** → **Metrics** → **All metrics**.

2. Find namespace **CWAgent**.

3. Click **CWAgent** → **ImageId, InstanceId, InstanceType, device, fstype, path**.

4. Confirm these metrics are listed:
   - `disk_used_percent`
   - `mem_used_percent`
   - `mem_available`

5. Check the checkbox next to `mem_used_percent` → click **Add to graph** → confirm data points appear.

---

## Task 1.7 — Trigger a CPU Alarm (Stress Test)

1. Connect to your EC2 instance via **EC2 Instance Connect** (as in Task 1.5).

2. Run the stress tool (pre-installed by the stack):
   ```bash
   stress --cpu 2 --timeout 360
   ```

3. Leave this running and wait **5–6 minutes**.

4. In a new browser tab, go to **CloudWatch** → **Alarms** → **`{StackName}-HighCPUUtilization`**.

5. The alarm state changes from **OK** → **In alarm** (shown in red).

6. Check your email — you should receive:
   > Subject: **ALARM: "{StackName}-HighCPUUtilization" in US East**

7. Return to the EC2 terminal → press **Ctrl+C** to stop the stress test.

8. Within 5–10 minutes the alarm returns to **OK** and you receive another email:
   > Subject: **OK: "{StackName}-HighCPUUtilization"**

---

## Validation Checkpoint 1

Before moving to Exercise 2, confirm all of the following:

- [ ] CloudFormation stack status is **CREATE_COMPLETE**
- [ ] SNS email subscription confirmed
- [ ] Dashboard `{StackName}-Dashboard` shows all 5 metric widgets
- [ ] All 5 CloudWatch alarms are visible
- [ ] IAM Role has `CloudWatchAgentServerPolicy` and `CloudWatchCustomMetrics` policies
- [ ] SSM Parameter `/AmazonCloudWatch-{StackName}/config` exists with agent configuration
- [ ] CloudWatch Agent status is `active (running)` on the EC2
- [ ] `CWAgent` namespace shows `mem_used_percent` and `disk_used_percent` metrics
- [ ] Stress test triggered `{StackName}-HighCPUUtilization` alarm
- [ ] Received alarm email notification

---

**Proceed to Exercise 2 →**
