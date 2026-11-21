# B. COMPUTE (10 services, 2 categories)

Two categories, two different questions:
- **Compute ladder:** "How much of the server do I want to manage?" (spectrum)
- **Containers:** "I use containers: where do I store, orchestrate and run them?" (job-sort)

---

## 5. COMPUTE LADDER: "How much do I want to manage?"

Same job (run code), different amount of work left to you.

```
MORE CONTROL / MORE WORK                              LESS CONTROL / LESS WORK
EC2  →  (ECS/EKS on EC2)  →  Elastic Beanstalk  →  Fargate  →  Lambda
        Lightsail = simplified branch (fixed-price server bundle)
        Batch = job scheduler, Auto Scaling = add-on that adds/removes EC2
```

| Service | What it is | Why it exists | Cost logic |
|---|---|---|---|
| **EC2** | Virtual servers (IaaS). You pick OS, size, storage, network | The base layer. Needed when you want full control: custom OS, Windows, licensing, any software | Pay per second/hour by instance type. Cheaper with Savings Plans, Reserved, Spot |
| **Lightsail** | Simple virtual private server **bundled with fixed monthly price** (compute, storage, data transfer, static IP). One-click blueprints like WordPress | EC2 + VPC + storage + IPs is too much for a blog or small site. Lightsail hides it | **Fixed, predictable monthly price**: that is the reason it exists |
| **Elastic Beanstalk** | **PaaS**: upload your code, AWS provisions EC2, load balancing, Auto Scaling and monitoring | Developers want to **deploy code, not design infrastructure**. Sits between EC2 and Lambda | **Beanstalk itself is free**: you pay for the resources it creates |
| **Lambda** | **Serverless** functions that run on events. **Max 15 minutes** per run | For short, event-driven work with zero server management | Pay per **request + duration**. Nothing runs, nothing paid. Generous free tier |
| **Batch** | Runs large numbers of **batch jobs**: queues and schedules them, picks the right compute (EC2 incl. Spot, or Fargate) | Heavy, long, queued jobs need a scheduler, not just a server | **No extra charge**: you pay for the compute it uses |
| **Auto Scaling** | Automatically **adds or removes instances** to match demand. Replaces unhealthy ones. Min / desired / max. Dynamic, scheduled, or predictive scaling | Delivers **elasticity** and availability so you don't guess capacity | **Free**: you pay for instances it launches |

**EC2 must-knows**
- **Instance families:** General Purpose, Compute Optimized, Memory Optimized, Storage Optimized, Accelerated Computing (GPU).
- **AMI** = template for a server. **User data** = script that runs at first boot.
- **Shared responsibility:** *you* patch the guest OS on EC2.
- Pricing options (Savings Plans, Reserved, Spot, Dedicated Hosts) are covered in the Cost category.

**Auto Scaling note:** it normally works with a load balancer in front (ELB is not in your sheet, but the exam mentions it). Spread across **multiple AZs** for high availability.

**Cue words**
- "full control, custom OS, Windows, own licenses, long-running" → **EC2**
- "simple, predictable monthly price, small website, WordPress, beginner" → **Lightsail**
- "upload code, don't manage infrastructure, deployment" → **Elastic Beanstalk**
- "serverless, event-driven, short, pay only when code runs" → **Lambda**
- "batch jobs, thousands of jobs, job queue" → **Batch**
- "automatically add/remove instances, elasticity" → **Auto Scaling**

**Trap pairs**
- **Beanstalk vs Lambda:** Beanstalk still runs EC2 for you. Lambda has no servers at all.
- **Beanstalk vs CloudFormation:** Beanstalk = application platform. CloudFormation = templates that define infrastructure (Dev Tools category).
- **Lambda vs Batch:** short event-driven functions vs long heavy queued jobs. Anything over 15 min is not Lambda.
- **Lightsail vs EC2:** same idea, but Lightsail trades control for simplicity and fixed price.
- **Auto Scaling vs Lambda scaling:** Auto Scaling adds EC2 instances. Lambda scales by itself.

**Who patches the OS?** EC2 = you. Beanstalk = AWS manages the platform, but you still own the app. Lambda = AWS.

**Check 1.** A small business wants a WordPress site with a predictable monthly bill and minimal setup. → **Lightsail**
**Check 2.** A developer wants to upload a Node.js app and let AWS handle servers, load balancing and scaling. → **Elastic Beanstalk**
**Check 3.** A nightly job runs for 40 minutes. Can it run on Lambda? → **No** (15 min max). Use Batch/EC2/Fargate.

---

## 6. CONTAINERS: "Store, orchestrate, run"

A container packages an app with everything it needs. Running many of them needs three different jobs, so there are three kinds of service.

| Service | Job | What it is | Why it exists separately | Cost logic |
|---|---|---|---|---|
| **ECR** | **Store** | Registry for container images: store, version, scan, share | Containers need a secure, AWS-integrated place for images | Pay for storage and data transfer |
| **ECS** | **Orchestrate** | AWS-native container orchestrator. Simple, tightly integrated with AWS | Easiest way to run containers on AWS without learning Kubernetes | **Control plane free**: pay for EC2/Fargate underneath |
| **EKS** | **Orchestrate** | Managed **Kubernetes** (open-source standard) | Teams already using Kubernetes want the same tools and portability (on-prem or other clouds) | **Hourly cluster fee** plus the compute underneath |
| **Fargate** | **Run (serverless)** | Serverless compute engine for **ECS and EKS** containers. No EC2 servers to manage | Even with an orchestrator you manage EC2 nodes. Fargate removes them | Pay per vCPU and memory per second |

**Why both ECS and EKS?** ECS = simplicity, AWS-only. EKS = Kubernetes standard and portability. Same goal, different audience.

**Why Fargate if ECS/EKS exist?** ECS/EKS decide *what runs where*. Fargate decides *what machine it runs on*, and removes the machine. Two launch choices: **EC2 launch type** (you manage servers) vs **Fargate** (you don't).

**Cue words**
- "store Docker images, container registry" → **ECR**
- "Docker containers, AWS-native, simple" → **ECS**
- "Kubernetes" → **EKS**
- "containers without managing servers/EC2" → **Fargate**

**Trap pairs**
- **Fargate is not an orchestrator.** It runs ECS or EKS tasks.
- **Fargate vs Lambda:** Fargate = containers, can run long. Lambda = functions, max 15 min.
- **ECS vs EKS:** "Kubernetes" in the question means EKS.
- **ECR vs ECS:** the first stores images, the second runs them.

**Check 1.** A team already uses Kubernetes on-premises and wants the same tooling on AWS. → **EKS**
**Check 2.** A team wants to run containers but never manage or patch the underlying servers. → **Fargate**

---

## One-page recap

| Need | Pick |
|---|---|
| Full control of a server | EC2 |
| Simple site, fixed monthly price | Lightsail |
| Upload code, AWS handles the infrastructure | Elastic Beanstalk |
| Event-driven, serverless, under 15 min | Lambda |
| Many queued/scheduled batch jobs | Batch |
| Add/remove instances automatically | Auto Scaling |
| Store container images | ECR |
| AWS-native container orchestration | ECS |
| Kubernetes | EKS |
| Containers without servers | Fargate |
