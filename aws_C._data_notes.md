# C. DATA (12 services, 2 categories)

Two categories, same question asked two ways: **"What shape is my data?"**
- **Storage:** files and raw data. Shape = *how you access it* (object, archive, block, file).
- **Databases:** structured, queryable data. Shape = *the data model and the queries* (relational, key-value, cache, document, graph, warehouse).

---

## 7. STORAGE: "Object, archive, block, or file?"

| Service | Type | What it is | Why it exists separately | Cost logic |
|---|---|---|---|---|
| **S3** | **Object** | Unlimited storage of objects (files + metadata) in buckets, accessed over HTTP. **11 9s durability**. Bucket names are globally unique, buckets live in a Region | The default for almost everything: backups, websites, data lakes, media, logs | Pay per GB stored + requests + data transfer out. Colder class = cheaper storage |
| **S3 Glacier** | **Archive** | Archive storage classes of S3 for rarely accessed data, with slower retrieval | Long-term archives would be wasteful at normal S3 prices | Very cheap storage, but you pay (time or fees) to retrieve |
| **EBS** | **Block** | Virtual hard disk for **one EC2 instance**, in **one AZ**. Persists after the instance stops. **Snapshots** go to S3 | EC2 needs a fast boot and data disk | Pay per provisioned GB, **even if empty** |
| **EFS** | **File (Linux)** | Shared network file system (NFS) mounted by **many Linux instances across AZs**. Grows automatically | EBS can't be shared across instances and AZs | Pay per GB **actually used** (pricier per GB than EBS) |
| **FSx** | **File (specialized)** | Managed third-party file systems: **Windows File Server (SMB + Active Directory)**, Lustre (high-performance computing), NetApp ONTAP, OpenZFS | EFS is Linux/NFS only. Windows apps and HPC need their own file systems | Pay for provisioned capacity and throughput |

### S3 is also a spectrum (cost vs access speed)

| Class | Use when | Notes |
|---|---|---|
| **Standard** | Frequent access | Default |
| **Intelligent-Tiering** | **Unknown or changing** access pattern | Moves objects automatically, **no retrieval fee** |
| **Standard-IA** | Infrequent access, still needs milliseconds | Retrieval fee, multi-AZ |
| **One Zone-IA** | Infrequent and **easy to recreate** | **Single AZ**, cheaper |
| **Glacier Instant Retrieval** | Archive accessed ~quarterly, needs **milliseconds** | |
| **Glacier Flexible Retrieval** | Archive, minutes to hours retrieval | |
| **Glacier Deep Archive** | Long-term compliance, **cheapest**, ~12+ hours retrieval | |

**S3 features:** Versioning (recover overwrites/deletes), Lifecycle policies (auto-move or expire objects), Replication (to another bucket/Region), static website hosting, Block Public Access, Object Lock (WORM), encryption.

**Cue words**
- "object storage, backups, static website, data lake, unlimited, 11 9s" → **S3**
- "archive, long-term retention, lowest cost" → **S3 Glacier** (Deep Archive = cheapest)
- "disk for an EC2 instance, block storage, boot volume" → **EBS**
- "shared file system, many Linux instances, multiple AZs" → **EFS**
- "Windows file server, SMB, Active Directory, HPC" → **FSx**

**Trap pairs**
- **EBS vs EFS:** one instance, one AZ vs many instances, many AZs.
- **EFS vs FSx:** Linux/NFS vs Windows/SMB (or specialized).
- **EBS vs instance store:** EBS persists. Instance store is temporary and disappears on stop/terminate.
- **Versioning vs Replication:** versioning keeps old copies of the same object. Replication copies objects elsewhere (and copies an overwrite too).
- **Intelligent-Tiering vs Standard-IA:** unknown pattern vs known infrequent pattern.
- **Standard-IA vs One Zone-IA:** multi-AZ vs single AZ.

**Check 1.** Hundreds of Linux instances across AZs must share one file system. → **EFS**
**Check 2.** A company must keep compliance records 7+ years at the lowest cost, retrieval can take hours. → **S3 Glacier Deep Archive**
**Check 3.** A Windows application needs an SMB file share joined to Active Directory. → **FSx for Windows File Server**

