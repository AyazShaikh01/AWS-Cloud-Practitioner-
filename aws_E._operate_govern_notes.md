# E. OPERATE & GOVERN (21 services, 4 categories)

Four categories, four questions:
- **Observe & operate:** "What is happening, who did it, what changed, and how do I act on running systems?"
- **Multi-account governance:** "How do I control many accounts?"
- **Compliance & advice:** "How do I prove compliance and find improvements?"
- **Developer & deployment tools:** "How do I talk to AWS and ship code?"

---

## 11. OBSERVE & OPERATE

All six look at your running environment, but each answers a **different question**.

| Service | Question | What it does | Cost logic |
|---|---|---|---|
| **CloudWatch** | **How is it performing?** | **Metrics, logs, alarms, dashboards.** Basic monitoring 5 min, detailed 1 min (EC2). Alarms can trigger actions | Free tier, then pay for custom metrics, logs, alarms |
| **CloudTrail** | **Who did what?** | Records **API calls and account activity**: who, what, when, from where. **Event history is on by default (90 days)**. Create a trail to keep longer in S3 | Event history free; extra trails and data events cost |
| **Config** | **What changed, and is it compliant?** | Records **configuration history** of resources and checks them against **rules** (for example "no security group open to the world"). Can auto-remediate | Pay per configuration item and rule evaluation |
| **X-Ray** | **Where in the request path is the problem?** | **Traces requests** across microservices to find bottlenecks and errors | Pay per trace recorded/retrieved |
| **Health Dashboard** | **Is it AWS or me?** | Shows **AWS service health and events affecting your account** (outages, maintenance). Service health + your account health | **Free** |
| **Systems Manager** | **How do I operate my fleet?** | One place to **patch, run commands, and remote-manage** EC2 and on-prem servers. **Session Manager** = shell without SSH or open ports. **Parameter Store** = config and secrets. **Patch Manager** | Core tools free, some features paid |

**Why they're separate:** performance, audit, configuration, request tracing, AWS-side incidents and day-to-day operations are six different data sources and problems.

**Cue words**
- "metrics, alarm, CPU, dashboard, logs" → **CloudWatch**
- "who made the API call, audit, user activity" → **CloudTrail**
- "configuration history, compliance rule, what changed" → **Config**
- "trace requests, microservices, bottleneck" → **X-Ray**
- "AWS outage, scheduled maintenance, impact on my resources" → **Health Dashboard**
- "patch many instances, remote shell without SSH, manage fleet" → **Systems Manager**

**Trap pairs**
- **CloudWatch vs CloudTrail vs Config:** performance vs **who** vs **what changed**.
- **CloudWatch vs X-Ray:** metrics for a service vs tracing one request across services.
- **Health Dashboard vs Trusted Advisor:** events on AWS side vs best-practice checks on your setup.
- **Config vs CloudTrail:** Config shows the *state* of a resource over time. CloudTrail shows the *API call* that caused it.
- **Systems Manager vs CloudFormation:** operate existing servers vs define and create infrastructure.

**Check 1.** An alarm must fire when CPU stays above 80%. → **CloudWatch**
**Check 2.** A security group was changed last week. Who changed it? → **CloudTrail**. What did it look like before? → **Config**
**Check 3.** Patch 200 EC2 instances without opening SSH. → **Systems Manager**

---

## 12. MULTI-ACCOUNT GOVERNANCE: "Control many accounts"

Companies use many AWS accounts (per team, per environment) for isolation and billing. These services exist because **many accounts need central control**. They differ by **what they control**.

