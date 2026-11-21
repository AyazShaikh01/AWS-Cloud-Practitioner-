# F. MONEY & MIGRATION (15 services, 3 categories)

Three categories, three questions:
- **Cost & support:** "Watch the money, optimize it, get help, buy software."
- **Migration:** "Assess, plan, and move to AWS."
- **Backup & DR:** "Recover from failure."

---

## 15. COST & SUPPORT

### The cost tools differ by **when** you look

```
BEFORE deploying      → Pricing Calculator (not in your sheet, but the exam asks about it)
DURING (alerts)       → Budgets
AFTER (analysis)      → Cost Explorer (graphs + forecast) / CUR (raw, most detailed)
OPTIMIZE              → Compute Optimizer (right-sizing)
```

| Service                         | Role             | What it does                                                                                                                             | Why it exists separately                                                    | Cost logic                                |
| ------------------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- | ----------------------------------------- |
| **Budgets**                     | **Alert**        | Set cost or usage thresholds and get alerts on **actual or forecasted** spend. Can also track Reserved Instance / Savings Plans coverage | Seeing the bill after the month ends is too late                            | A couple of budgets free, small fee after |
| **Cost Explorer**               | **Analyze**      | **Visualize past spending** by service, account, tag. **Forecasts** future spend. Recommends Savings Plans                               | Humans need graphs and filters, not raw data                                | Console free                              |
| **CUR** (Cost and Usage Report) | **Raw detail**   | The **most comprehensive** line-item cost and usage data, delivered to **S3**                                                            | Finance and BI teams need every line to analyze in Athena/QuickSight        | Report free, you pay S3 storage           |
| **Compute Optimizer**           | **Optimize**     | ML-based **right-sizing** recommendations for EC2, Auto Scaling, EBS, Lambda based on usage                                              | Tells you where you are overpaying for oversized resources                  | Basic free                                |
| **Support**                     | **Human help**   | AWS Support plans (see below)                                                                                                            | Different customers need different response levels                          | Monthly, rises with plan                  |
| **Marketplace**                 | **Buy software** | Catalog of **third-party software and services** (AMIs, SaaS), billed on your AWS bill                                                   | Finding, buying and paying vendors is slow. Marketplace puts it on one bill | You pay the vendor's price through AWS    |

**Concepts that go with these tools**
- **Cost allocation tags:** key-value labels so cost reports can group spend by project/team. Tags must be **activated** in the Billing console.
- **Consolidated billing** (Organizations): one bill, combined usage for **volume discounts**, shared Reserved/Savings Plans.
- **Free Tier types:** Always Free, 12 Months Free, Trials. (New-account structure may be credit-based. Trust the console.)

**Pricing models quick reference**

| Model                     | Remember                                                                                                                            |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **On-Demand**             | No commitment, pay as you go                                                                                                        |
| **Savings Plans**         | $/hour commitment, 1-3 yr. **Compute SP** is most flexible (EC2, Fargate, Lambda). **EC2 Instance SP** is locked to family + Region |
| **Reserved Instances**    | 1-3 yr, up to ~72% off. **Standard** (fixed) vs **Convertible** (can change)                                                        |
| **Spot**                  | Up to **90%** off, **AWS can interrupt**                                                                                            |
| **Dedicated Hosts**       | Physical server, **BYOL**, compliance                                                                                               |
| **Capacity Reservations** | Guarantees capacity in an AZ, **no discount**                                                                                       |
| **Data transfer**         | **IN free. OUT to internet paid.** Cross-AZ and cross-Region paid                                                                   |

### Support plans (pattern matters more than exact names)

| Plan                   | Highlights                                                                                             |
| ---------------------- | ------------------------------------------------------------------------------------------------------ |
| **Basic** (free)       | 24/7 customer service for account/billing, docs, forums, Health Dashboard, **limited Trusted Advisor** |
| **Developer**          | Business-hours email support                                                                           |
| **Business**           | **24/7 phone, chat, email**, **full Trusted Advisor**, production system down < 1 hr                   |
| **Enterprise On-Ramp** | Pool of **TAMs**, faster response                                                                      |
| **Enterprise**         | **Designated TAM** (Technical Account Manager), Concierge team, fastest response                       |

