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
