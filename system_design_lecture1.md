# ML System Design & Data Engineering — Lecture 1
> Topics: System Design · Scaling · Load Balancer · DB Replication · CDN · Interview Frameworks

---

## 1. What is ML System Design?

> **The intersection of System Design and Data Engineering**

| Discipline | Description |
|---|---|
| **System Design** | Architecting software systems — how components communicate, scale, and stay reliable. |
| **Data Engineering** | Pipelines, storage, and processing that feed models with the right data. |

> 💡 In Machine Learning, both must work together — models are only as good as the infrastructure serving them.

---

## 2. Scaling Strategies

> **Two ways to handle increased load — scale up or scale out**

### ⬆️ Vertical Scaling
Upgrade a single machine to a more powerful one. Simpler but has an upper limit.

```
[ 1 TB / 8 GB ]  →  [ 4 TB / 32 GB ]
```

### ↔️ Horizontal Scaling
Add more machines of the same spec. Enables near-infinite scale-out.

```
[ 1 TB / 8 GB ]  →  [ 1 TB / 8 GB ] + [ 1 TB / 8 GB ] + [ 1 TB / 8 GB ]
```

| Strategy | Pros | Cons |
|---|---|---|
| **Vertical** | Simple, no code changes | Hard ceiling on resources |
| **Horizontal** | Near-infinite scale | More complex to manage |

---

## 3. Load Balancer & Routing

> **Sits at a public IP and distributes traffic across private-IP servers**

```
User (Vaibhav) ──→ Load Balancer (Public IP) ──→ IP1 (A–M)
                                              ──→ IP2 (N–X)  ← Vaibhav routed here
                                              ──→ IP3 (Y–Z)
```

Users are sharded by name range or hash:

| Server | Name Range | Note |
|---|---|---|
| IP1 | A – M | All users whose name starts A through M |
| IP2 | N – X | e.g. Vaibhav → V → routed here |
| IP3 | Y – Z | All users whose name starts Y through Z |

> 💡 `user name = Vaibhav` → starts with **V** → goes to **IP2**

---

## 4. Database Replication — Lower Write, Heavy Read

> **Master-slave (primary-replica) pattern to separate reads and writes**

```
Write ──→ Master DB ──→ replicates to ──→ Slave DB 1  ┐
                                      ──→ Slave DB 2  ├─ serve Read requests
                                      ──→ Slave DB 3  ┘
```

| Component | Role |
|---|---|
| **Master DB** | Receives all write operations |
| **Slave DBs** | Serve all read requests; hold replicas of master data |
| **TTL (Time To Live)** | Controls how long a slave's data is considered fresh before re-syncing |
| **Cache (RAM)** | Memcache/Redis sits in front of reads for ultra-fast responses |
| **Eviction Algorithm** | Decides which cache entries to drop when RAM is full (e.g. LRU, LFU) |

> 💡 Heavy read systems benefit the most — writes go to one place, reads are spread across many replicas.

---

## 5. Full System Architecture

> **Load balancer + app servers + DB tier with caching — combined**

```
User      ──→  Load Balancer  ──→  App Server IP1  ──→  Cache (RAM)  ──→  DB Replicas
(Vaibhav)      (Public IP)    ──→  App Server IP2                    ──→  Master DB
                              ──→  App Server IP3
```

---

## 6. CDN — Content Delivery Network

> **Static assets cached at geographically distributed edge servers**

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

> **A structured 6-step approach for system design interview questions**

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

> **Extends system design with 7 ML-specific stages**

| Step | Phase | Description |
|---|---|---|
| 1 | **Functional requirements** | Clarify the ML problem — classification, ranking, generation? |
| 2 | **Convert to AI/ML query** | Map business requirement to an ML objective (loss function, metric) |
| 3 | **Data** | Sources, labelling strategy, pipelines, feature engineering |
| 4 | **Model** | Architecture choice and justification — why this model? |
| 5 | **Evaluate** | Offline metrics, online A/B testing, bias/fairness checks |
| 6 | **Deployment & serving** | Batch vs real-time, latency constraints, infrastructure |
| 7 | **Monitoring** | Data drift, model drift, alerting, retraining triggers |

> 💡 Steps 1–3 mirror the System Design framework. Steps 4–7 are ML-specific additions.

---

*ML System Design & Data Engineering · Lecture 1 Notes*