## Exercise 3: Configure a Virtual Private Cloud (VPC)

> **Note:** A base VPC (`contoso-lab-vpc`) has been pre-deployed in your environment. In this exercise you will explore and extend it.

### Task 3.1 — Explore the Pre-deployed VPC

1. In the AWS Console search bar, type **VPC** and click **VPC** under Services.
2. Click **Your VPCs** in the left panel.
3. Locate **contoso-lab-vpc** with CIDR `10.0.0.0/16`.
4. Click on it and review the **Details** tab — note the VPC ID.

<img width="1920" height="935" alt="Screenshot (211)" src="https://github.com/user-attachments/assets/ad26ca0b-e9c2-4d89-b577-dd60f95b6b94" />

### Task 3.2 — Create a New Subnet

1. In the left panel, click **Subnets** → **Create subnet**.

<img width="1920" height="943" alt="Screenshot (212)" src="https://github.com/user-attachments/assets/19c68390-c739-4352-9e02-8b8c2e68721a" />

2. Fill in the following:

   | Field | Value |
   |---|---|
   | VPC ID | Select `contoso-lab-vpc` |
   | Subnet name | `contoso-private-subnet` |
   | Availability Zone | `us-east-1b` |
   | IPv4 CIDR block | `10.0.2.0/24` |

3. Click **Create subnet**.

<img width="1920" height="948" alt="Screenshot (213)" src="https://github.com/user-attachments/assets/906c0dde-6867-45e6-95ab-11c9bfef6686" />
<img width="1920" height="1080" alt="Screenshot (214)" src="https://github.com/user-attachments/assets/0597802f-fa3d-42d0-9ffd-03a6f5bafe8b" />

### Task 3.3 — Create and Attach an Internet Gateway

<img width="1920" height="1080" alt="Screenshot (215)" src="https://github.com/user-attachments/assets/b0882a1f-ab77-436a-86ae-79a9bc7ad626" />

1. In the left panel, click **Internet Gateways** → **Create internet gateway**.
2. Name it `contoso-lab-igw` and click **Create internet gateway**.
3. Click **Actions** → **Attach to VPC**.
4. Select `contoso-lab-vpc` and click **Attach internet gateway**.

<img width="1920" height="1080" alt="Screenshot (216)" src="https://github.com/user-attachments/assets/48e02f24-5649-42fa-b61e-b5098c708701" />

### Task 3.4 — Create a Route Table

1. In the left panel, click **Route Tables** → **Create route table**.
2. Fill in:

   | Field | Value |
   |---|---|
   | Name | `contoso-public-rt` |
   | VPC | `contoso-lab-vpc` |

3. Click **Create route table**.
4. Select `contoso-public-rt` → click **Edit routes** → **Add route**:

<img width="1920" height="1080" alt="Screenshot (217)" src="https://github.com/user-attachments/assets/380c7248-f1e2-4b93-9c64-7402db4d8047" />

   | Field | Value |
   |---|---|
   | Destination | `0.0.0.0/0` |
   | Target | Select **Internet Gateway** → `contoso-lab-igw` |
   
<img width="1920" height="1080" alt="Screenshot (218)" src="https://github.com/user-attachments/assets/e756cd98-c4d0-4676-85a8-929fbc532f0f" />

5. Click **Save changes**.
6. Go to the **Subnet associations** tab → **Edit subnet associations**.
7. Select `contoso-private-subnet` and click **Save associations**.

### Task 3.5 — Configure a Security Group

1. In the left panel, click **Security Groups** → **Create security group**.

<img width="1920" height="1080" alt="Screenshot (219)" src="https://github.com/user-attachments/assets/3b090057-becb-4dce-9186-88f5b401bdf2" />

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

<img width="1920" height="1080" alt="Screenshot (220)" src="https://github.com/user-attachments/assets/ac85fbb1-8405-4b63-877a-417a11e6d981" />

### ✅ Validation Checkpoint 3
- [ ] New subnet `contoso-private-subnet` (10.0.2.0/24) created
- [ ] Internet Gateway `contoso-lab-igw` created and attached to VPC
- [ ] Route table `contoso-public-rt` configured with route to internet gateway
- [ ] Security group `contoso-web-sg` created with HTTP and HTTPS rules

---