**Pattern: higher plan = faster response + more features. TAM only at the top.** AWS has updated its support plan structure, so confirm current names in the exam guide.
**Other support resources:** re:Post (community Q&A), Knowledge Center, Professional Services, Partner Network (APN), Solutions Architects.

**Cue words**
- "alert when spend exceeds, forecasted spend" → **Budgets**
- "visualize past spend, filter by service/tag, forecast" → **Cost Explorer**
- "most detailed, line-item, delivered to S3" → **CUR**
- "right-size, oversized instances, low utilization" → **Compute Optimizer**
- "24/7 phone, TAM, response time" → **Support**
- "third-party software, buy through AWS, AMI from a vendor" → **Marketplace**

**Trap pairs**
- **Budgets vs Cost Explorer:** *alert* vs *analyze*.
- **Cost Explorer vs CUR:** graphs and filters vs raw, most detailed data.
- **Pricing Calculator vs Cost Explorer:** *future estimate* vs *past analysis*.
- **Compute Optimizer vs Trusted Advisor:** specific right-sizing suggestions from usage vs broad best-practice checks.
- **Marketplace vs Service Catalog:** third-party software vs your own approved products.

**Check 1.** Alert the team when forecasted monthly spend will exceed $5,000. → **Budgets**
**Check 2.** Finance wants every line item of usage loaded into Athena. → **CUR**
**Check 3.** EC2 instances average 8% CPU. Which tool suggests smaller sizes? → **Compute Optimizer**

---

## 16. MIGRATION: "Assess, plan, move"

These services differ by the **phase of migration**, and then by **what is being moved** (servers, databases, bulk data).

```
ASSESS         →   PLAN / TRACK   →   MOVE
Discovery           Migration Hub      Application Migration Service (servers)
Migration Evaluator                    DMS (databases) + SCT (schema conversion)
                                       Snow Family (bulk data, offline)
```

| Service | Phase | What it does | Cost logic |
|---|---|---|---|
| **Application Discovery Service** | **Assess** | Collects **inventory, utilization and dependencies** of your on-premises servers so you know what you have | **Free** |
| **Migration Evaluator** | **Assess** | Builds the **business case**: projected AWS cost vs current cost, right-sized | **Free** |
| **Migration Hub** | **Track** | **One dashboard to track** migration progress across tools and accounts | **Free** |
| **Application Migration Service** | **Move servers** | **Lift and shift** (Rehost): continuously replicates servers to AWS with minimal downtime | Free for a limited period per server, then hourly |
| **DMS** (Database Migration Service) | **Move databases** | Migrates databases with **minimal downtime**, same-engine or different-engine, with continuous replication | Pay for replication instance |
| **SCT** (Schema Conversion Tool) | **Convert** | Converts the **database schema and code** when engines differ (for example Oracle to Aurora PostgreSQL). Used **with DMS** | **Free** |
| **Snow Family** | **Move bulk data (offline)** | **Physical devices** shipped to you: **Snowcone** (small), **Snowball Edge** (terabytes to petabytes, also compute), **Snowmobile** (exabytes, a truck) | Per device/job |

**Why so many?** Migrating is not one task. You must **know what you have**, **justify it**, **track it**, then **move three different things** (servers, databases, huge data) that need three different mechanisms.

**Link to the 7 Rs**
- **Retire** = discovery shows an unused app.
- **Rehost** (lift and shift) = **Application Migration Service**.
- **Replatform** (self-managed DB to RDS/Aurora) = **DMS** (+ SCT if the engine changes).

