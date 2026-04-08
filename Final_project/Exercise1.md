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

<img width="1920" height="449" alt="Screenshot (230)" src="https://github.com/user-attachments/assets/036c0484-73ec-4162-8da2-75e264ba15fb" />

4. Click the stack → click the **Outputs** tab.

5. Record these values — you will use them throughout the lab:

   | Output Key | What It Contains |
   |---|---|
   | `EC2InstanceId` | Instance ID of your monitored EC2 |
   | `EC2PublicIP` | Public IP of the EC2 instance |
   | `SNSTopicArn` | ARN of the SNS alarm topic |
   | `CloudWatchDashboardURL` | Direct URL to your dashboard |
   | `CloudWatchAgentConfigParam` | SSM parameter name holding CW Agent config |

<img width="1920" height="886" alt="Screenshot (231)" src="https://github.com/user-attachments/assets/d5d33a71-214e-456b-9085-2e31a1ca5b76" />

---

## Task 1.2 — Confirm SNS Email Subscription

1. Check the inbox of the email address configured in your lab parameters.

2. Find the email with subject: **AWS Notification - Subscription Confirmation**.

3. Click **Confirm subscription**.

<img width="1920" height="487" alt="Screenshot (232)" src="https://github.com/user-attachments/assets/390fc2f6-cc14-4953-894e-37c63d9e8202" />

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

   <img width="1920" height="884" alt="Screenshot (244)" src="https://github.com/user-attachments/assets/70e983b8-2053-471d-a237-ae744e8e6c2b" />

> **Note:** Memory and Disk widgets may show "Insufficient Data" for the first 3–5 minutes while the CloudWatch Agent warms up.

---

## Task 1.4 — Verify CloudWatch Alarms

1. Navigate to **CloudWatch** → **Alarms** → **All alarms**.

<img width="1920" height="844" alt="Screenshot (233)" src="https://github.com/user-attachments/assets/730443b8-7671-4edd-9645-8d9d21ba4900" />

2. Confirm all alarms exist:

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
   
   <img width="1920" height="857" alt="Screenshot (245)" src="https://github.com/user-attachments/assets/ad79929f-5b3a-4117-adaf-b04997760c07" />

<img width="1920" height="851" alt="Screenshot (235)" src="https://github.com/user-attachments/assets/86837256-8f5f-4282-86f9-9b4baf7a4ac7" />

---

## Task 1.5 — Verify IAM Role and CloudWatch Agent Config

**Verify the IAM Role:**

1. Navigate to **IAM** → **Roles** → search for `EC2MonitoringRole` or `monitoringlab`.

<img width="1920" height="857" alt="Screenshot (236)" src="https://github.com/user-attachments/assets/cfff4289-93ee-4f1d-94c0-73f6eb50343d" />

2. Click the role → confirm these policies are attached:
   - `CloudWatchAgentServerPolicy` (managed)
   - `AmazonSSMManagedInstanceCore` (managed)
   - `CloudWatchCustomMetrics` (inline — allows PutMetricData)

**Verify the SSM Parameter (CloudWatch Agent Config):**

1. Navigate to **Systems Manager** → **Parameter Store**.

2. Find the parameter: `/AmazonCloudWatch-{StackName}/config`

<img width="1920" height="735" alt="Screenshot (237)" src="https://github.com/user-attachments/assets/e62df9e8-f9e5-452c-8ee9-ea805fd3a1b6" />

3. Click it → click **Value** tab → you will see the JSON configuration that tells the agent to collect:
   - `mem_used_percent`, `mem_available`, `mem_total` every 60 seconds
   - `disk_used_percent`, `inodes_free` on the root volume every 60 seconds

<img width="1920" height="851" alt="Screenshot (238)" src="https://github.com/user-attachments/assets/84bfaacf-4c22-45c6-ac7f-8067cb7466c6" />

**Verify the Agent is Running on EC2:**

1. Navigate to **EC2** → **Instances** → select your instance → **Connect** → **EC2 Instance Connect** → **Connect**.


<img width="1920" height="847" alt="Screenshot (239)" src="https://github.com/user-attachments/assets/c92926e0-66d8-4ec0-9367-1a19bc50f378" />

2. Run:
   ```bash
   sudo systemctl status amazon-cloudwatch-agent
   ```
   Expected: `active (running)`

3. Verify the agent config was fetched from SSM:
   ```bash
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status
   ```
   Expected: `"status": "running"`

<img width="1920" height="899" alt="Screenshot (240)" src="https://github.com/user-attachments/assets/c46dc205-ff54-482e-88d5-ae8c31303768" />

---

## Task 1.6 — Verify Custom Metrics (Memory and Disk)

1. Navigate to **CloudWatch** → **Metrics** → **All metrics**.

2. Find namespace **CWAgent**.

<img width="1920" height="905" alt="Screenshot (241)" src="https://github.com/user-attachments/assets/60f0b1f2-f5dc-454d-9cb0-887b23ee0c63" />

3. Click **CWAgent** → **ImageId, InstanceId, InstanceType, device, fstype, path**.

4. Confirm the metrics are listed:

<img width="1920" height="860" alt="Screenshot (243)" src="https://github.com/user-attachments/assets/574d9194-d972-4968-b2a1-d658afe9b18c" />

5. Check the checkbox next to `mem_used_percent` → click **Add to graph** → confirm data points appear.

---

## Task 1.7 — Trigger a CPU Alarm (Stress Test)

1. Connect to your EC2 instance via **EC2 Instance Connect** (as in Task 1.5).

2. Run the stress tool (pre-installed by the stack):
   ```bash
   stress --cpu 2 --timeout 360
   ```
   <img width="1920" height="419" alt="Screenshot (246)" src="https://github.com/user-attachments/assets/142e877d-0bfc-435b-bda9-ca9dedaea822" />

3. Leave this running and wait **5–6 minutes**.

4. In a new browser tab, go to **CloudWatch** → **Alarms** → **`{StackName}-HighCPUUtilization`**.

5. The alarm state changes from **OK** → **In alarm** (shown in red).
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6d2093a9-9fd2-46e8-afc1-6efafeedfc4d" />

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
