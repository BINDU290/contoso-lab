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