| Service                           | Controls                    | What it does                                                                                                              | Cost logic                                       |
| --------------------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| **Organizations**                 | **The accounts themselves** | Group accounts in **OUs**, **consolidated billing**, create accounts, apply **SCPs** (permission guardrails)              | **Free**                                         |
| **Control Tower**                 | **A ready-made safe setup** | Sets up a secure **multi-account landing zone** with best-practice blueprints and **guardrails**. Builds on Organizations | No extra fee: pay for the services it configures |
| **RAM** (Resource Access Manager) | **Sharing**                 | **Share resources across accounts** (for example subnets, Transit Gateway) instead of duplicating them                    | **Free**                                         |
| **Service Catalog**               | **Approved products**       | A catalog of **pre-approved IT products** (CloudFormation-based) that teams can self-deploy with governance               | Low usage-based; you pay for what gets launched  |
| **License Manager**               | **Software licenses**       | Tracks and enforces **software licenses** (Microsoft, Oracle, SAP) across AWS and on-prem. Helps with BYOL                | **Free**                                         |
| **Service Quotas**                | **Limits**                  | View and request increases for **service limits (quotas)** per account and Region                                         | **Free**                                         |

**Reading the list as a story:** Organizations is the **foundation** (accounts). Control Tower is the **opinionated shortcut** on top of it. RAM shares resources among those accounts. Service Catalog controls **what users may deploy**. License Manager tracks **licenses**. Service Quotas tracks **limits**.

**Cue words**
- "multiple accounts, OUs, consolidated billing, SCP" → **Organizations**
- "set up a secure multi-account environment quickly, landing zone, guardrails" → **Control Tower**
- "share resources between accounts" → **RAM**
- "approved products, self-service, catalog of templates" → **Service Catalog**
- "track software licenses, BYOL compliance" → **License Manager**
- "service limit, quota, request an increase" → **Service Quotas**

**Trap pairs**
- **Organizations vs Control Tower:** Organizations = the tool and the rules. Control Tower = the pre-configured, best-practice setup built on it.
- **Control Tower vs Service Catalog:** governing the whole *environment* vs governing *which products* users can launch.
- **SCP (in Organizations) vs IAM Identity Center:** an SCP is a guardrail. **It never grants permissions.** It applies to member accounts, **not the management account**.
- **Service Quotas vs Trusted Advisor:** Service Quotas manages the limits. Trusted Advisor includes a check that warns when you approach them.

**Check 1.** A company wants a well-architected multi-account environment with guardrails set up in a few clicks. → **Control Tower**
**Check 2.** A VPC subnet must be shared with other accounts in the organization. → **RAM**
**Check 3.** An account hit the limit on the number of EC2 instances. → **Service Quotas**

---

## 13. COMPLIANCE & ADVICE: "Prove it, and improve it"

Four services, **two jobs**: provide *evidence* (Artifact, Audit Manager) and give *advice* (Trusted Advisor, Well-Architected Tool).

