Here is the complete, comprehensive technical project report documenting your entire journey from a blank AWS console to a fully automated, production-ready cloud deployment.

This document is written from your perspective ("I") so you can easily use it for your professional portfolio, resume, or GitHub repository.

---

# PROJECT REPORT: COMPREHENSIVE ARCHITECTURE & DEPLOYMENT OF A SECURE SERVERLESS STATIC WEBSITE WITH AUTOMATED CI/CD

---

## 1. Executive Summary

This project details the architectural engineering, security hardening, and deployment automation of a serverless personal portfolio website on Amazon Web Services (AWS). The design goal was to move away from legacy server hosting and instead implement a modern cloud-native system featuring a zero-trust storage backend, a global Content Delivery Network (CDN) edge cache, and a fully automated Continuous Integration/Continuous Deployment (CI/CD) pipeline.

By combining Amazon S3, Amazon CloudFront, and GitHub Actions, I successfully launched a highly scalable, blazing-fast web platform that updates globally within seconds of a code push—all while maintaining a 100% private backend isolated from direct internet threats.

---

## 2. Infrastructure Architecture Blueprint

The system architecture is bifurcated into two foundational planes: the **Global Delivery Plane** and the **DevOps Automation Plane**.

```
DEVELOPMENT & DEPLOYMENT PLANE (CI/CD)
[ VS Code / Local Workspace ] ──(git push)──> [ GitHub Repo ] ──(Trigger)──> [ GitHub Actions ]
                                                                                   │
                      ┌────────────────────────────────────────────────────────────┘
                      ▼ (Authenticated via Secure IAM Secrets Keys)
AWS HOSTING & DISTRIBUTION PLANE
[ Public User ] ──(HTTPS)──> [ CloudFront CDN Edge Cache ] ──(OAC Security)──> [ Private S3 Bucket ]
                                                                                   │
                                                                             [ /frontend assets ]

```

---

## 3. Step-by-Step Implementation Narrative

### Phase 1: Storage Layer Isolation (Amazon S3)

The foundational tier of the architecture serves as the permanent object store for all frontend assets.

* **Bucket Initialization:** I provisioned an isolated Amazon S3 bucket named `bhanuprakash-static-website-s3-bucket` within the `us-east-1` (N. Virginia) region.
* **Security Hardening Baseline:** To enforce a strict security baseline, I enabled **Block *all* public access**. This explicitly guarantees that the bucket is completely sealed against anonymous web browsing and direct HTTP access attempts.
* **Static Hosting Properties:** Within the bucket configuration, I designated **`index.html`** as the default entry point document.

### Phase 2: Global Content Delivery & Access Governance (CloudFront & OAC)

To expose the website securely to global users with minimal latency, I integrated an edge-routing content delivery layer.

* **CDN Provisioning:** I deployed an Amazon CloudFront distribution, mapping its upstream origin server directly to my newly created S3 bucket domain.
* **Origin Access Control (OAC) Enforcement:** To maintain data privacy, I rejected legacy access configurations and implemented **Origin Access Control (OAC)**. OAC establishes a cryptographically verified connection profile, forcing AWS to verify that incoming traffic originates strictly from my specific CloudFront distribution before releasing objects.
* **Access Policy Injection:** I injected a highly restrictive bucket policy into the S3 instance. This explicit policy grants `s3:GetObject` capabilities exclusively to the CloudFront distribution identity, locking out the rest of the public internet.
* **Entry Object Resolution:** I mapped the distribution default root object configuration to **`index.html`**. This ensures seamless resolution to the main page when navigating to the root URL (`d1912p20xx46k7.cloudfront.net`).

### Phase 3: Identity & Access Management (AWS IAM)

With the core architecture functional, I began building the automation bridge by creating a secure identity inside AWS.

* **Programmatic Identity Creation:** I provisioned an AWS IAM user named **`github-s3-cloudfront-deployer`**. Following the principle of least privilege, I explicitly blocked management console web access, keeping this user strictly programmatic.
* **Custom IAM Security Policy:** I designed a custom permission profile called **`github-s3-cloudfront-policy`**. This profile restricts the user to precisely four actions: syncing web assets to S3 (`s3:PutObject`, `s3:DeleteObject`, `s3:ListBucket`) and clearing cache edges (`cloudfront:CreateInvalidation`).
* **API Credential Generation:** Navigating to the user's **Security credentials** menu, I bypassed default templates, selected the **Other** option, and safely generated an **Access Key ID** and **Secret Access Key** pair to authorize my remote deployment environment.

### Phase 4: CI/CD Pipeline Orchestration (GitHub Actions & VS Code)

The final step was implementing a fully automated, continuous deployment workflow to replace manual asset hand-offs.

* **Repository Scaffolding:** I created a public repository on GitHub named **`Static-Website-Project`**.
* **Secrets Vaulting:** To protect my administrative AWS credentials, I stored them in GitHub's encrypted secrets storage area. I mapped five secure environmental secrets: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `S3_BUCKET`, and `CLOUDFRONT_DIST_ID`.
* **Declarative Pipeline Configuration:** Using VS Code, I constructed a **YAML workflow file** at the root path: **`.github/workflows/deploy.yml`**. This script automates four sequential deployment tasks whenever a push event hits the `main` branch:
1. Bootstraps a fresh, clean Ubuntu virtual runner.
2. Authenticates securely into AWS using the encrypted repository secret keys.
3. Executes an automated `aws s3 sync` to upload only changed assets from the local `./frontend` folder into S3, cleanly removing old files.
4. Triggers an `aws cloudfront create-invalidation` across path `/*` to clear out edge-node memory caches worldwide.



---

## 4. Live Operational Validation & Results

To test the entire infrastructure end-to-end, I made a live modification to the source files inside VS Code.

I modified my portfolio's subtitle, expanding it from `"AWS Cloud & DevOps Engineer"` to add my security specialty: **`"AWS Cloud & DevOps Engineer & Cyber security"`**.

Upon running a `git push` command, the architecture responded beautifully:

1. **Instant Hook:** GitHub Actions caught the commit immediately.
2. **Automated Run:** The runner spun up, validated credentials, synchronized the new frontend folder, and cleared the global edge caches.
3. **Flawless Finish:** The workflow completed cleanly in just **14 seconds**.
4. **Live Verification:** Refreshing the cloud URL displayed the updated, secure website live to the world.

---

## 5. Final Key Technical Summary

| Infrastructure Layer      | Selected Service | Security Mechanism | Deployment Status |
| --- | --- | --- | --- |
| **Object Data Storage** | Amazon S3 | CloudFront OAC Only (Private) | **Fully Operational** |
| **Content Delivery / Cache** | Amazon CloudFront | SSL/TLS Global Edge Caching | **Fully Operational** |
| **Identity Delegation** | AWS IAM | Granular Least-Privilege Policy | **Fully Operational** |
| **Pipeline Automation** | GitHub Actions | Encrypted Environment Secrets | **Fully Operational (14s build)** |