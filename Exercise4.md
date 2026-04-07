## Exercise 4: Connect to the Jump VM and Verify Resources

### Task 4.1 — Open the Jump VM Desktop

1. Go back to your CloudLabs environment tab.
2. Click the **Connect** button next to **contoso-jump-vm**.

<img width="1920" height="952" alt="Screenshot (221)" src="https://github.com/user-attachments/assets/5e52b91c-3045-46ba-9763-e59ad8018682" />

<img width="1920" height="953" alt="Screenshot (222)" src="https://github.com/user-attachments/assets/cecef5ef-a033-4ab9-91d1-1dfdb8cd7fd6" />

3. The Linux desktop (XFCE) will open in your browser — no RDP client needed.
4. Use the credentials from your **Environment Details** panel to log in.

### Task 4.2 — Access AWS CLI on the Jump VM

1. On the Jump VM desktop, right-click and open **Terminal Emulator**.
2. Configure the AWS CLI with your lab credentials:

<img width="1920" height="992" alt="Screenshot (223)" src="https://github.com/user-attachments/assets/c3a0db89-2168-4327-b337-3baf2d64f4e9" />

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
   
<img width="1920" height="992" alt="Screenshot (225)" src="https://github.com/user-attachments/assets/cebe1e9f-7258-4a24-b2fa-0724280c7348" />

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

<img width="1920" height="992" alt="Screenshot (225)" src="https://github.com/user-attachments/assets/484725a1-6b8a-46f4-b214-25ebbd1982af" />

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
