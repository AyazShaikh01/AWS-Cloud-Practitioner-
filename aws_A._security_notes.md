# A. SECURITY (16 services, 4 categories)

How the 4 categories fit together (outside in):

**Network protection** (stop attacks at the edge) → **Identity & access** (who gets in) → **Data protection** (keys, secrets, certificates) → **Threat detection & response** (what if something still goes wrong).

Each category exists because it answers a different question. Services inside a category exist because the *user*, the *data source*, or the *reach* is different.

---

## 1. IDENTITY & ACCESS: "Who are you, and what may you do?"

The four services differ by **who the user is and where they log in**.

| Service                 | Who it's for                                    | What it does                                                                                    | Why it exists separately                                                                                     | Cost logic                                                                      |
| ----------------------- | ----------------------------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| **IAM**                 | Admins, apps, AWS services (inside one account) | Users, groups, **roles**, **policies**, MFA. Decides what each identity can do to AWS resources | The foundation. Every AWS action passes through it. Global service                                           | **Free**: security should never be a paid barrier                               |
| **IAM Identity Center** | **Your employees**                              | **Single sign-on (SSO)** to **many AWS accounts** and business apps with one login              | IAM users per account don't scale: 20 accounts would mean 20 logins per person. Works with AWS Organizations | **Free**: AWS wants multi-account setups to be secure and easy                  |
| **Cognito**             | **Your app's customers** (public users)         | Sign-up, sign-in, social login (Google, Facebook), MFA for web/mobile apps, scales to millions  | You never want customers to be IAM users. Different scale, different login UX                                | **Free tier**, then pay per monthly active user: cost tracks your customer base |
| **Directory Service**   | Windows/Active Directory-dependent workloads    | **Managed Microsoft Active Directory** (and LDAP) in AWS. Domain-join EC2, run AD-aware apps    | Lots of enterprise software needs AD. Running AD servers yourself is painful                                 | **Paid**: AWS runs real directory servers for you                               |

**Cue words**
- "permissions, policy, role, least privilege, MFA, root user" → **IAM**
- "employees sign in once, multiple AWS accounts, SSO" → **IAM Identity Center**
- "customers sign up / sign in to our mobile or web app" → **Cognito**
- "Active Directory, LDAP, domain join, Windows" → **Directory Service**

**Trap pairs**
- **Identity Center vs Cognito:** employees vs customers.
- **Identity Center vs Directory Service:** Identity Center = *sign-in to AWS*. Directory Service = *hosts an actual AD*.
- **IAM vs Identity Center:** IAM = permissions inside an account. Identity Center = one login across accounts. (An **SCP** from Organizations is neither: it is a guardrail, not an identity tool.)

**IAM must-knows:** users (long-term), groups (users only), roles (temporary credentials, for services and cross-account), policies (JSON). Default deny, **explicit deny always wins**. Root user: MFA, rarely used. Credential Report = all users' credential status. Access Advisor = services last accessed.

**Check 1.** A company has 30 AWS accounts. Employees should log in once with corporate credentials and reach their assigned accounts. → **IAM Identity Center**
**Check 2.** A game app needs player registration with Google login for 2 million players. → **Cognito**

---

## 2. DATA PROTECTION: "Keep keys, secrets, and certificates safe"

Three different things to protect, so the services differ by **what they protect**. Inside "keys" there is one more split: who holds the hardware.

| Service | Protects | What it does | Why it exists separately | Cost logic |
|---|---|---|---|---|
| **KMS** | **Encryption keys** | Create/manage keys used by most AWS services (S3, EBS, RDS...). AWS-managed or customer-managed keys. Usage logged in CloudTrail | The default, easy key service integrated everywhere. **Shared** AWS-managed hardware | Pay per customer-managed key + requests (AWS-managed keys are free) |
| **CloudHSM** | **Encryption keys** (dedicated hardware) | **Single-tenant hardware** security module. **You** control the keys; AWS cannot access them | Some regulations demand dedicated hardware and full key control | **Paid hourly** per HSM: you rent dedicated hardware |
| **Secrets Manager** | **Passwords, API keys, DB credentials** | Stores secrets and **rotates them automatically** | KMS holds *keys*; apps also need *secrets* with lifecycle and rotation | Pay per secret + API calls: you pay for storage and rotation automation |
| **ACM** | **SSL/TLS certificates** | Provisions, deploys, **auto-renews** certificates for HTTPS (ELB, CloudFront, API Gateway) | Certificates expire and need renewing. Manual renewal causes outages | **Public certificates free**: you only pay for the resources that use them |

**Cue words**
- "manage encryption keys, encrypt S3/EBS/RDS" → **KMS**
- "dedicated hardware, single-tenant, full control of keys, regulatory" → **CloudHSM**
- "store and rotate database passwords / API keys" → **Secrets Manager**
- "HTTPS, SSL/TLS certificate" → **ACM**

**Trap pairs**
- **KMS vs CloudHSM:** managed and shared vs dedicated and yours. Default answer is KMS unless the question says dedicated/single-tenant/compliance.
- **KMS vs Secrets Manager:** keys encrypt data. Secrets are things your app *uses* (passwords). Secrets Manager uses KMS underneath.
- **ACM vs KMS:** certificates for HTTPS in transit vs keys for encrypting data at rest.

