# ML System Design & Data Engineering
> Lecture Notes · Topics 1–8

---

## 1. What is ML System Design?

The intersection of two disciplines applied to building ML-powered products:

| Discipline | Description |
|---|---|
| **System Design** | Architecting software systems — how components communicate, scale, and stay reliable. |
| **Data Engineering** | Pipelines, storage, and processing that feed models with the right data. |

In Machine Learning, both must work together — models are only as good as the infrastructure serving them.

---

## 2. Scaling Strategies

### Vertical Scaling
Upgrade a single machine to a more powerful one. Simpler but has an upper limit.

```
[ 1 TB / 8 GB ]  →  [ 4 TB / 32 GB ]
```

### Horizontal Scaling
Add more machines of the same spec. Enables near-infinite scale-out.

```
[ 1 TB / 8 GB ]  →  [ 1 TB / 8 GB ] + [ 1 TB / 8 GB ] + [ 1 TB / 8 GB ]
```

---

## 3. Load Balancer & Routing

A load balancer sits at a **public IP** and distributes traffic across multiple **private-IP** servers, each handling a subset of users.

```
User (Vaibhav) ──→ Load Balancer (Public IP) ──→ IP1 (A–M)
                                              ──→ IP2 (N–X)  ← Vaibhav routed here
                                              ──→ IP3 (Y–Z)
```

Users are sharded by name range or hash:

| Server | Name Range | Example |
|---|---|---|
| IP1 | A – M | All users whose name starts A through M |
| IP2 | N – X | All users whose name starts N through X (e.g. Vaibhav → V) |
| IP3 | Y – Z | All users whose name starts Y through Z |

---

## 4. Database Replication — Lower Write, Heavy Read

Separate write and read concerns using a **master-slave (primary-replica)** pattern.

```
Write ──→ Master DB ──→ replicates to ──→ Slave DB 1
                                      ──→ Slave DB 2  } serve Read requests
                                      ──→ Slave DB 3
```

| Component | Role |
|---|---|
| **Master DB** | Receives all write operations |
| **Slave DBs** | Serve all read requests; hold replicas of master data |
| **TTL (Time To Live)** | Controls how long a slave's data is considered fresh before re-syncing |
| **Cache (RAM)** | Memcache/Redis sits in front of reads for ultra-fast responses |
| **Eviction Algorithm** | Decides which cache entries to drop when RAM is full (e.g. LRU, LFU) |

---

## 5. Full System Architecture

Combining the load balancer, app servers, and the DB tier with caching:

```
User  ──→  Load Balancer  ──→  App Server IP1  ──→  Cache (RAM)  ──→  DB Replicas
(Vaibhav)  (Public IP)    ──→  App Server IP2                    ──→  Master DB
                          ──→  App Server IP3
```

---

## 6. CDN — Content Delivery Network

Static assets (images, JS, CSS) are cached at geographically distributed **CDN edge servers**. Users fetch from the nearest node, reducing latency and origin server load.

```
User (Vaibhav) ──→ CDN Server (nearest edge)
                        │
               cache hit? ──yes──→ serve directly ✓
                        │
                        no
                        │
                        └──→ Origin (Load Balancer) ──→ fetch & cache ──→ serve
```

| Scenario | Behaviour |
|---|---|
| **Cache hit** | CDN serves asset directly — fast, no origin request |
| **Cache miss** | CDN fetches from origin, stores a copy, then serves |
| **Benefit** | Lower latency for distributed users; reduced origin load |

---

## 7. Engineering Interview Framework — System Design

A structured **6-step** approach for tackling system design interview questions:

| Step | Phase | Description |
|---|---|---|
| 1 | **Functional requirements** | Clarify the question — what must the system do? |
| 2 | **Non-functional requirements** | Scale, latency, availability, consistency |
| 3 | **Maths estimate** | QPS, storage, bandwidth back-of-envelope calculations |
| 4 | **Data flow / API design** | Endpoints, request/response shapes, contracts |
| 5 | **High-level design** | Component diagram, major services and their relationships |
| 6 | **Detailed design** | Deep dive into bottleneck components |

---

## 8. ML Engineering Interview Framework

A **7-step** framework extending system design with ML-specific stages:

| Step | Phase | Description |
|---|---|---|
| 1 | **Functional requirements** | Clarify the ML problem — classification, ranking, generation? |
| 2 | **Convert to AI/ML query** | Map business requirement to an ML objective (loss function, metric) |
| 3 | **Data** | Sources, labelling strategy, pipelines, feature engineering |
| 4 | **Model** | Architecture choice and justification — why this model? |
| 5 | **Evaluate** | Offline metrics, online A/B testing, bias/fairness checks |
| 6 | **Deployment & serving** | Batch vs real-time, latency constraints, infrastructure |
| 7 | **Monitoring** | Data drift, model drift, alerting, retraining triggers |

---

*ML System Design & Data Engineering · Lecture Notes*