# Scalable Systems — Consistency vs Stability

> !! DISCLAIMER !! this is an OCR of my manual notes, might not be perfect. Check the original notes:
> <https://github.com/romain325/romind_dataholder/blob/main/notes/scalable_sys_1.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/scalable_sys_2.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/scalable_sys_3.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/scalable_sys_4.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/scalable_sys_5.jpg>

---

## Definitions

| Term            | Definition                                                      |
| --------------- | --------------------------------------------------------------- |
| **Scalable**    | System that can meet increasing demand while staying responsive |
| **Consistency** | If all members have the same view of a state at a given time    |
| **Available**   | If it remains responsive despite any failure                    |

### Performance ≠ Scalability

- Performance → optimizes **response time**
- Scalability → ability to **handle load**

```
Throughput
    │  ╱ linear scalability (utopic)
    │ ╱
    │╱────── effect of contention
    └──────────────────────────→ concurrency
```

> → Performance lowers the graph but shift follows a smoother nb of req → scalability

---

## Consistency Problem

> **"If you have 2 contradictory information but we don't care about this precise information, then we don't care"**

- If the need for details about those → provide more information to be sure that we can decide what is the proper state we want to be in
- We can add a **third observer** as a medium → resolve, merge, deduct desired state
- Or we can just decide to **trust one part of the system** — especially used in sharding

### The Inevitable Problem

→ We have to accept that a transfer of information separated by space **always implies a short (or not) amount of time** where two parts of the system are not in sync. There is time required to **reach consensus**.

> **Reality is Eventually Consistent** — the receiver is always dealing with stale data.

---

## ACID

> **Atomicity, Consistency, Isolation, Durability**

---

## Types of Consistency

| Type                     | Description                                                                                        |
| ------------------------ | -------------------------------------------------------------------------------------------------- |
| **Eventual Consistency** | Will get back to a state of consistency — when 1 thing causes another, it will be processed before |
| **Causal**               | When 1 thing causes another, it will be processed in causal order                                  |
| **Sequential**           | All items are processed in order but **can't parallelize**                                         |

### Strong Consistency _(used by monoliths for example)_

> "An update needs all node agreements before being true"

→ We can achieve via **Locks** — but that means there is ONE responsible (one DB for ex) → which means the "locked" isn't distributed.

> ⚠️ **Always visualize those systems as humans interacting — it makes things much easier**

---

## Good example for Eventual Consistency: SVC like git

---

## The Effect of Contention

> When 2 things compete for 1 limited resource → only one winner, others are forced to wait

When number of competitors increases → **time to free ↑**

### Amdahl's Law

→ Defines **max improvement** gained by parallelization

```
Throughput
    │  ╱ linear scalability (utopic)
    │ ╱
    │╱────── effect of contention
    └──────────────────────────→ concurrency
```

> There is a **limitation to capacity to parallelize** because at one time you'll need a resource that is locked.

---

## Effect of Coherency Delay _(crosstalk / gossip)_

Each node of the system will send messages to each other informing of changes. The time to sync is called **coherency delay**.

> More nodes = more delay

---

## Gunther's Universal Scalability Law _(built upon Amdahl)_

```
Throughput
    │ ╱ linear (utopic)
    │╱
    │  ╲── Contention + Coherence
    └──────────────────────────→ concurrency
```

> Cost of coordination can **exceed benefits**

---

## Laws of Scalability _("The Mythical Man Month" problem)_

We have to perfectly balance to accept those law limitations.

### Reactive System ponders with

- **Contention** → isolating locks, eliminating transactions, avoid blocking operations
- **Coherency delay** → embrace eventual consistency, build in autonomy

---

## CAP Theorem

> A distributed system **cannot provide more than 2** of:
>
> - Consistency
> - Availability
> - Partition Tolerance

```
          Partition Tolerance
              /\
             /  \
         AP /    \ CP
           /  ●   \
          /────────\
    Availability    Consistency
         CA
```

> When we want more of one, we have to sacrifice the opposite.

---

## Partition Tolerance

> "Systems continue to operate despite an arbitrary number of messages being dropped/delayed by the network"

→ _Utopic_ → no system is safe from partitions

### How to deal with partitions

**AP** → Sacrifice consistency, allow writes to both sides of the partition → will need a merge solution when resolving the part.

**CP** → Sacrifice Availability, disable / terminate 1 side of the part → some part of the system will be unavailable.

> Never really one or another — **balance the 2 concerns** and maybe favor one.

---

## Strong Consistency with Scalability?

### Isolating Contention

→ Reduce crosstalk; when unable to eliminate it, **isolate them**

