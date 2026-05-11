# CQRS & Event Sourcing

> !! DISCLAIMER !! this is an OCR of my manual notes, might not be perfect. Check the original notes:
> <https://github.com/romain325/romind_dataholder/blob/main/notes/cqrs_es_1.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/cqrs_es_2.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/cqrs_es_3.jpg>

**Command Query Responsibility Segregation**

> 📖 **Reference Book:** _"Reactive Microsystems: Evolution of microservices at scale"_ by Jonas Bonér

> ⚠️ **These solutions are expensive to set up — consider pros & cons!**

---

## State-Based Experience (Traditional)

Traditionally → when data is changed we just **overwrite it**.

In cases we need to know how things happened and how we ended up with a given state (commercial, domain, observability, ...):

> The knowledge of **"everything that happened to achieve that state"** helps us handle human error. When correcting errors we can just locate "what happened and how to revert it".

---

## Event Sourcing

> Said simply: **persist state + changelog ("audit log")**

→ Keep only the logs to capture intent → events represent **intents**
→ We keep all the **journey**, not the destination

### Recovery

→ When we load an entity, we **replay all the events** recreating the final state.

- ⚠️ **Don't replay event side effects** — they already happened!

### Snapshot

→ Sometimes it's long to replay events → periodically **save snapshots**
→ So we can take the latest snapshot and replay events from there.
→ **Optimization, not primordial**

> More than the clarity and auditability, **append-only DB is more efficient**

---

## Evolving the Model

### Most Important Point → INTEGRITY

> Our data exists **because of the log** — they represent the history of the system.
> So: **immutability, no update, no delete, only append** 🔒

### Versioning Events

→ The events themselves may evolve, but immutability blocks it.
→ So we must have a **versioning system for events** (so use a flexible format: JSON / Protobuf)

---

## Command Sourcing

Similar to Event Sourcing — just **persist the Command**.

- Commands are **synchronously saved**, asynchronously processed
- Complexity → must be **idempotent**
  - If fail happens → can get stuck in queue
  - Decoupling so hard that the requester cannot be advised of a failure
  - → Validate command first, **event then**

---

## Read VS Write Models

When DDD + Event Sourcing → events represent Entities / Aggregates — but it gets complicated when doing **cross-aggregate requests**.

→ You can't simply query and must rebuild all of them → awful

If we have an aggregate root being "Reservation", but "ReservationByCustomer" is a really conflicting query:

> The **requirements for READ and WRITE are different** — 1 model may not be enough

---

## COMMAND QUERY RESPONSIBILITY SEGREGATION

### Basic CQRS

```
        Queries ──→ [Read Model]  ──→
[API]                                   [Data Store]
        Commands ──→ [Write Model] ──→
```

→ Optimize for **precise purpose**

### When applied with Event Sourcing

```
        Queries ──→ [Read Model] ──Query──→ [Denormalized  ]  ← called a
[API]                                        [   Store      ]    projection
                                                 ↑ events
        Commands ──→ [Write Model] ──Events──→ [Event Store]
```

**Benefits:**

- Very **flexible**
- Very **optimized** for each purpose
- Can be **polyglot** (different persistence type)
- Creating projections is **easy and retroactive**
- R/W can **evolve independently**

---

## Fine-Grained Microservices

### Models As Microservices

→ Read and Write Model each become their **own microservice**
→ (Even each projection can be a microservice)

> ⚠️ Might not be useful — check your needs. It has a cost, make it worth.
> Be pragmatic: do a **general purpose Read Model**, identify the one source of problem, create a Read Model for that one struggling Query — and that's it.

---

## The Big 3: Consistency, Availability, Scalability (in CQRS view)

### CONSISTENCY

|             | Detail                                    |
| ----------- | ----------------------------------------- |
| **CQRS**    | Same as non-CQRS                          |
| **CQRS/ES** | Can implement specific concern for R or W |

- **Strong consistency on WRITE** (transaction, bd, sharding for opti)
- **READ doesn't need it**

### SCALABILITY

|             | Detail                                                     |
| ----------- | ---------------------------------------------------------- |
| **CQRS/ES** | Independent scaling                                        |
|             | Optimized techniques per profile                           |
|             | Can horizontally scale projections with multiple instances |
|             | Caching is possible on projections                         |

### AVAILABILITY

|             | Detail                                            |
| ----------- | ------------------------------------------------- |
| **CQRS/ES** | Allow independent down (can't Write but can Read) |

---

## AT WHAT COST?

- It can be **complex** but also allows specialized data model instead of bloated 1 entry model
- At one time we would have created a view of the big table — in CQRS it's already split
- **Eventual Consistency is explicit** and we have to acknowledge it
- Lots of objects and classes with Commands, Events, Read Models, etc.
- Maybe **multiple data stores**
- **Support for old event versions**
- Long event story needs **more storage**