**Check 1.** A bank's regulator requires keys generated and stored on dedicated hardware that the bank controls. → **CloudHSM**
**Check 2.** An app's database password must rotate every 30 days without code changes. → **Secrets Manager**

---

## 3. THREAT DETECTION & RESPONSE: "Something may be wrong: find, scan, discover, investigate, gather"

Five services exist because **security needs five different questions, each with a different data source**.

| Service | Question | Looks at | What it does |
|---|---|---|---|
| **Inspector** | "Do my workloads have **known vulnerabilities**?" | EC2, container images (ECR), Lambda | Automated **vulnerability scanning** (software flaws, unintended exposure). Prevention |
| **Macie** | "**Where is my sensitive data**?" | **S3** buckets | ML finds **PII** (names, card numbers) and exposure risk |
| **GuardDuty** | "Is something **suspicious happening now**?" | CloudTrail, VPC Flow Logs, DNS logs | **Threat detection** with ML and threat intelligence. No agents needed |
| **Detective** | "**Why** did it happen?" | Same logs, linked into graphs | **Root cause investigation** of a finding, with visual timelines |
| **Security Hub** | "What's my **overall posture**?" | Findings from GuardDuty, Inspector, Macie and others | **Aggregates** findings in one dashboard and checks compliance standards |

**The flow (memory hook):**
Prevent and discover (**Inspector, Macie**) → Detect (**GuardDuty**) → Investigate (**Detective**) → Aggregate (**Security Hub**).

**Hook words:** Inspector = **inspects** software. Macie = sensitive **data**. GuardDuty = **guards** against bad behavior. Detective = **detective** work. Hub = **hub** that collects.

**Cost logic:** all are pay-per-use (data analyzed, resources scanned, findings ingested), usually with a free trial. You pay in proportion to how much AWS has to analyze.

**Cue words**
- "malicious IP, compromised instance, unusual API calls, crypto mining" → **GuardDuty**
- "vulnerabilities, CVE, software scan" → **Inspector**
- "PII, credit card numbers, sensitive data in S3" → **Macie**
- "root cause, investigate, visualize relationships" → **Detective**
- "single dashboard, aggregate findings, compliance standards" → **Security Hub**

**Trap pairs**
- **GuardDuty vs Inspector:** *behavior right now* vs *flaws that exist in software*.
- **Detective vs Security Hub vs GuardDuty:** detect, investigate, aggregate are three different jobs.
- **Macie vs Inspector:** data in S3 vs software on compute.

**Check 1.** An EC2 instance is talking to a known cryptocurrency-mining server. → **GuardDuty**
**Check 2.** A security team wants one dashboard combining findings from several AWS security services. → **Security Hub**

---

## 4. NETWORK PROTECTION: "Block attacks at the edge"

Different **attack types and reach**.

| Service | Protects against | What it does | Cost logic |
|---|---|---|---|
| **WAF** | **Layer 7 web exploits**: SQL injection, XSS, bad bots, rate abuse | Rules to allow/block requests by IP, geography, pattern. Attaches to CloudFront, ALB, API Gateway | Pay per web ACL, rule and request |
| **Shield** | **DDoS attacks** (traffic floods) | **Standard:** automatic, always on, **free**. **Advanced:** extra protection, **24/7 DDoS Response Team**, **cost protection** for DDoS-driven scaling bills | Standard free for everyone. Advanced has a high monthly fee for high-profile apps |
| **Firewall Manager** | Inconsistent rules across accounts | **Central management** of WAF rules, Shield Advanced and security groups across accounts in AWS Organizations. **Does not filter traffic itself** | Pay per policy per Region |

**Why Shield has free and paid:** Standard protects the whole platform for everyone (basic DDoS is AWS's problem too). Advanced exists for businesses that need a response team and bill protection.

**Cue words**
- "SQL injection, XSS, block IP/country, rate limiting" → **WAF**
- "DDoS" → **Shield** ("response team" or "cost protection" → **Shield Advanced**)
- "multiple accounts, central firewall rules, Organizations" → **Firewall Manager**

**Trap pairs**
- **WAF vs Shield:** application-layer attacks vs traffic-flood attacks.
- **Firewall Manager:** it *manages* rules, it is not itself a firewall.
- **Specific IP blocked at the subnet** → network ACL (VPC concept, covered in Networking), not a service here.

**Check 1.** A web app behind CloudFront is hit by SQL injection attempts. → **WAF**
**Check 2.** A company with 40 accounts must enforce the same WAF rules everywhere. → **Firewall Manager**

---

## One-page recap

| Need | Pick |
|---|---|
| Permissions inside an account | IAM |
| Employee SSO across accounts | IAM Identity Center |
| Customer sign-up/sign-in | Cognito |
| Active Directory in AWS | Directory Service |
| Encryption keys (default) | KMS |
| Encryption keys, dedicated hardware | CloudHSM |
| Rotate passwords/API keys | Secrets Manager |
| HTTPS certificates | ACM |
| Suspicious behavior | GuardDuty |
| Software vulnerabilities | Inspector |
| Sensitive data in S3 | Macie |
| Root cause investigation | Detective |
| Aggregate findings | Security Hub |
| SQL injection / XSS | WAF |
| DDoS | Shield (Advanced = team + cost protection) |
| Central firewall rules across accounts | Firewall Manager |
