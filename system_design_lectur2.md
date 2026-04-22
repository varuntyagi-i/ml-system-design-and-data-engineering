# ML System Design & Data Engineering — Lecture 2
> Topics: Redis · Benchmarking · Heartbeat · Microservices · Monolith · API Gateway · Queues

---

## A. Redis (Cache)

<span style="background-color:#E6F1FB; padding:2px 8px; border-radius:4px;">**Stored as key-value pairs**</span>

- In-memory storage on your server
- Used for **saving** records with fields like: `active / not active`, `time-to-live (TTL)`

**Example — saving user activity:**

```
User 1 → 1 year, 3 days   → active
User 2 → 2 years, 4 days  → active
```

> 💡 We don't need to care about "not active" status — **we only need active status students.**

---

## B. Connection Pooling in Database

<span style="background-color:#EAF3DE; padding:2px 8px; border-radius:4px;">**Blocking Queue — pre-warmed, ready-made connections**</span>

```
[ ][ ][ ][ ][ ][ ][ ][ ]  →→  TCP Connection
```

**Why connection pooling?**

- If a user logs in again, creating a fresh TCP connection is **time-consuming and a heavy task**
- To avoid this overhead, we use **TCP connections stored in a blocking queue**
- Connections are pre-warmed and reused instead of re-created every time

---

## 1. Benchmarking & Back-of-Envelope Estimation

<span style="background-color:#FAEEDA; padding:2px 8px; border-radius:4px;">**Scaler → Dashboard → Programming Competition (Rough estimate)**</span>

**Given:**
- 1K requests/sec
- 1 TB of memory storage

**Calculation:**

```
1 request = 1 byte × 8 bits = 8 bytes

Total requests = 1,000,000,000 (1 billion)
              × 8
             ──────────────────
→ 1B × 8 = 8 × 10⁹ B
→ 8 × 10³ MB
→ 8 GB
```

> 💡 This gives a rough memory estimate for the system under load.

---

## 2. Heartbeat

<span style="background-color:#FAECE7; padding:2px 8px; border-radius:4px;">**Mechanism to check if servers/users are alive**</span>

### ❌ Bad Strategy — Poll every 2 seconds

```
User 1 ──→
User 2 ──→  [ Scaler Server ]  ←── available? (yes/no) every 2 sec
User 3 ──→
```

Not a good strategy — it is expensive for the Scaler server to ping every machine constantly.

### ✅ Better Strategy — Users send heartbeat every 2–5 seconds

```
User 1 ──→
User 2 ──→  [ Scaler Server ]   ← users push heartbeat every 2–5 sec
User 3 ──→
```

To make sure we're **receiving users' heartbeats on regular intervals.**

**Example — current time: `11:34:01`**

| User | Last Heartbeat | Status |
|---|---|---|
| U1 | 11:34:00 | ✅ Active |
| U2 | 11:34:01 | ✅ Active |
| U3 | 11:10:00 | ❌ Not Active |

> 💡 If the last heartbeat timestamp is too old, the user/server is marked **not active**.

---

## 3. Monolith vs Microservice

<span style="background-color:#FAEEDA; padding:2px 8px; border-radius:4px;">**Example — taking an order, processing payment, sending notification**</span>

```
→ Taking your order
→ Processing your payment
→ Sending confirmation notification
```

| Architecture | How it works |
|---|---|
| **Monolith** | All three services running on the **same application server** |
| **Microservice** | Each service running on a **different application server** |

---

## 4. Microservices Architecture

<span style="background-color:#EAF3DE; padding:2px 8px; border-radius:4px;">**Each service runs independently with its own load balancer**</span>

### Example — E-commerce Order Flow

```
Users ──→ LB ──→ [ Order Service ]
               ──→ LB ──→ [ Payment Service ]
                        ──→ LB ──→ [ Notification Service ]
```

Each service has its own **Load Balancer (LB)**, its own **application server**, and can be **scaled independently**.

### Queues in Microservices — Real-world examples

| Service | Pattern |
|---|---|
| YouTube upload | File → Queue → Processing → Running |
| Google Drive upload | File → Queue → Processing → Running |
| AWS S3 → EC2 | S3 → ECS cluster → Processing → Running |

<span style="background-color:#EEEDFE; padding:2px 8px; border-radius:4px;">**💡 Key Idea: Use queues for non-real-time tasks**</span>

- Put all items in a queue
- Whatever things **don't need to be done in real time** → don't do them in real time, do them **async / deferred**
- In microservices, if any service doesn't need to run in real time → **put it in a queue**

---

## 5. Scaler — Full System Design

<span style="background-color:#E6F1FB; padding:2px 8px; border-radius:4px;">**User → Scaler Server → DB Store + Object Store (S3)**</span>

```
User ←──→ [ Scaler Server ] ←──→ [ DB Store      ]
                                  [ Object Store (S3) ]
```

**User record schema:**

| Field | Example Value |
|---|---|
| `user_id` | — |
| `name` | "Vaibhav" |
| `video` | video_link (stored in S3) |

**Database Sharding by follower count:**

```
S3 ──→ replication + data
         │
         ├── Cluster 1:  1 – 10,000    (cool + Ma tier)
         ├── Cluster 2:  10K – 20K     (moderate tier)
         └── Cluster 3:  20K+          (top tier)
```

- Application server → **horizontal scaling**
- Database → **sharding** by follower count range

---

## 6. API Gateway in Microservices

<span style="background-color:#FAECE7; padding:2px 8px; border-radius:4px;">**Single entry point that routes to the right microservice**</span>

```
User 1 ──→
User 2 ──→  [ API Server ]  ──→  LB  ──→  [ View Page Service   ]
User 3 ──→                  ──→  LB  ──→  [ Purchase Service     ]
```

**API Gateway responsibilities:**

| # | Responsibility | Description |
|---|---|---|
| 1 | **Which microservice** | Route the request to the correct service |
| 2 | **Rate limiting** | Throttle excessive or abusive requests |
| 3 | **Security** | Handle authentication & authorization |

---

*ML System Design & Data Engineering · Lecture 2 Notes*