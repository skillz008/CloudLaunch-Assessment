# 🌩 CloudLaunch AWS Infrastructure Project

## 📖 Overview
This project sets up **core AWS infrastructure** for the CloudLaunch initiative in two major phases:

- **Task 1:** Host a static website using **Amazon S3**, implement secure bucket policies, and configure an **IAM user with least-privilege access**.
- **Task 2:** Design a secure **VPC architecture** to prepare for future application and database deployment.

*(Bonus: Configured CloudFront for HTTPS and improved performance globally.)*

---

## ✅ Task 1 – Static Website Hosting (S3 + IAM)

### 🔹 Steps Taken
- **Created 3 S3 Buckets:**
 1. `launch-site-bucket` – Publicly hosts a static HTML/CSS/JS website.
 2. `launch-private-bucket` – Secure storage for private files (not public).
 3. `launch-visible-only-bucket` – Only listable, not readable by IAM user.

- **Configured Permissions:**
  - Enabled **static website hosting** on `launch-site-bucket`.
  - Used **bucket policies** for public read-only access on the site bucket.
  - Kept other buckets private (no public access).

- **Created IAM User (`cloudlaunch-user`):**
  - Applied **least privilege principle**:
    - `ListBucket` on all three buckets.
    - `GetObject` on `launch-site-bucket`.
    - `GetObject` & `PutObject` on `launch-private-bucket` (no Delete).
    - **No object access** on `launch-visible-only-bucket`.

- **(Bonus)** Set up **CloudFront** for:
  - HTTPS-enabled access.
  - Caching for global performance.

---

### 🌐 Access URLs
- **S3 Static Website:**  
  `http://launch-site-bucket.s3-website-us-east-1.amazonaws.com`

- **CloudFront (Bonus):**  
  `https://du30aeqjcfr6s.cloudfront.net/index.html`  

*(CloudFront provides HTTPS and better performance)*

---

## ✅ Task 2 – VPC Design

### 🔹 Network Layout
- **VPC:**  
  - Name: `cloudlaunch-vpc`  
  - CIDR: `10.0.0.0/16`

- **Subnets:**  
  - **Public Subnet (10.0.1.0/24):** For future load balancers or public endpoints.  
  - **Application Subnet (10.0.2.0/24):** Private, for app servers.  
  - **Database Subnet (10.0.3.0/28):** Private, for database instances.

- **Routing & Gateways:**  
  - Added **Internet Gateway (`cloudlaunch-igw`)** for the public subnet.  
  - **Public Route Table:** Sends `0.0.0.0/0` to IGW.  
  - **Private Route Tables:** No Internet route — fully private.

- **Security Groups:**  
  - `cloudlaunch-app-sg`: Allows HTTP (port 80) only within VPC.  
  - `cloudlaunch-db-sg`: Allows MySQL (3306) **only from the app subnet**.

- **IAM Permissions:**  
  - `cloudlaunch-user` has **read-only access** to:
    - VPC, Subnets, Route Tables, Security Groups.

---

## 📜 IAM Policy for `cloudlaunch-user`

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowListOnAllBuckets",
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": [
                "arn:aws:s3:::cloudlaunch-private-bucket",
                "arn:aws:s3:::cloudlaunch-site-bucket",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket"
            ]
        },
        {
            "Sid": "AllowGetPutOnPrivateBucketOnly",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-private-bucket/*"
        },
        {
            "Sid": "AllowGetOnlyOnSiteBucket",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cloudlaunch-site-bucket/*"
        },
        {
            "Sid": "ExplicitDenyDeleteObject",
            "Effect": "Deny",
            "Action": "s3:DeleteObject",
            "Resource": [
                "arn:aws:s3:::cloudlaunch-private-bucket/*",
                "arn:aws:s3:::cloudlaunch-site-bucket/*",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket/*"
            ]
        },
        {
            "Sid": "DenyObjectAccessOnVisibleOnlyBucket",
            "Effect": "Deny",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-visible-only-bucket/*"
        },
        {
            "Sid": "AllowReadOnlyVPCInfo",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/Name": "cloudlaunch-vpc"
                }
            }
        }
    ]
}
