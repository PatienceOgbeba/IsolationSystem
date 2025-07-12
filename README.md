𝐀𝐮𝐭𝐨𝐦𝐚𝐭𝐞𝐝 𝐄𝐂𝟐 𝐓𝐡𝐫𝐞𝐚𝐭 𝐃𝐞𝐭𝐞𝐜𝐭𝐢𝐨𝐧 & 𝐈𝐬𝐨𝐥𝐚𝐭𝐢𝐨𝐧 𝐒𝐲𝐬𝐭𝐞𝐦 𝐰𝐢𝐭𝐡 𝐀𝐖𝐒 𝐋𝐚𝐦𝐛𝐝𝐚 & 𝐄𝐯𝐞𝐧𝐭𝐁𝐫𝐢𝐝𝐠𝐞 -⁣ 𝐀𝐧 𝐀𝐮𝐭𝐨𝐦𝐚𝐭𝐞𝐝 𝐂𝐥𝐨𝐮𝐝 𝐒𝐞𝐜𝐮𝐫𝐢𝐭𝐲 𝐭𝐨 𝐌𝐢𝐧𝐢𝐦𝐢𝐳𝐞 𝐅𝐢𝐧𝐚𝐧𝐜𝐢𝐚𝐥 & 𝐑𝐞𝐩𝐮𝐭𝐚𝐭𝐢𝐨𝐧 𝐑𝐢𝐬𝐤⁣⁣⁣⁣⁣
⁣
---

````markdown
# 🚨 Automated EC2 Threat Detection & Response on AWS

A lightweight, serverless security automation solution that detects suspicious EC2 instance launches and immediately isolates them — built using AWS Lambda, EventBridge, and CloudTrail. Designed to work in real-time, within the AWS Free Tier.

---

## 📌 Overview

Cloud computing offers agility and scale, but it also introduces new risks. One major concern is how quickly unauthorized or malicious EC2 instances can be spun up — leading to potential data breaches, compliance failures, or unexpected costs.

This project automates the detection of suspicious EC2 activity and takes immediate action without human intervention.

---

## ⚙️ Architecture

- **AWS Lambda** – Executes logic to inspect and isolate suspicious EC2 instances.
- **Amazon EventBridge** – Triggers Lambda in real-time on `RunInstances` events.
- **AWS CloudTrail** – Captures and logs API activity across the account.
- **Amazon CloudWatch Logs** – Provides audit and observability for actions taken.
- **Security Groups** – Quarantine mechanism by blocking inbound traffic.

---

## ✅ Prerequisites

- [ ] AWS account (Free Tier eligible)
- [ ] IAM permissions for EC2, Lambda, EventBridge, CloudTrail, CloudWatch, and Security Groups
- [ ] Familiarity with the AWS Management Console

---

## 🔐 Step 1: Enable CloudTrail

- [ ] Go to **CloudTrail** in the AWS Console.
- [ ] Create or use the **default trail**.
- [ ] Enable **Management Events** with **Write-only** or **All**.
- [ ] Ensure logs are being delivered to an **S3 bucket** (and optionally to **CloudWatch**).

---

## 🛡️ Step 2: Create a Quarantine Security Group

- [ ] Go to **EC2 → Security Groups → Create**.
- [ ] Name: `quarantine-sg`.
- [ ] **Inbound Rules**: Leave empty (blocks all access).
- [ ] **Outbound Rules**: Allow or restrict as required.
- [ ] Save the **Security Group ID** (e.g., `sg-xxxxxxxx`).

---

## 🧠 Step 3: Create the Lambda Function

- [ ] Navigate to **AWS Lambda → Create Function**.
- [ ] Choose “Author from scratch”.
  - Name: `IsolateSuspiciousEC2`
  - Runtime: Python 3.12
  - Role: Create new role with basic Lambda permissions
- [ ] Paste the following code in the inline editor:

```python
import boto3

ec2 = boto3.client('ec2')

def lambda_handler(event, context):
    instance_id = event['detail']['responseElements']['instancesSet']['items'][0]['instanceId']
    user = event['detail']['userIdentity']['userName']

    # Customize trusted users
    if user not in ['authorized-user1', 'authorized-user2']:
        print(f"Suspicious launch detected from user: {user}")
        ec2.modify_instance_attribute(
            InstanceId=instance_id,
            Groups=['sg-xxxxxxxx']  # Replace with your quarantine SG ID
        )
        ec2.stop_instances(InstanceIds=[instance_id])
        print(f"Instance {instance_id} stopped and quarantined.")
    else:
        print(f"Authorized instance launch: {user}")
````

* [ ] Replace `'sg-xxxxxxxx'` with your quarantine SG ID.
* [ ] Click **Deploy**.

---

## Step 4: Add IAM Permissions to Lambda

* [ ] Go to **IAM → Roles → Lambda execution role**.
* [ ] Attach these policies:

  * [ ] `AmazonEC2FullAccess`
  * [ ] `CloudWatchLogsFullAccess`

---

## Step 5: Create the EventBridge Rule

* [ ] Go to **Amazon EventBridge → Rules → Create Rule**.
* [ ] Name: `DetectSuspiciousEC2`
* [ ] Event Source: **AWS events**
* [ ] Event Pattern:

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventName": ["RunInstances"]
  }
}
```

* [ ] Target: Select **Lambda Function → IsolateSuspiciousEC2**
* [ ] Click **Create Rule**

---

## Step 6: Test the Setup

* [ ] Create an **IAM user** that is NOT on the trusted list in the Lambda function.
* [ ] Launch an **EC2 instance** using that user.
* [ ] Lambda should:

  * [ ] Stop the instance
  * [ ] Apply the quarantine security group
  * [ ] Log the action to **CloudWatch**

---

## Step 7: Monitor in CloudWatch

* [ ] Go to **CloudWatch → Log Groups → /aws/lambda/IsolateSuspiciousEC2**
* [ ] View recent logs to confirm detection and isolation actions were taken.

---

## Convert to CloudFormation

I have packaged and added the cloudformation template to this repository for ease of deployment via the console. 

---

## Why This Matters

Without automation, your security posture is only as good as your response time. This project:

* [x] Detects unauthorized EC2 launches instantly
* [x] Stops malicious instances in real time
* [x] Prevents data breaches and unnecessary costs
* [x] Runs on AWS Free Tier

---

## My Security Philosophy

> **"Proactivity over reactivity. Automation over manual toil."**

---

## Author

**Patience Sonia Ogbeba**
*I build intelligent automated systems that take smart actions.*

---

## License

MIT License — Feel free to fork, build, and improve upon it.
