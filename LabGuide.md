# Contoso Certification Academy — AWS Practice Lab Guide

**Lab Duration:** 8 Hours  
**Retake Policy:** Up to 3 attempts  
**Lab Environment:** GUI-based Linux Jump VM (browser RDP)  
**Cloud Platform:** Amazon Web Services (AWS)

---

## Overview

Welcome to the **Contoso AWS Certification Practice Lab**! In this self-paced lab, you will get hands-on experience with core AWS services used in real-world cloud deployments.

By the end of this lab, you will have:
- Created and configured an **Amazon S3 storage bucket**
- Deployed a **small EC2 virtual machine**
- Configured a **Virtual Private Cloud (VPC)** with subnets and routing
- Connected to a **GUI-based Linux Jump VM** via browser RDP

---

## Prerequisites

- A modern web browser (Chrome or Edge recommended)
- Your lab registration email and credentials (provided after launch)
- No local software installation required — everything runs inside the Jump VM

---

## Accessing Your Lab Environment

1. After clicking **Launch Lab**, wait approximately **15 minutes** for your dedicated AWS environment to be provisioned.
2. Once deployment completes, your screen will show:
   - **AWS Console URL**
   - **AWS Account ID**
   - **IAM Username**
   - **IAM Password**
   - **Jump VM Connect** button
3. Click the **Connect** button next to the Jump VM to open the Linux desktop in your browser.
4. Log in using the credentials shown in the **Environment Details** panel.

> **Note:** Do not close the lab tab — your session timer runs continuously from the moment you launch.

---

## Exercise 1: Create an Amazon S3 Storage Bucket

### Task 1.1 — Navigate to S3

1. Inside the Jump VM browser, open a new tab and go to the **AWS Console**:  
   `https://console.aws.amazon.com`
2. Sign in using the **IAM Username** and **IAM Password** from your Environment Details panel.
3. In the top search bar, type **S3** and click **S3** under Services.

### Task 1.2 — Create a New Bucket

1. Click **Create bucket**.
2. Fill in the following details:

   | Field | Value |
   |---|---|
   | Bucket name | `contoso-lab-<your-unique-id>` (e.g. `contoso-lab-user01`) |
   | AWS Region | `us-east-1` |
   | Object Ownership | ACLs disabled (default) |
   | Block Public Access | Keep all options **checked** (default) |
   | Versioning | Disabled |
   | Encryption | SSE-S3 (default) |

3. Scroll down and click **Create bucket**.

### Task 1.3 — Upload a File to the Bucket

1. Click on your newly created bucket name.
2. Click **Upload** → **Add files**.
3. Create a simple text file on the Jump VM desktop named `test-upload.txt` with the content:  
   `Hello from Contoso Lab!`
4. Upload the file and click **Upload**.
5. Verify the file appears in the bucket with status **Succeeded**.

### ✅ Validation Checkpoint 1
- [ ] S3 bucket created with the correct naming format
- [ ] File successfully uploaded to the bucket

---

## Exercise 2: Deploy a Small EC2 Virtual Machine

### Task 2.1 — Navigate to EC2

1. In the AWS Console search bar, type **EC2** and click **EC2** under Services.
2. Click **Instances** in the left navigation panel.
3. Click **Launch instances**.

### Task 2.2 — Configure the Instance

Fill in the launch wizard with the following values:

**Step 1 — Name and Tags**

| Field | Value |
|---|---|
| Name | `contoso-small-vm` |

**Step 2 — Application and OS Image (AMI)**

| Field | Value |
|---|---|
| AMI | Amazon Linux 2023 AMI (Free tier eligible) |
| Architecture | 64-bit (x86) |

**Step 3 — Instance Type**

| Field | Value |
|---|---|
| Instance type | `t3.micro` |

**Step 4 — Key Pair**

| Field | Value |
|---|---|
| Key pair | Select **Proceed without a key pair** (for lab purposes) |

**Step 5 — Network Settings**

