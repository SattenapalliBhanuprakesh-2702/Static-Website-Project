# Secure Serverless Static Website Deployment with Automated CI/CD Pipeline

## 1. Project Executive Summary
This project demonstrates a production-ready, cloud-native architecture for hosting a secure, high-performance static portfolio website on Amazon Web Services (AWS). Moving away from legacy server-managed hosting, this solution implements a zero-trust storage backend utilizing **Amazon S3**, a global content delivery network (CDN) edge-caching layer via **Amazon CloudFront**, and a fully automated **GitHub Actions** CI/CD pipeline. 

By employing **Origin Access Control (OAC)**, the storage layer remains entirely isolated from public web threats. The automated pipeline guarantees that modifications made locally in **VS Code** are securely built, synchronized to S3, and globally refreshed via edge cache invalidations within seconds of a `git push`.

---

## 2. System Architecture Blueprint

The system splits operational tasks across two core areas: global end-user content delivery and continuous automation.
DEVELOPMENT & AUTOMATION PLANE (CI/CD)
[ VS Code Workspace ] ──(git push)──> [ GitHub Repository ] ──(Triggers)──> [ GitHub Actions ]
│
┌────────────────────────────────────────────────────────────┘
▼ (Authenticated via Secure IAM Access Keys)
AWS HOSTING & DISTRIBUTION PLANE
[ Web User ] ──(HTTPS)──> [ CloudFront CDN Edge Cache ] ──(OAC Security)──> [ Private S3 Bucket ]
│
[ /frontend assets ]
---

## 3. Step-by-Step Engineering Narrative

### Phase 1: Storage Tier & Isolation Layer (Amazon S3)
The foundation of the architecture hosts the web application's static assets inside an object storage bucket.

1. **Bucket Provisioning:** I created a globally unique bucket named `aws-static-website-s3-bucket` in the `us-east-1` region.
2. **Access Hardening:** I activated **Block *all* public access** settings to prevent anonymous internet users from traversing or reading the raw S3 bucket directly.
3. **Static Web Configuration:** I enabled static website hosting properties on the bucket, defining the default index presentation page to map to `index.html`.

![S3 Static Website Hosting Options](static-website-hosting-option.png)

---

### Phase 2: Global Content Delivery & Origin Access Control (Amazon CloudFront)
To expose the website files safely to global users without exposing the underlying storage bucket, I integrated an edge-routing routing framework.

1. **Distribution Launch:** I initialized an Amazon CloudFront distribution pointing its origin endpoint to the private S3 bucket.

![CloudFront Distribution Initialization](distribution_button.png)

2. **Origin Access Control (OAC):** I implemented an **Origin Access Control (OAC)** gatekeeper profile. This creates a secure signature requirement for any traffic attempting to communicate with the bucket origin.

![Origin Access Control Profile Creation](origin_control_access.png)

3. **Restricting the S3 Bucket Policy:** I applied a secure, custom S3 bucket policy. This policy explicitly restricts data extraction (`s3:GetObject`) solely to the unique CloudFront distribution identifier `EX612QS1RFB1C`.

![Secured S3 Bucket Policy Configuration](bucket_policy.png)

4. **Assigning the Default Root Object:** To resolve direct domain queries cleanly, I modified the distribution settings and mapped the **Default root object** field to `index.html`. This tells CloudFront exactly which file to look for when users land on the base domain identifier.

![CloudFront Domain and Root Object Mapping](distribution_name.png)

---

### Phase 3: Identity & Access Management (AWS IAM)
Before configuring the external automation engine, I had to create an identity profile inside AWS with specific, limited permissions.

1. **Programmatic Identity Account:** I provisioned an isolated IAM user named `github-s3-cloudfront-deployer` with management console access disabled.

![IAM User Creation Profile](iam_user_details.png)

2. **Custom IAM Permissions Policy:** I attached a custom inline policy named `github-s3-cloudfront-policy`. This enforces strict least-privilege rules, limiting the account to explicit S3 sync actions and CloudFront invalidation requests.

![Custom IAM Policy Details](github_policy.png)

3. **Programmatic Key Generation:** Under the **Security credentials** configuration pane, I selected the **Other** option to safely generate an active **Access Key ID** and **Secret Access Key** pair.

![Access Key Selection Configuration](security_credentails_option.png)

![IAM Key Acknowledgment Window](access_key.png)

![IAM User Security Credentials Home](iam-user-security-credentails.png)

---

### Phase 4: CI/CD Pipeline Engineering (GitHub Actions)
The final stage links the source repository to the hosting cloud to manage updates automatically.

1. **Repository & Secret Injection:** I established a public repository named `Static-Website-Project`. Inside the actions configuration vault, I securely added my deployment parameters as encrypted environmental secrets.

![GitHub Actions Encrypted Variables Repository Vault](gthub_credentials.png)

2. **Workflow Scaffolding:** Within my local project environment, I laid down the directory paths to establish the pipeline automation logic file at `.github/workflows/deploy.yml`.

![Workspace Folder Structural Directory](github_directory.png)

![YAML Pipeline Script Logic Configuration](depoly_yml.png)

---

## 4. Live Production Validation & Cache Triumph

To test the entire infrastructure end-to-end, I made a live modification to the website header source code inside VS Code, expanding my professional title to read: **`"AWS Cloud & DevOps Engineer & Cyber security"`**.

1. **Manual Invalidation Baseline:** Initially, the cache system required manual cache clearance requests (`/*`) to force updates out to global nodes.

![Creating a Manual CloudFront Invalidation](invalidation_creation.png)

![Successfully Processed Cache Invalidation Track](successful_invalidation.png)

2. **Automated Run Execution:** Upon running a `git push` command, the GitHub Actions runner intercepted the commit and flawlessly executed the deployment scripts in a fast **14-second** runtime window.

![GitHub Actions Successful Build Pipelines](github_actions.png)

3. **Live Deployment Outcome:** The system automatically synced the new assets to S3 and flushed the CloudFront edge nodes. Bypassing the local browser cache loop verified that the website reflects the live title changes globally.

![Original Live Web Presentation Frame](Screenshot%202026-06-14%20074135.png)

![Final Live Web Presentation displaying Title Additions](image_6f38e3.png)

---

## 5. Completed Infrastructure Metrics Matrix

| Architecture Block | Managed Service Node | Security Access Protocol | Deployment Verification Status |
| :--- | :--- | :--- | :--- |
| **Object Data Storage** | Amazon S3 | CloudFront OAC Principal Only | **100% Operational (Private Boundary)** |
| **Edge Delivery Content**| Amazon CloudFront | SSL/TLS Encrypted Edge Cache | **100% Operational (Global Edge Nodes)**|
| **Identity Administration** | AWS IAM | Granular Least-Privilege Rules | **100% Operational (Programmatic API Only)**|