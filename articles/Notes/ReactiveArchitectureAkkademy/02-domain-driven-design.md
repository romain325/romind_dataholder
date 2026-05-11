# Reactive Architecture — Domain Driven Design

> !! DISCLAIMER !! this is an OCR of my manual notes, might not be perfect. Check the original notes:
> <https://github.com/romain325/romind_dataholder/blob/main/notes/ddd_1.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/ddd_2.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/ddd_3.jpg>

> 📖 **To Read:** Eric Evans — _Domain Driven Design_

---

## What is DDD?

- Break large problems into **small problems**
- Allow **expert & dev** to work better together
- Extract technical problems to **not flood domain**

> DDD & Reactive Microservices tend to resolve the same problems.

---

## What is a Domain?

- A **sphere of knowledge** we are trying to model
- The goal is that the **domain expert understands the model**
- The model represents an **understanding of the domain** and we try to implement it
- The software is an **implementation of the domain** — doc is canonical, etc.

### Ubiquitous Language

→ **Dictionary of the words used by the domain expert**

- Allows communication with domain expert **without flooding with technical terms**

---

## [ CASE STUDY ]

Business domains are large and complicated.

### Subdomains

→ Grouping **related ideas** together in a domain.

- Some concepts exist at multiple places, however they **evolve differently** → don't over-abstract domain ideas

> ⟹ **Bounded Context**

### Bounded Context

- Is the **start point of microservice definition**
- How to determine bounded context:
  - **Personas**
  - Change in Ubiquitous Language
  - Look for unnecessary info

---

## [ EVENT FIRST DDD ]

Instead of focusing on **objects** (Food, Reservation, Customer), focus on **activities** (Customer makes a reservation, Food is served).

→ Definition can be made via **Event Storming**

### Domain Activities — Subject Verb Object notation

> Example: "Host Checks Reservations"

Help extract information.

---

## Maintaining the Purity of the Bounded Context

### Anti-Corruption Layer

→ Translate from one context to another — **prevent context leak and coupling**

```
[Bounded Context A] ──→ [Anti-Corruption Layer] ──→ [Bounded Context B]
                              (abstract interface)
                         pure representation of our context
```

- Can also be used to protect when connecting **impure legacy code** to pure bounded context.

---

## Context Maps (graph)

→ Interaction of contexts between each other

**Direct VS Indirect Object:** When multiple objects in a sentence:

> "Bartender collects payment for a Drink Order"
>
> - **Direct:** payment
> - **Indirect:** Drink Order

---

## Domain Activities

### COMMAND

→ Request to perform an action / **Change the state** of a domain

### EVENTS

→ Represent an **action that happened in the past**

- Broadcasted most of the time
- Record a change in the state of the domain

### QUERIES

→ Request **information** about a domain

- **Doesn't alter** the state of the domain

> In a Reactive System, these 3 represent the messages — they form the **API for the bounded context**.

---

## Domain Objects

### Value Object

→ Defined by its **attributes**, **immutable**, can contain business logic

> ⚠️ Messages are implemented as Value Objects

### Entities

→ Defined by a **unique ID**, change attributes not its identifier, can contain business logic
→ Match **actor model**

### Aggregates

→ A **collection of domain objects** around a root entity

```
Person ──→ Phone number
       └──→ Address
```

- Access to data in aggregate must go **through the Root**
- Transactions should **not span** aggregate roots
- Good candidates for **distribution** in Reactive Systems
- Treated as **one unit**

---

## Domain Abstraction

### SERVICES

→ Business logic that **doesn't fit in entity** — can abstract away an anti-corruption layer

- Should be **stateless**
- ⚠️ Too many services leads to **anemic domain**

### FACTORIES _(can be combined)_

→ Abstract the logic of creation

- Domain interface, 1…n concrete implementations
- `CREATE` → full CRUD

### REPOSITORIES _(can be combined)_

→ Retrieve & Update existing objects

- `READ`, `UPDATE`, `DELETE`

---

## Hexagonal Architecture

_(also called **Ports and Adapters**)_

```
                    Infra
   REST API           API
   UI      ┌─────────────────┐
            │   Deps  Domain  │
   Message  └─────────────────┘
   broker        ↑       ↑
   ──────    API=ports   Infra adapts I/O to ports
    PORT      Adapters

  inner don't know outer
  outer depends on inner
```

### Onion view

```
[Infra / API]
  └─→ [Deps]
       └─→ [Domain]  ← PURE
```

- Ensure **separation** of infrastructure from Domain → PURE
- Avoid **bleeding**
- Can be enforced by packages / project structure, etc.
- Allow domain to be **portable**

---

## References

- [domainlanguage.com/ddd](https://domainlanguage.com/ddd)