| Field | Value |
|---|---|
| VPC | Select the **contoso-lab-vpc** (pre-deployed) |
| Subnet | Select **contoso-lab-subnet** |
| Auto-assign public IP | Enable |
| Security group | Create a new security group named `contoso-vm-sg` |
| Inbound rule | Allow SSH (port 22) from Anywhere |

**Step 6 — Storage**

| Field | Value |
|---|---|
| Root volume | 8 GiB gp3 |

### Task 2.3 — Launch the Instance

1. Review your settings and click **Launch instance**.
2. Click **View all instances**.
3. Wait until the **Instance State** shows **Running** and the **Status Check** shows **2/2 checks passed** (this takes 2–3 minutes).

### Task 2.4 — Verify the Instance

1. Select your `contoso-small-vm` instance.
2. In the **Details** tab at the bottom, note down the:
   - **Instance ID**
   - **Public IPv4 address**
   - **Private IPv4 address**

### ✅ Validation Checkpoint 2
- [ ] EC2 instance named `contoso-small-vm` is in **Running** state
- [ ] Instance type is `t3.micro`
- [ ] Instance passes 2/2 status checks

---

## Exercise 3: Configure a Virtual Private Cloud (VPC)

> **Note:** A base VPC (`contoso-lab-vpc`) has been pre-deployed in your environment. In this exercise you will explore and extend it.

### Task 3.1 — Explore the Pre-deployed VPC

1. In the AWS Console search bar, type **VPC** and click **VPC** under Services.
2. Click **Your VPCs** in the left panel.
3. Locate **contoso-lab-vpc** with CIDR `10.0.0.0/16`.
4. Click on it and review the **Details** tab — note the VPC ID.

### Task 3.2 — Create a New Subnet

1. In the left panel, click **Subnets** → **Create subnet**.
2. Fill in the following:

   | Field | Value |
   |---|---|
   | VPC ID | Select `contoso-lab-vpc` |
   | Subnet name | `contoso-private-subnet` |
   | Availability Zone | `us-east-1b` |
   | IPv4 CIDR block | `10.0.2.0/24` |

3. Click **Create subnet**.

### Task 3.3 — Create and Attach an Internet Gateway

1. In the left panel, click **Internet Gateways** → **Create internet gateway**.
2. Name it `contoso-lab-igw` and click **Create internet gateway**.
3. Click **Actions** → **Attach to VPC**.
4. Select `contoso-lab-vpc` and click **Attach internet gateway**.

### Task 3.4 — Create a Route Table

1. In the left panel, click **Route Tables** → **Create route table**.
2. Fill in:

   | Field | Value |
   |---|---|
   | Name | `contoso-public-rt` |
   | VPC | `contoso-lab-vpc` |

3. Click **Create route table**.
4. Select `contoso-public-rt` → click **Edit routes** → **Add route**:

   | Field | Value |
   |---|---|
   | Destination | `0.0.0.0/0` |
   | Target | Select **Internet Gateway** → `contoso-lab-igw` |

5. Click **Save changes**.
6. Go to the **Subnet associations** tab → **Edit subnet associations**.
7. Select `contoso-private-subnet` and click **Save associations**.

### Task 3.5 — Configure a Security Group

1. In the left panel, click **Security Groups** → **Create security group**.
2. Fill in:

   | Field | Value |
   |---|---|
   | Security group name | `contoso-web-sg` |
   | Description | Allow HTTP and HTTPS traffic |
   | VPC | `contoso-lab-vpc` |

3. Under **Inbound rules**, click **Add rule** twice:

   | Type | Protocol | Port | Source |
   |---|---|---|---|
   | HTTP | TCP | 80 | Anywhere (0.0.0.0/0) |
   | HTTPS | TCP | 443 | Anywhere (0.0.0.0/0) |

4. Click **Create security group**.

### ✅ Validation Checkpoint 3
- [ ] New subnet `contoso-private-subnet` (10.0.2.0/24) created
- [ ] Internet Gateway `contoso-lab-igw` created and attached to VPC
- [ ] Route table `contoso-public-rt` configured with route to internet gateway
- [ ] Security group `contoso-web-sg` created with HTTP and HTTPS rules

