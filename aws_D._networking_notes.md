# D. NETWORKING (12 services, 2 categories)

Two categories, two questions:
- **Networking & edge:** "Build the network, name it, speed it up, and link networks inside AWS."
- **Hybrid connectivity:** "How do my own offices, data centers and users connect to AWS?"

---

## 9. NETWORKING & EDGE

| Service | Job | What it is | Why it exists separately | Cost logic |
|---|---|---|---|---|
| **VPC** | **Build** | Your private virtual network in a Region: subnets, route tables, gateways, security | Everything you launch needs a network, and it must be isolated by default | **VPC itself free**. NAT gateways, endpoints and data transfer cost |
| **Route 53** | **Name** | **DNS** (name to IP), domain registration, health checks, routing policies | Users type names, not IPs. Also routes traffic by latency, geography, failover | Per hosted zone + queries |
| **CloudFront** | **Speed (cache)** | **CDN**: caches content at **edge locations** close to users | Distance means latency. Caching static content near users fixes it | Pay per data transfer out and requests |
| **Global Accelerator** | **Speed (path)** | **Two static anycast IPs** as one entry point. Sends TCP/UDP traffic over the **AWS global network** to the best healthy endpoint. **No caching** | Some traffic (gaming, VoIP, non-HTTP) can't be cached but still needs speed, fixed IPs and fast failover | Hourly fee + data transfer premium |
| **Transit Gateway** | **Link (many)** | **Hub** that connects many VPCs and on-premises networks | VPC peering is **non-transitive**, so many VPCs need a messy mesh. A hub fixes it | Per attachment hour + data |
| **PrivateLink** | **Link (one service)** | Private connection to a **single service** (AWS or third-party) over the AWS network, **no internet** | Sometimes you only need to reach one service, not connect two whole networks | Per endpoint hour + data |

### VPC essentials (high-yield)
- **Subnet** = part of a VPC, lives in **one AZ**. **Public subnet** has a route to an **Internet Gateway**. **Private subnet** does not.
- **NAT Gateway** = lets private subnets reach the internet **outbound only**. It sits in a **public** subnet.
- **Security Group** = instance level, **stateful**, **allow rules only**.
- **Network ACL (NACL)** = subnet level, **stateless**, **allow and deny** rules. Block a specific IP here.
- **VPC peering** = connects two VPCs. **Not transitive.**
- **VPC endpoint** = reach AWS services without the internet. *Gateway endpoint* for S3 and DynamoDB. *Interface endpoint* (powered by PrivateLink) for others.

### The three "link" services differ by reach
| | What it connects | Direction | Use |
|---|---|---|---|
| **VPC peering** (VPC feature) | Two VPCs | Both ways, **not transitive** | Simple 1-to-1 |
| **Transit Gateway** | Many VPCs + on-premises | Hub and spoke | Large setups |
| **PrivateLink** | **One service** to consumers | One way, service only | Expose or reach a single service privately |

### The three "edge" services differ by what they do
| | Does | Typical question |
|---|---|---|
| **Route 53** | Resolves names, routes by policy | "DNS", "failover routing", "latency routing" |
| **CloudFront** | **Caches** content near users | "Static content, reduce latency, CDN" |
| **Global Accelerator** | Fast **path** + static IPs, no cache | "Static anycast IPs, TCP/UDP, fast failover" |

**Cue words**
- "private network, subnet, route table" → **VPC**
- "DNS, domain name, routing policy, health check" → **Route 53**
- "CDN, cache, edge locations, static content" → **CloudFront**
- "static anycast IP, TCP/UDP, gaming, fast regional failover, no caching" → **Global Accelerator**
- "hub, connect many VPCs, hub-and-spoke" → **Transit Gateway**
- "private access to a service, no internet, no peering" → **PrivateLink**

**Trap pairs**
- **CloudFront vs Global Accelerator:** both use the AWS edge. CloudFront **caches**. Global Accelerator **doesn't**, it gives fixed IPs and a faster path.
- **Peering vs Transit Gateway vs PrivateLink:** two networks vs many networks vs one service.
- **Security group vs NACL:** instance and stateful vs subnet and stateless. "Deny a specific IP" always means NACL.
- **NAT Gateway:** outbound only, public subnet. An Internet Gateway makes a subnet public.
- **Route 53 vs CloudFront:** one answers *where is this name*, the other *serves cached content*.z

**Check 1.** A global gaming service needs two fixed IP addresses and fast failover between Regions; nothing needs caching. → **Global Accelerator**
**Check 2.** A company has 25 VPCs and an on-premises network and wants one central hub instead of many peering links. → **Transit Gateway**
**Check 3.** Private-subnet instances must call an AWS service without using the internet. → **PrivateLink / VPC endpoint**