- Locks with a broad scope create contention → use **smaller scopes** (Row/Record locks)
- **Sharding** → limit scope of contention + less crosstalk — done within application and DB

### Sharding for Consistency (Strong)

→ Application level (not limited by DB features), limit communication

**Partition of entities / actors in the domain according to their ID**

```
┌─────────────────────┐
│  Node (192.168.1.2) │
│  ┌───────────────┐  │
│  │  Shard A-B    │  │ ← Sharding
│  │ [RepoA][RepoB]│  │
│  └───────────────┘  │
└─────────────────────┘
```

- Each entity exists in **only 1 Shard**
- Each shard exists in **only 1 location** → eliminate distribution concept
- Entity acts as a **consistency boundary**

To work, we have to route message to the appropriate shard → **COORDINATOR**
Entity ID is used to calculate the appropriate shard.

> **Aggregate roots are a good candidate for sharding**

#### Balancing Shards

→ Shard key should provide **even / randomized distribution** → use UUID / hash code
→ Otherwise it creates hotspot and is awful

> ≈ **10x Shards than nodes**

---

## PROS and CONS of Sharding

### Contention in Sharding

→ Not eliminated, **just isolated** — isolate to individual entities
→ In one entity there is contention

- Router/Coordinator is a **source of contention**
  → Primordial to limit their work (precompute, cache, etc.)

### Results

| Property               | Benefit                                                        |
| ---------------------- | -------------------------------------------------------------- |
| **Scalability**        | By distributing shards over multiple machines + good shard key |
| **Strong Consistency** | Isolate operations to specific entities                        |

### CAP Theorem and Shards

→ Primarily **CP** so availability is sacrificed

If a shard goes down → unavailability, but a shard can migrate to another node.

---

## Caching with Shards

→ Sharding helps caching → **only one cache per entity / per cache**

```
GetById(A) ──→ [RootA | Cache] ──→ read in mem-cache
                                          ↑
CancelId(B) ──→ [Entity B | Cache] ──write──→ [DB]
                                    Update cache and DB
```

Very efficient for **read-heavy apps**.

_Especially problematic in distributed systems → Sharding is application-level_

---

## Availability and Scalability

→ CAP Theorem still forces us to choose Consistency or Availability

### CRDT — _Conflict Free Replicated Data Type_

→ Solution based on **async replication** → AP

Still at **application level** — Highly available / Eventually Consistent

```
[Replica] ──update──→              ──merge──→  max(1,2) ──→ ●
                       max(0,1)  ↗
[Replica] ──update──→              ──merge──→  max(1,2) ──→ ●
                       max(0,2)  ↗
[Replica] ──────────→ merge──→ merge──→        max(1,2) ──→ ●
                       max(0,2)  max(1,2)
```

Updates are applied on 1 then copied async — all updates are merged for final state.

### CvRDTS vs CmRDTS

|                         | CvRDTS (Convergent)                                        | CmRDTS (Commutative)             |
| ----------------------- | ---------------------------------------------------------- | -------------------------------- |
| **Cv = Convergent**     | Copy State between replicas → need a way to resolve merges | Copy operations between Replicas |
| **Operations must be:** | Commutative → applied in any order                         | —                                |
|                         | Associative → grouped in any order                         | —                                |
|                         | Idempotent → duplicates don't change result                | —                                |

---

## Example: G-SET (Grow-only Set)

_Their merge op is adding to a set_

```
Rep1 ──(1)──────────→ {1,2} ──────→ {1,2,4} ──→ ●
Rep2 ──(2)──→ {2,4}─→ {1,2,4} ────────────────→ ●
Rep3 ──(2)─→ {2,4} ─→ {1,2,4} ─────────────────→ ●
```

CRDT works with specialized data structures: **Registers, Counters, Sets, Maps**

- They can be nested
- You can create your own if the merge function can be created

---

## PROS AND CONS of CRDT

### PROS

- CRDT are **in-memory data** → entire structure has to fit in memory (small dataset, infrequent updates, need high availability)
- Can be **copied to disk**: helps recovery if a replica fails → in normal use, all data is memory not disk

### CONS

- They don't work with all data types → need appropriate **merge function**
- For delete we need a **"tombstone"** mechanism to virtually remove it: data only gets bigger
- Nothing is ever really deleted → why we want **infrequent updates**

---

## How to Choose between Consistency or Availability?

→ Decision made at **business level** — Domain experts and PO

- Also depends on the part of the situation — **all is balance**
- Factor the **revenue impact** between unavailability or eventual consistency

---

## Sources

- [perfdynamics.com/Manifesto/USL_scalability.html](http://perfdynamics.com/Manifesto/USL_scalability.html)