---

## Exercise 4: Connect to the Jump VM and Verify Resources

### Task 4.1 — Open the Jump VM Desktop

1. Go back to your CloudLabs environment tab.
2. Click the **Connect** button next to **contoso-jump-vm**.
3. The Linux desktop (XFCE) will open in your browser — no RDP client needed.
4. Use the credentials from your **Environment Details** panel to log in.

### Task 4.2 — Access AWS CLI on the Jump VM

1. On the Jump VM desktop, right-click and open **Terminal Emulator**.
2. Configure the AWS CLI with your lab credentials:

   ```bash
   aws configure
   ```

   Enter the following when prompted:

   | Prompt | Value |
   |---|---|
   | AWS Access Key ID | (from Environment Details panel) |
   | AWS Secret Access Key | (from Environment Details panel) |
   | Default region | `us-east-1` |
   | Default output format | `json` |

### Task 4.3 — Verify All Resources via CLI

Run the following commands to verify your lab resources:

**Check S3 bucket:**
```bash
aws s3 ls
```
> You should see your `contoso-lab-<id>` bucket listed.

**Check EC2 instance:**
```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=contoso-small-vm" \
  --query "Reservations[*].Instances[*].{ID:InstanceId,State:State.Name,Type:InstanceType}" \
  --output table
```
> You should see your instance with state `running`.

**Check VPC:**
```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=contoso-lab-vpc" \
  --query "Vpcs[*].{VpcId:VpcId,CIDR:CidrBlock,State:State}" \
  --output table
```
> You should see `contoso-lab-vpc` with CIDR `10.0.0.0/16` and state `available`.

**Check Subnets:**
```bash
aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=<your-vpc-id>" \
  --query "Subnets[*].{Name:Tags[0].Value,CIDR:CidrBlock,AZ:AvailabilityZone}" \
  --output table
```
> You should see both subnets listed.

### ✅ Validation Checkpoint 4
- [ ] Successfully connected to Jump VM via browser RDP
- [ ] AWS CLI configured with lab credentials
- [ ] S3 bucket visible via `aws s3 ls`
- [ ] EC2 instance shows state `running`
- [ ] VPC shows state `available`

---

## Lab Summary

Congratulations! 🎉 You have successfully completed the **Contoso AWS Certification Practice Lab**.

Here is a summary of what you accomplished:

| Task | Resource | Status |
|---|---|---|
| Exercise 1 | Amazon S3 Bucket created and file uploaded | ✅ |
| Exercise 2 | EC2 Virtual Machine deployed and running | ✅ |
| Exercise 3 | VPC configured with subnet, IGW, route table, and security group | ✅ |
| Exercise 4 | Jump VM accessed via browser RDP, CLI verification complete | ✅ |

---

## Scoring & Retake Policy

- A **passing score** requires completing all 4 validation checkpoints.
- If you do not achieve a passing score, you may **retake this lab up to 3 times**.
- Each retake provisions a **fresh, dedicated AWS environment**.
- Your previous environment is deleted when a retake is initiated.

---

## Cleanup (Optional — Admin Managed)

Your lab environment will be **automatically deleted** when the lab duration expires. You do not need to manually delete resources.

---

## Troubleshooting

| Issue | Solution |
|---|---|
| Jump VM not connecting | Refresh the page and try again. If it persists, click the VM restart button in the Environment Resources tab. |
| AWS Console login fails | Double-check you are using the IAM Username and Password from the Environment Details panel (not your personal AWS account). |
| EC2 instance stuck in Pending | Wait 3–5 minutes. AWS takes time to allocate resources. |
| CLI command not found | Run `sudo apt install awscli -y` in the Jump VM terminal. |
| S3 bucket name already exists | S3 bucket names are globally unique — add a number suffix e.g. `contoso-lab-user01-2`. |

---

## Additional Resources

- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)

---

*Lab Guide Version 1.0 | Contoso Certification Academy | Powered by CloudLabs (Spektra Systems)*
