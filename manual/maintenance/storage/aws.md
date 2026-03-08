---
description: >-
  Instructions for connecting AWS S3 as cloud storage for automatic uploading  
  of call recordings from MikoPBX
---

# Connecting AWS S3 Storage (In dev)

### Creating a Bucket

1. Go to the AWS console ([link](https://console.aws.amazon.com/)). Navigate to **"All services"** -> **"Storage"** -> **"S3"**.

<figure><img src="../../../.gitbook/assets/awsS3section.png" alt=""><figcaption><p>"S3" section in AWS</p></figcaption></figure>

2. Click **"Create bucket"**.

<figure><img src="../../../.gitbook/assets/awsS3createBucketBtn.png" alt=""><figcaption><p>Button for creating a bucket</p></figcaption></figure>

3. Enter any name for the bucket (field **"Bucket name"**). Leave all other parameters as default and click **"Create bucket".**

<figure><img src="../../../.gitbook/assets/awsS3bucketParametersUpdated.png" alt=""><figcaption></figcaption></figure>

#### Creating an IAM User and Access Keys

1. Go to **"All services"** -> **"Security, Identity, & Compliance"** -> **"IAM"**.
2. Next, create a new IAM user. Go to the **"Access Management"** tab, then **"Users"**. Click **"Create user"**.
3. Enter the name of the IAM user in the **"User name"** field.

Click **"Next"**.

4. Select **"Attach policies directly"** as the **"Permissions options"**. Scroll down the page.
5. In the **"Permissions policies"** section click **"Create policy"**.
6. In the newly opened tab, in the **"Policy editor"**, select **"JSON"** as the format and paste the following content into the parameters field:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::your-bucket-name",
        "arn:aws:s3:::your-bucket-name/*"
      ]
    }
  ]
}
```

