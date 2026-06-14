# Comprehensive Architecture & Deployment of a Secure Serverless Static Website with Automated CI/CD

## 1. Project Executive Summary
This project showcases the end-to-end engineering lifecycle of a highly resilient, serverless web platform hosted on Amazon Web Services (AWS). Built to shift away from traditional server-managed hosting architectures, this infrastructure leverages **Amazon S3** for secure asset storage, **Amazon CloudFront** for global Content Delivery Network (CDN) edge caching, and **GitHub Actions** for an automated Continuous Integration and Continuous Deployment (CI/CD) pipeline.

By enforcing an **Origin Access Control (OAC)** policy signature framework, the data store remains entirely private and hidden from direct public web access. All infrastructure actions are driven purely by automation: local modifications pushed from **VS Code** trigger a GitHub runner that authenticates securely via AWS IAM credentials, synchronizes the codebase into S3, and clears edge memory allocations worldwide via programmatic cache invalidations in under 15 seconds.

---

## 2. Infrastructure Architecture Blueprint

The cloud-native design separates operational tasks into an isolated global delivery layer and a decoupled deployment pipeline:

DEVELOPMENT & DEPLOYMENT PLANE (CI/CD Pipeline)
[ VS Code Workspace ] ──(git push)──> [ GitHub Repository ] ──(Triggers)──> [ GitHub Actions ]
│
┌────────────────────────────────────────────────────────────┘
▼ (Authenticated via Secure Programmatic IAM Access Keys)
AWS HOSTING & GLOBAL DISTRIBUTION PLANE
[ Public Web User ] ──(HTTPS)──> [ CloudFront CDN Edge Cache ] ──(OAC Security)──> [ Private S3 Bucket ]
│
[ /frontend assets ]

---

## 3. Detailed Infrastructure Implementation Engineering Log

### Phase 1: Storage Layer Provisioning & Network Isolation (Amazon S3)
To establish an immutable, serverless storage base for the static web assets, I engineered an isolated object storage bucket container.

1. **Bucket Lifecycle Initialization:** I navigated to the Amazon S3 Engine and provisioned a globally unique bucket identifier named `aws-static-website-s3-bucket` pinned natively inside the `us-east-1` (N. Virginia) region.
2. **Access Control Hardening Baseline:** Following modern security principles, I activated the **Block *all* public access** master flag. This forcefully guarantees that the underlying data objects are fundamentally shielded from anonymous external scans, direct HTTP URL traversal, or public bucket listings.
3. **Static Website Hosting Integration:** Inside the bucket properties, I turned on the static hosting metadata route. I explicitly defined the primary presentation root entry point, mapping the application default interface to point to `index.html`.

![S3 Static Website Hosting Engine Options Configuration](images/static-website-hosting-option.png)

---

### Phase 2: Global Edge Distribution & Access Governance (Amazon CloudFront)
To expose the website files safely to global users with minimal latency, I wrapped the private storage pool behind an edge-routed caching content delivery network.

1. **CDN Distribution Launch:** I initialized a global Amazon CloudFront distribution, binding its upstream origin endpoint directly to the regional secure S3 origin URL string.

![CloudFront Edge Distribution Initial Creation](images/distribution_button.png)

2. **Origin Access Control (OAC) Configuration:** I implemented an **Origin Access Control (OAC)** profile signature framework. This profile commands the edge locations to attach cryptographic authentication tokens to requests before they are passed down to the S3 bucket origin.

![Origin Access Control Gatekeeper Profile Creation](images/origin_control_access.png)

3. **Applying the S3 Bucket Policy Matrix:** To seal the storage tier completely, I modified the S3 bucket permissions by injecting a custom bucket policy block. This policy allows `s3:GetObject` data collection capabilities *only* if the incoming request originates from the dedicated CloudFront distribution identity service principal (`EX612QS1RFB1C`).

![Locked S3 Bucket Access Control Policy Configuration](images/bucket_policy.png)

4. **Default Root Object Parameter Resolution:** To prevent access denial faults when a user hits the base naked domain without specifying a path, I edited the distribution properties. I updated the **Default root object** field to target `index.html` explicitly, enabling flawless default loading over my assigned endpoint (`d1912p20xx46k7.cloudfront.net`).

![CloudFront Live Distribution State and Default Root Object Mapping](images/distribution_name.png)

---

### Phase 3: Programmatic Identity Delegations & Access Controls (AWS IAM)
To establish a secure authentication bridge between the third-party GitHub automation workspace and my internal AWS environment, I created an isolated machine identity profile.

1. **Programmatic Identity Account:** I provisioned a dedicated AWS Identity and Access Management (IAM) user identity named **`github-s3-cloudfront-deployer`**. Following zero-trust best practices, I explicitly blocked AWS Management Console web access, keeping this user strictly programmatic.

