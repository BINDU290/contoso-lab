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