---

## 10. HYBRID CONNECTIVITY: "How do my world and AWS meet?"

Four different answers to "bridge my world to AWS": a **line**, a **tunnel for people**, a **box**, and a **storage bridge**.

| Service              | Bridge type              | What it is                                                                                                                                                                                     | Why it exists separately                                                                                      | Cost logic                                                     |
| -------------------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| **Site-to-Site VPN** | **Tunnel for a network** | **Encrypted** (IPsec) tunnel over the **public internet** between your office/data center and your VPC. Set up in minutes to hours                                                             | Quick, cheap way to connect a whole site                                                                      | Per connection hour + data transfer                            |
| **Client VPN**       | **Tunnel for people**    | Individual users (remote workers) connect securely to AWS or on-premises resources                                                                                                             | Site-to-Site connects networks. People need their own access                                                  | Per endpoint association + per connection hour                 |
| **AWS VPN**          | **Family name**          | The umbrella for **Site-to-Site VPN + Client VPN**                                                                                                                                             | Shown separately in your sheet as the group heading                                                           | See the two above                                              |
| **Direct Connect**   | **Dedicated line**       | **Private, dedicated** network connection from your site to AWS. **Not over the public internet.** Consistent latency, high bandwidth. **Not encrypted by default**. Takes **weeks** to set up | VPN rides the internet, so performance varies. Some workloads need steady, high-volume links                  | Port-hour + data transfer (lower than internet rates)          |
| **Outposts**         | **A box**                | **AWS-owned infrastructure installed in your data center**, same AWS services and APIs                                                                                                         | For workloads that must stay on-site (low latency to local machines, data residency) but should still use AWS | Paid for the installed capacity; AWS installs and maintains it |
| **Storage Gateway**  | **Storage bridge**       | A gateway appliance that lets **on-premises apps use cloud storage** (S3, Glacier, EBS) with local caching. File, Volume and Tape types                                                        | Existing on-prem apps expect local file/tape/disk protocols, not S3 APIs                                      | Pay per GB stored and gateway usage                            |
|                      |                          |                                                                                                                                                                                                |                                                                                                               |                                                                |

### VPN vs Direct Connect (the classic comparison)
| | Site-to-Site VPN | Direct Connect |
|---|---|---|
| Path | Public internet | Private dedicated line |
| Encrypted | **Yes** | **No** by default |
| Setup | Minutes to hours | Weeks |
| Performance | Varies | **Consistent** |
| Cost | Low | Higher |

Many companies use **Direct Connect for the main link and a VPN as backup**, or VPN over Direct Connect for encryption.

**Cue words**
- "encrypted tunnel over the internet, quick, connect office to VPC" → **Site-to-Site VPN**
- "remote employees, individual users connect" → **Client VPN**
- "dedicated private connection, consistent latency, not through the internet" → **Direct Connect**
- "AWS infrastructure in our own data center, same APIs, local processing" → **Outposts**
- "on-premises apps store data in S3, hybrid storage, replace tape" → **Storage Gateway**

**Trap pairs**
- **Site-to-Site VPN vs Client VPN:** a site/network vs individual users.
- **VPN vs Direct Connect:** internet and encrypted vs private and consistent.
- **Outposts vs Storage Gateway:** a whole AWS rack on-site vs just a storage bridge.
- **Storage Gateway vs Snow Family / DataSync** (Migration): ongoing hybrid storage vs moving data once.

**Check 1.** A company needs a quick encrypted connection from its office to AWS over the internet. → **Site-to-Site VPN**
**Check 2.** A bank needs a private, consistent, high-bandwidth link to AWS, and accepts a longer setup time. → **Direct Connect**
**Check 3.** A factory needs AWS services running on its own premises for very low latency to local machines. → **Outposts**

---

## One-page recap

| Need | Pick |
|---|---|
| Private isolated network in AWS | VPC |
| DNS / routing by latency, geography, failover | Route 53 |
| Cache content near users | CloudFront |
| Static anycast IPs, TCP/UDP acceleration | Global Accelerator |
| Hub for many VPCs + on-prem | Transit Gateway |
| Private access to one service | PrivateLink |
| Encrypted tunnel over the internet | Site-to-Site VPN |
| Remote users connecting | Client VPN |
| Dedicated private line | Direct Connect |
| AWS hardware in your data center | Outposts |
| On-prem apps using cloud storage | Storage Gateway |