| Service | Job | What it does | Cost logic |
|---|---|---|---|
| **Artifact** | Evidence (**AWS's**) | On-demand download of **AWS compliance reports** (SOC, ISO, PCI) and agreements. For auditors asking "is AWS compliant?" | **Free** |
| **Audit Manager** | Evidence (**yours**) | **Continuously collects evidence of your own AWS usage** and maps it to frameworks (PCI DSS, HIPAA, GDPR) to simplify audits | Pay per assessment/evidence |
| **Trusted Advisor** | Advice (**automated**) | **Automated checks of your live account** for **cost optimization, performance, security, fault tolerance, service limits** (newer docs add operational excellence). Core checks free for all, **full checks need Business support or higher** | Free tier of checks; full set tied to the support plan |
| **Well-Architected Tool** | Advice (**guided review**) | Free self-service tool: answer questions about a workload against the **6 pillars**, get an improvement plan | **Free** |

**Why both evidence services?** The Shared Responsibility Model: AWS proves *its* part (Artifact). You prove *yours* (Audit Manager).
**Why both advice services?** Trusted Advisor **scans** your account automatically. The Well-Architected Tool **walks you through** an architecture review. One is a scanner, one is a guided questionnaire.

**Cue words**
- "download compliance reports, SOC, ISO, agreements" → **Artifact**
- "collect evidence for our audit, map usage to frameworks" → **Audit Manager**
- "best-practice checks, cost/security/performance recommendations" → **Trusted Advisor**
- "review workload against the six pillars, improvement plan" → **Well-Architected Tool**

**Trap pairs**
- **Artifact vs Audit Manager:** AWS's reports vs your own evidence.
- **Trusted Advisor vs Well-Architected Tool:** automated checks vs guided review.
- **Trusted Advisor vs Config:** advice on best practices vs history and custom rules.
- **Trusted Advisor vs Health Dashboard:** your setup vs AWS events.

**Check 1.** An auditor wants AWS's SOC 2 report. → **Artifact**
**Check 2.** A company wants automated recommendations on idle resources and open ports in its live account. → **Trusted Advisor**

---

## 14. DEVELOPER & DEPLOYMENT TOOLS: "How do I talk to AWS and ship code?"

| Service | Role | What it is | Cost logic |
|---|---|---|---|
| **Management Console** | **GUI** | Browser interface to browse and manage services. Good for learning and one-off tasks | Free |
| **CLI** | **Command line** | Control services with commands. For **scripting and automation**. Uses access keys (**programmatic access**) | Free |
| **CloudFormation** | **Infrastructure as Code** | Define resources in a **JSON/YAML template** and create them as a **stack**. Repeatable, version-controlled, can roll back | **Free**: you pay for the resources |
| **CodeBuild** | **Build** | Managed service that **compiles code, runs tests, produces artifacts** | Pay per build minute |
| **CodePipeline** | **Orchestrate (CI/CD)** | Automates the **release pipeline**: source → build → test → deploy | Pay per active pipeline |

**Ways to interact with AWS (exam list):** Management Console, **CLI**, **SDKs** (use AWS from code), and **APIs**. CLI, SDK and API are all **programmatic access**.

**Why separate?** The console is for humans clicking. The CLI is for scripts. CloudFormation is for repeatable infrastructure. CodeBuild builds. CodePipeline chains build, test and deploy into one flow. (The deploy step itself is done by CodeDeploy, which is not in your sheet.)

**Cue words**
- "GUI, browser, manage services visually" → **Management Console**
- "commands, scripts, automate from a terminal" → **CLI**
- "template, JSON/YAML, repeatable environments, IaC" → **CloudFormation**
- "compile code, run unit tests, build artifacts" → **CodeBuild**
- "automate the release process, CI/CD, stages" → **CodePipeline**

**Trap pairs**
- **CodeBuild vs CodePipeline:** one *step* (build) vs the *whole flow*.
- **CloudFormation vs Elastic Beanstalk:** CloudFormation defines *infrastructure* from templates. Beanstalk deploys an *application* and creates infrastructure for you.
- **CloudFormation vs Systems Manager:** create resources vs operate existing ones.
- **Console vs CLI:** manual and visual vs scriptable and repeatable.

**Check 1.** A team must recreate identical dev, test and prod environments from one template. → **CloudFormation**
**Check 2.** A team wants every commit to be built, tested and released automatically. → **CodePipeline** (with CodeBuild for the build step)

---

## One-page recap

| Need | Pick |
|---|---|
| Metrics, alarms, logs | CloudWatch |
| Who made the API call | CloudTrail |
| What changed / is it compliant | Config |
| Trace a request across services | X-Ray |
| AWS outage affecting me | Health Dashboard |
| Patch/manage a fleet, no SSH | Systems Manager |
| Manage many accounts, SCPs, consolidated billing | Organizations |
| Pre-built secure multi-account setup | Control Tower |
| Share resources across accounts | RAM |
| Approved self-service products | Service Catalog |
| Track software licenses | License Manager |
| Service limits | Service Quotas |
| AWS compliance reports | Artifact |
| Your audit evidence | Audit Manager |
| Automated best-practice checks | Trusted Advisor |
| Guided architecture review (6 pillars) | Well-Architected Tool |
| GUI | Management Console |
| Scripts/commands | CLI |
| Infrastructure from templates | CloudFormation |
| Build and test code | CodeBuild |
| CI/CD pipeline | CodePipeline |