---

## 8. DATABASES: "What shape are my data and my queries?"

Each database exists because **no single database model is best at everything**.

| Service | Model | What it is | Cue | Cost logic |
|---|---|---|---|---|
| **RDS** | **Relational (SQL)** | **Managed** MySQL, PostgreSQL, MariaDB, Oracle, SQL Server. AWS handles patching, backups, failover | "managed relational DB, SQL, move an existing database" | Pay per instance hour + storage |
| **Aurora** | **Relational (SQL)** | AWS-built, MySQL/PostgreSQL-compatible. Faster and more resilient: data copied **6 times across 3 AZs**, auto-growing storage, serverless option | "high-performance relational, cloud-native, MySQL/PostgreSQL compatible" | Costs more than standard RDS engines: you pay for performance and resilience |
| **DynamoDB** | **NoSQL key-value** | **Serverless**, single-digit millisecond at any scale. Global Tables, optional in-memory cache (DAX) | "NoSQL, key-value, serverless, massive scale" | Pay per request or provisioned capacity |
| **ElastiCache** | **In-memory cache** | Redis/Memcached cache in front of a database. Microsecond reads | "caching, in-memory, reduce database load" | Pay per node hour |
| **DocumentDB** | **Document (JSON)** | **MongoDB-compatible** managed document database | "MongoDB, JSON documents" | Pay per instance + storage |
| **Neptune** | **Graph** | Stores and queries **relationships** between things | "graph, social network, fraud detection, recommendations" | Pay per instance + storage |
| **Redshift** | **Data warehouse (OLAP)** | Petabyte-scale SQL **analytics** on structured data | "data warehouse, analytics, BI, petabytes" | Pay per node (or serverless usage) |

### Why similar ones exist

- **RDS vs Aurora:** both relational and managed. Aurora is AWS's own engine redesigned for the cloud, so it exists for customers who want higher performance and availability and will pay more.
- **DynamoDB vs DocumentDB:** both NoSQL. DynamoDB is AWS's own serverless key-value design. DocumentDB exists so **MongoDB apps can move to AWS without rewriting**.
- **DynamoDB vs ElastiCache:** DynamoDB is a **primary database** (durable). ElastiCache is a **cache** in front of one (speed, not storage of record).
- **RDS vs Redshift:** RDS = **OLTP** (many small transactions, e.g. orders). Redshift = **OLAP** (few huge analytical queries).

### Must-knows
- **RDS Multi-AZ** = standby in another AZ for **availability/failover**. **Read Replicas** = extra copies for **read performance**. Different purposes.
- **Shared responsibility:** on RDS, AWS patches the OS and the database engine. You manage users, security groups, data. A database installed on EC2 is fully yours to patch.
- Moving databases to AWS uses **DMS/SCT** (Migration category).

**Trap pairs**
- **RDS vs Aurora:** "MySQL/PostgreSQL compatible, better performance, 6 copies" = Aurora. Plain "managed relational" = RDS.
- **DynamoDB vs RDS:** serverless NoSQL key-value vs SQL tables.
- **ElastiCache vs DynamoDB:** cache vs durable database.
- **Redshift vs Athena:** Redshift = a warehouse you load data into. Athena (Analytics) = queries data directly in S3.

**Check 1.** An app needs a serverless NoSQL database with single-digit millisecond responses at any scale. → **DynamoDB**
**Check 2.** A company wants to migrate its MongoDB application to AWS with minimal changes. → **DocumentDB**
**Check 3.** Reporting queries are slowing the production RDS database. → **Read replica**

---

## One-page recap

| Need | Pick |
|---|---|
| Object storage, backups, static website | S3 |
| Cheapest long-term archive | S3 Glacier (Deep Archive) |
| Disk for one EC2 instance | EBS |
| Shared Linux file system, multi-AZ | EFS |
| Windows file share / HPC file system | FSx |
| Managed SQL database | RDS |
| High-performance MySQL/PostgreSQL-compatible | Aurora |
| Serverless NoSQL key-value | DynamoDB |
| In-memory cache | ElastiCache |
| MongoDB-compatible | DocumentDB |
| Graph / relationships | Neptune |
| Data warehouse / analytics | Redshift |