**Cue words**
- "inventory on-premises servers, dependencies, utilization" → **Application Discovery Service**
- "build a business case, estimate cost savings of migrating" → **Migration Evaluator**
- "track migration progress in one place" → **Migration Hub**
- "lift and shift servers, minimal downtime" → **Application Migration Service**
- "migrate a database, minimal downtime" → **DMS**
- "convert schema, Oracle to PostgreSQL" → **SCT**
- "move tens of terabytes, slow network, physical device" → **Snow Family**

**Trap pairs**
- **Discovery vs Migration Evaluator:** *what do I have* vs *what will it cost on AWS*.
- **Application Migration Service vs DMS:** servers vs databases.
- **DMS vs SCT:** DMS moves the data. SCT converts the schema when engines differ.
- **Snow Family vs Direct Connect/VPN:** one-time **offline** bulk transfer vs a network link. (Storage Gateway is ongoing hybrid storage.)

**Check 1.** A company wants a list of its on-premises servers and their dependencies before planning. → **Application Discovery Service**
**Check 2.** Migrate an Oracle database to Aurora PostgreSQL. → **SCT** (schema) + **DMS** (data)
**Check 3.** 80 TB of data must reach AWS and the network is too slow. → **Snow Family** (Snowball Edge)

---

## 17. BACKUP & DR: "Recover from failure"

| Service | Role | What it does | Cost logic |
|---|---|---|---|
| **AWS Backup** | **Backup** | **Central, policy-based backup** for many services (EBS, RDS, DynamoDB, EFS, S3...). Schedules, retention, cross-Region and cross-account copies | Pay for backup storage and restores |
| **Elastic Disaster Recovery** | **Disaster recovery** | **Continuously replicates servers** (on-premises or cloud) into a low-cost staging area in AWS, so you can **fail over in minutes** | Pay per replicating server + staging storage |

**Why both?** A backup is a **point-in-time copy** you restore from (slow, cheap). DR keeps a **near-current replica** ready to run (fast, a bit costlier). **Backup is not High Availability.**

### DR strategies: cost vs recovery speed

| Strategy | RPO / RTO | What it means | Cost |
|---|---|---|---|
| **Backup & Restore** | Hours | Restore backups after the event. Lower-priority workloads | **$** |
| **Pilot Light** | Tens of minutes | Data stays live, services idle. Provision and scale after the event | **$$** |
| **Warm Standby** | Minutes | Always running, but smaller. Scale up after the event | **$$$** |
| **Multi-site Active/Active** | Real-time | Zero downtime, near-zero data loss. Mission critical | **$$$$** |

**RTO** = how fast you recover. **RPO** = how much data you can afford to lose (as time).
AWS Backup supports the cheap end (**Backup & Restore**). Elastic Disaster Recovery works like **Pilot Light** (low-cost staging, then fail over).

**Cue words**
- "centralize backups, backup policy, across services/accounts" → **AWS Backup**
- "disaster recovery, replicate servers, fail over in minutes" → **Elastic Disaster Recovery**
- "cheapest DR, tolerate hours of downtime" → **Backup & Restore**
- "zero downtime, mission critical" → **Multi-site Active/Active**

**Check 1.** A company wants one place to define backup schedules for RDS, EBS and DynamoDB. → **AWS Backup**
**Check 2.** The cheapest DR strategy, with several hours of downtime acceptable. → **Backup & Restore**

---

## One-page recap

| Need | Pick |
|---|---|
| Alert on actual/forecasted spend | Budgets |
| Visualize past spend + forecast | Cost Explorer |
| Most detailed cost data in S3 | CUR |
| Right-size oversized resources | Compute Optimizer |
| 24/7 phone, TAM | Support (Business / Enterprise) |
| Buy third-party software | Marketplace |
| Inventory on-prem servers | Application Discovery Service |
| Business case for migrating | Migration Evaluator |
| Track migration progress | Migration Hub |
| Lift and shift servers | Application Migration Service |
| Move a database | DMS |
| Convert database schema | SCT |
| Bulk offline data transfer | Snow Family |
| Central backup policies | AWS Backup |
| DR failover of servers | Elastic Disaster Recovery |
