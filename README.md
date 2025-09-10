# AWS Secure Landing Zone

In this project we have created a multi-account landing zone implementation setup in AWS, featuring organizational governance, security guardrails, and centralized audit logging following AWS Well-Architected Framework principles. In this setup, we used AWS Organizations to create multiple accounts under one management account, each with a specific purpose (like Security, Logging, or Workloads). Service Control Policies (SCPs) are applied at organization level to all these accounts to control the actions an account is allowed/not allowed to perform. With centralized CloudTrail logging, all API calls are recorded to know who did what, when, and from where to enforce that only permissible acts are performed by the users. The centralized logging is enabled in a seprate logging account, to ensure that no other account can access it to modify, delete, or disable it. IAM policies are also enforced to ensure strict security controls are in place and rules are implemented like password policy, MFA requirement to keep the accounts secure. 
 
**Key benefits of this Secure Landing Zone:**
- Separation of responsibilities across multiple accounts.
- Centralized logging for auditing and incident response.
- Organizational guardrails that protect against mistakes, hackers, and compliance failures.
- Strong IAM security controls (password policies, MFA enforcement).

##**Project Architecture**
  - Management Account (Root)
  - Organization: o-xxxxxxxxxx
        * Security OU
        * Production OU
        * Sandbox OU
        * Shared Services OU
  - Org-wide CloudTrail → S3 bucket in Logging Account (with KMS, SSE, Versioning)
  - Centralized Logging & Guardrails
  - SCPs at Root enfored on OUs
  - IAM Guardrails 
    
## Step 1. Create AWS Organization
- Created an AWS Organization in the admin account (management account). 
- This allows us to manage multiple accounts under one root.  

## Step 2. Create Organizational Units (OUs) and Accounts
We created **4 OUs** and relevant accounts and mapped them with the relevant OU. Even though only the **Logging account** was actively used in this project, the OU structure prepared us for future growth, separation of duties and follwing best Governent practices. 

- **Production** → For production workloads
- **Security** → For security-related accounts
- **Sandbox** → For developers/test accounts
- **Shared Services** → For shared resources

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

## Step 5. Centralized Loggging (S3 + KMS + CloudTrail):
We switched role to Logging account that was already created under the Security OU.

5.1. Create S3 bucket:
- Created an S3 bucket named as "org-central-logs-<account-id>".
Enabled:
        -Versioning → to keep all versions of log files.
        -Server-side encryption with KMS.
  
5.2 Create KMS Key in Logging Account:
- We created a customer-managed KMS key to encrypt logs. This ensures logs are encrypted at rest and that only approved accounts/services (CloudTrail + management account) can decrypt them. It prevents accidental/malicious tampering.

## Step 6. Organization-Wide CloudTrail:

- Created CloudTrail(orgtrail) in the Management account.
- Applied to all accounts in the Organization.
- Logs delivered to the Logging account bucket.
- Enabled multi-region and log validation.

## Step 7. IAM Account-Level Guardrails:

- Enabled strong password policy (12 characters, upper caase,lower case, symbols, numbers).
- Enforced MFA for root and IAM users

## Validation Results:
**Security Control Testing**

- Cross-Account Access Functional: Role switching working properly
- CloudTrail Deletion Blocked: Attempted deletion denied by SCP
- MFA Enforcement Working: IAM actions without MFA properly blocked
- Regional Restrictions Active: Resource creation outside ca-central-1 denied

**Logging Verification**

- S3 Log Delivery: Recent log files appearing within 15 minutes
- CloudTrail Status: "Logging" status confirmed active
- Encryption Validation: KMS encryption confirmed on all log files

**Access Pattern Validation**

- Management Account Access: Can assume roles in all member accounts
- Logging Account Security: Centralized log storage accessible
- Account Isolation: No unauthorized access between accounts

## **Future Expansion**

This landing zone foundation enables additional security and governance capabilties which would be covered in upcoming projets.
- Organization-wide compliance (AWS Config),
- Continuous threat detection (GuardDuty), and
- Network visibility (VPC Flow Logs).
