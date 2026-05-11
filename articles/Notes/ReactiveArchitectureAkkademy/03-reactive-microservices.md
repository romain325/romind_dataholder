# Reactive Architecture — Reactive Microservices

> !! DISCLAIMER !! this is an OCR of my manual notes, might not be perfect. Check the original notes:
> <https://github.com/romain325/romind_dataholder/blob/main/notes/reac_micro_1.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/reac_micro_2.jpg>

> Monolith and Microservice isn't black or white → **more of a spectrum**

---

## MONOLITHS

> **"Ball of mud"** → No isolation, complex deps, Spaghetti

- Deploy as a single unit
  - Shared DB
  - Deep coupling
  - Sync method calls
  - Long release cycle

- Scaling via multiplying instances — they don't know each other → **DB is the shared truth**

### CLEAN UP

- Divide by domain boundaries
- Through libraries

### PROS vs CONS

| PROS                 | CONS                             |
| -------------------- | -------------------------------- |
| Cross-module refacto | Limited to vertical scale        |
| Consistency          | DB is a scaling limit too        |
| ↓ Deploy process     | Inflexibility                    |
| ↓ Monitor            | Slow Dev                         |
| Easy Scaling         | Error brings whole monolith down |

---

## Service Oriented Architecture (SOA)

- Each boundary represents a context, they **don't share a DB**
- Communication is made via the **service exposed API**
- Can live at the same place but **structurally separated**

→ Pros of Microservice **without the deployment mess**

---

## MICROSERVICES

> Are SOA **independently deployed** → can be sync or async
> Are **independent and self-governing**

- Loose coupling between components
- Rapid / Continuous deploy
- DevOps oriented

### How to Scale?

→ All micros are scaled **independently**
→ Each machine hosts a **subset of the entire system**

### PROS vs CONS

| PROS                               | CONS                                 |
| ---------------------------------- | ------------------------------------ |
| Individual scaling                 | Complex deploy and monitoring        |
| Isolated serious failures          | Cross-service refacto complex        |
| Isolation → decoupling → more flex | Support old APIs                     |
| Multiple platform and language     | Organizational change is challenging |

### Single Responsibility Principle

> "managing account"

- A change to the internals of one should **not change** another microservice
- **Bounded context** is a good starting split-up definition

---

## Principle of Isolation

> Mono → SOA → Micro → **more isolation**

Reactive microservices are isolated in: **State, Space, Time, Failure**

### Isolation of State

→ All access to state is made **through API**
→ Allow internal evolution

### Isolation of Space

→ We should not care where the other microservice is deployed

### Isolation of Time

→ Async and non-blocking — Between micros we expect **"eventual consistency"**

### Isolation of Failure

→ Isolate failure, remain operational, maybe less options but still working

---

## BULKHEADING

Tool to **isolate failures**. Still operational, maybe degraded.

> Example: We want to pay for meal → we need ORDER, PAYMENT, LOYALTY
> If LOYALTY is down → just proceed without it

---

## Circuit Breakers

> Retry can make the load **even worse** in case of stress

**Quarantine** the failing service so it fails fast. Allow the failing service to recover and avoid overload. Multiple algorithms for the attempt reset.

```
                    Reset
         ┌────────────────────┐
         ↓                    │
    (Closed)                  │
    (normally)  ──Trip──→  (Half Open)   * Trip = fail
         ↑                    │
    (Open)   ←──Trip──────────┘
   (fail fast)
    └── Attempt Reset ──→
```

- **Closed** (normal): requests pass through
- **Open** (fail fast): requests fail immediately, allows service recovery
- **Half Open**: attempt reset — test if service recovered

---

## Message Driven Architecture

→ Async non-blocking helps with **Time & Failure isolation**

### Autonomy

→ A micro only guarantees its own behaviour (via API)
→ Provide enough info to resolve conflicts and repair failures

| Aspect                                                  | Detail |
| ------------------------------------------------------- | ------ |
| Only async on external                                  | —      |
| Maintain enough internal state to function in isolation | —      |
| Eventual Consistency                                    | —      |
| Don't direct access                                     | —      |

**PROS:**

- More scalability and availability
- Tolerate more failures
- _Utopic but get as close as possible_

---

## Gateway Services

> One request may require **multiple microservice calls** → complex for the client → client side more complex than backend

```
client → gateway → multiple micros → aggregate multi micros
```

- Can be **specialized by domain**
- Gateway handles micro failures too → **flows isolation**
