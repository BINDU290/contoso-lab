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