![IAM Programmatic User Configuration Record](images/iam_user_details.png)

2. **Custom IAM Permissions Policy Blueprint:** I drafted and assigned a custom least-privilege tracking policy called **`github-s3-cloudfront-policy`**. This policy limits the account's operational permissions exclusively to necessary tasks: syncing assets (`s3:PutObject`, `s3:DeleteObject`, `s3:ListBucket`) and clearing edge-node cache records (`cloudfront:CreateInvalidation`).

![Custom Highly Restrictive IAM Policy Structure](images/github_policy.png)

3. **API Credential Matrix Generation:** Inside the identity **Security credentials** panel, I selected the **Other** application use case type, reviewed the security alternatives, and generated an API credential set consisting of an **Access Key ID** and a hidden **Secret Access Key**.

![API Access Key Category Selection Workflow](images/security_credentails_option.png)

![IAM Access Key Security Risk Validation Warning](images/access_key.png)

![IAM User Security Credentials Account Panel](images/iam-user-security-credentails.png)

---

### Phase 4: CI/CD Pipeline Architecture & Engine Engineering (GitHub Actions)
The final milestone coordinates code version state changes with real-time, zero-touch deployment workflows.

1. **Encrypted Environment Secrets Injection:** I established my public source repository under the identifier **`Static-Website-Project`**. To protect my administrative cloud keys, I saved them within GitHub's encrypted secrets repository vault, mapping out `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `S3_BUCKET`, and `CLOUDFRONT_DIST_ID`.

![Configured GitHub Actions Repository Secrets Vault](images/gthub_credentials.png)

2. **Workspace Infrastructure Scaffolding:** Using VS Code on my local machine, I mapped out the directory scaffolding paths to construct the declarative automation workflow playbook file located at `.github/workflows/deploy.yml`.

![VS Code File System Structural Layout](images/github_directory.png)

3. **Automated Workflow Scripting (YAML):** I developed a precise build process designed to run on any `push` event to the `main` branch. The script runs sequentially: it checks out the code, configures the AWS credentials, runs an optimized `aws s3 sync` from the local `./frontend` directory to upload assets while removing dead files from S3, and issues a global `/*` cache invalidation call to clear all CloudFront edge memory arrays.

![Declarative YAML Pipeline Build Sequence Code](images/depoly_yml.png)

---

## 4. Production Pipeline Testing & Caching Validation

To validate the entire system end-to-end, I simulated a real-world developer update loop by modifying the landing page title code inside VS Code. I updated my professional title to add my cybersecurity specialization: **`"AWS Cloud & DevOps Engineer & Cyber security"`**.

1. **Initial Manual Cache Verification:** Early testing required running manual invalidation records (`/*`) via the AWS console to force the edge locations to clear out the historical cache files.

![Initiating a Manual Cache Invalidation Workspace Run](images/invalidation_creation.png)

![Successfully Completed AWS Invalidation Processing Track](images/successful_invalidation.png)

2. **Automated Deployment Hook:** Once the file change was saved, running a `git push` command automatically triggered the GitHub Actions runner. The pipeline successfully ran all deployment tasks in just **14 seconds**.

![GitHub Actions Automation Green Light Run Success Logs](images/github_actions.png)

3. **Bypassing the Local Browser Cache:** To evade the local browser's sticky memory cache, I executed a **Hard Refresh** sequence. The browser pulled the newly updated asset through the CloudFront distribution, proving that updates flow seamlessly from code commit to live deployment worldwide.

![Initial Production Web Presentation Frame](images/Screenshot%202026-06-14%20074135.png)

![Final Production Web Presentation with Title Updates Live](images/image_6f38e3.png)

---

## 5. Finalized Technical Architecture Summary Matrix

| Infrastructure Block Layer | Selected AWS / DevOps Engine | Underlying Security Access Mechanism | Deployment Verification Status |
| :--- | :--- | :--- | :--- |
| **Object Data Storage** | Amazon Simple Storage Service (S3) | CloudFront OAC Canonical Policy Only | **100% Operational (Private Boundary)** |
| **Edge Distribution Cache** | Amazon CloudFront Edge Network | SSL/TLS Global Cache Invalidation Engine | **100% Operational (Edge Caching Live)**|
| **Identity Delegation** | AWS Identity & Access Management (IAM) | Programmatic Custom Least-Privilege Rules | **100% Operational (API Restricted)** |
| **Pipeline Automation Orchestrator** | GitHub Actions Runner Service | Repository Vault Encrypted Environment Tokens | **100% Operational (14s Execution)** |