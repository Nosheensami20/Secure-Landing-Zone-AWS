# AWS Secure Landing Zone

This project demonstrates how to build a **Secure Landing Zone** on AWS using different services including AWS Organizations, Service Control Policies (SCPs), centralized logging (S3 + KMS + CloudTrail), and IAM guardrails. This Secure Landing Zone gives an organization a strong, scalable, and auditable base for AWS. 

**Key benefits of this Secure Landing Zone:**
- Separation of responsibilities across multiple accounts.
- Centralized logging for auditing and incident response.
- Organizational guardrails that protect against mistakes, hackers, and compliance failures.
- Strong IAM security controls (password policies, MFA enforcement).

##**Project Architecture**
Organization Root (o-xxxxxxxxxx)
├── Production OU
│   └── Production Account
├── Sandbox OU
│   └── Sandbox Account
├── Security OU
│   ├── Logging Account
│   └── Security Account
└── Shared Services OU
    └── Shared Services Account
Centralized Logging & Guardrails:
- Org-wide CloudTrail → S3 bucket in Logging Account (with KMS, SSE, Versioning)
- SCPs at Root/OU enforce:
  - Deny CloudTrail Deletion
  - Require MFA for IAM Actions
  - Restrict to ca-central-1 region
- IAM Guardrails (per account):
  - Strong password policy
  - MFA enforced for all users


## Step 1. Create AWS Organization
- Created an AWS Organization in the admin account (management account). 
- This allows us to manage **multiple accounts under one root**.  


## Step 2. Create Organizational Units (OUs) and Accounts
We created **4 OUs** and relevant accounts and mapped them with right OU. Even though only the **Logging account** was actively used in this project, the OU structure prepared us for future growth, separation of duties and follwing best Governent practices. 
-**Production** → For production workloads
- **Security** → For security-related accounts
- **Sandbox** → For developers/test accounts
- **Shared Services** → For shared resources

![Alt text](screenshots/Organization units and accounts.png)
 

## Step 3. Baseline IAM Roles & Cross-Account Access
- Every account that we created came with the role `OrganizationAccountAccessRole`.  
- This role was used to switch from the Management account into the Logging account securely and perform assuming that role like created S3 bucket for storing cloudtrail logs in Logging account. 


## Step 4. Service Control Policies (SCPs)
We created and attached 3 SCPs to secure accounts:

1. **Deny CloudTrail Deletion**
  This policy prevents anyone from disabling or deleting CloudTrail and ensures centralized logging can never be tampered. Even if an account is accessed by unautohrized user or hacker, CloudTrail can't be deleted or disabled.

2. **Require MFA for IAM Actions**
The policy forces administrators to use Multi-Factor Authentication (MFA) when managing IAM or assuming roles. This prevents attackers from abusing IAM if credentials are leaked or compromised. 

3. **Restrict AWS Region (ca-central-1 only)**
This prevents users from launching resources outside the approved region (ca-central-1) and improves security posture, compliance, and controls cost by keeping all workloads in one place.

![Alt text](screenshots/Service Control Policies.png)


## **Testing SCPs**:

Verified that SCPs are working:
- Try to delete CloudTrail → action should be denied.
- Try IAM action without MFA → denied.
- Try launching resource in another region (e.g., us-east-1) → denied.


## Step 5. Centralized Loggging (S3 + KMS + CloudTrail):
We switched role to Logging account that was already created under the Security OU.

5.1. Create S3 bucket:
- Created an S3 bucket named as "org-central-logs-<account-id>".
Enabled:
        -Versioning → to keep all versions of log files.
        -Server-side encryption with KMS.

![Alt text](screenshots/Centralized logging in S3 bucket.png) 

5.2 Create KMS Key in Logging Account:
- We created a customer-managed KMS key to encrypt logs. This ensures logs are encrypted at rest and that only approved accounts/services (CloudTrail + management account) can decrypt them. It prevents accidental/malicious tampering.

## Step 6. Organization-Wide CloudTrail:

- Created CloudTrail(orgtrail) in the Management account.
- Applied to all accounts in the Organization.
- Logs delivered to the Logging account bucket.
- Enabled multi-region and log validation.

![Alt text](screenshots/Cloudtraid configuration.png)


## Step 7. IAM Account-Level Guardrails:

- Enabled strong password policy (12 characters, upper caase,lower case, symbols, numbers).
- Enforced MFA for root and IAM users

![Alt text](screenshots/Password policy.png)
![Alt text](screenshots/MFA policy.png)


##**Future Expansion**

-Future enhancements could include organization-wide compliance (AWS Config), continuous threat detection (GuardDuty), and network visibility (VPC Flow Logs). These would be included in upcoming projects. 

"# Secure-Landing-Zone-AWS" 
"# Secure-Landing-Zone-AWS" 
