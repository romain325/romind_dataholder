# Actor Model

> !! DISCLAIMER !! this is an OCR of my manual notes, might not be perfect. Check the original notes:
> <https://github.com/romain325/romind_dataholder/blob/main/notes/actor_1.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/actor_2.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/actor_3.jpg>

> **Context:** Concurrent & Distributed app handling  
> **Origin:** Carl Hewitt 1973 — model came from physics, early computing optimization

---

## What is an ACTOR?

An Actor is:

- The **computational entity** in the Actor model
- Called via an **address**
- Receives messages and **processes them one at a time**
- Communicates via **messages** to other actors

### Actor can

- Change its own **state**
- Change how it will **handle the next message**
- **Create** sub-actors
- **Send or schedule** messages
- **Kill itself**
- Be **idle** when waiting for the next message

---

## PROS

| Advantage                       | Detail                        |
| ------------------------------- | ----------------------------- |
| **Conceptual Simplicity**       | Split into small problematics |
| **High Parallelism and Safety** | No lock / sync                |
| **Resilient / Distributed**     | —                             |

---

## Actor = State + Behavior + Mailbox

> On the side, there is a **mailbox**

- The mailbox receives messages from other actors and reads them **in order**
- Messages are **immutable**, they define I/O — typically `struct` / `record`

### FIT: DDD → an actor is an entity

**(Command, Event, Query, Result)**

---

## Components of an Actor

### MAILBOX

- Number of messages allowed
- Priority for messages
- Delivery order
- Enqueueing behavior (bounded / ∞)
- **Do not persist**

### STATE

- Can access its state **only during message processing**
- State is requested via messages
- State is **ephemeral** / optionally persistent — but it's just a representation

### BEHAVIORS

- How a message is processed
- Can have **multiple** behaviors but only **one active at a time**
- Example behaviors: update state, create actors, send message, change to other behavior

---

## MUTATION

- State can have any structure
- **Forbid concurrent access and mutation**
- **NEVER share mutable state** — access an actor state directly
- Behavior mutation → a message can tell the actor to change behavior for next messages
  - Allows **FSM design**
  - Can change node / become notify
  - Act as queues for Input and Batch Output

---

## Actor Lifecycle

### Actor Creation

→ When creating → identified as children → Split into easier scope, allow distribution

### Actor Termination

- **"Let it crash"** philosophy
- Terminate themselves or their children
- When killed → kill all children
- Can ask another to kill via messages

### Actor Communication

- Actor address → must only be in state or current message
- Use **reply-to pattern** for response communication
- Can talk to itself
- Can schedule

---

## Example FSM

```
        work
  ┌─────────────────────────────────────────────┐
  │                                             │
Start          finish            stop
  ●──→[Waiting]──→(Processing)──→[Clean up]──→◉
        │              ↑
  Create Actor      Initiate
```

> Each FSM state is an actor behaviour. The behaviour decides which messages are useful or not.

---

## ACTOR SYSTEMS ⚠️ HEAVYWEIGHT

> Have **1 System per logical application**

### Definition

An ensemble of actors in a **hierarchy** with a fixed number of threads for processing the mailboxes.

### Help

- Collaborate
- Break down to small simple pieces
- Sagas
- SAGA pattern → Transactional behaviour

### Best Practice

- Move responsibility to children
- Short-lived actors
- Failure prone to children

### Role of Actor System

`queue`, `dequeue`, `dispatch message`, `schedule`, `log`, `config`

---

## HOW — Actor System Handles Key Concerns

| Concern          | How                                                                |
| ---------------- | ------------------------------------------------------------------ |
| **Concurrency**  | State exists only in actors → no lock, no reconciliation, no share |
| **Distribution** | Interactions are async and non-blocking                            |
| **Fault**        | Communication + Failure handling → Circuit breakers                |

---

## References

- [reactivemenifesto.org](https://www.reactivemanifesto.org)
