# Distributed Messaging Patterns

> !! DISCLAIMER !! this is an OCR of my manual notes, might not be perfect. Check the original notes:
> <https://github.com/romain325/romind_dataholder/blob/main/notes/messaging_1.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/messaging_2.jpg>

> Reactive Systems need to exist in a **Message Driven Architecture** — Async, non-blocking

- Send **without waiting** for a response; the answer may come back but async
- Sync can be used but in a more relaxed way: **acknowledge receipt** and that process will start later
- Sync should be lead by **domain design**, not technical convenience

### PROS

- Resources (thread, etc.) **freed immediately**
- Reduce contention
- Messages can be **queued** if receiver is offline

### CONS

- Can't use transactions → how can we manage **long-running transactions**?

---

## SAGA Pattern — Transaction in Distributed Systems

> Represented as a **finite state machine**
> Sequence of requirements

- Each action comes with a **compensating action** to revert
- If compensation fails, it **retries until success** → require **idempotence**

### Timeout handling in Saga

In case of timeout, request can be:

- Failed
- Succeeded but reply failed
- Still queued

→ Request MUST have succeeded → play the compensating action "just in case" → require **idempotence**

> Request may still be queued → therefore compensation may complete first
> → Compensation and request must be **commutative**

### Compensation ≠ Rollback

| Compensation                             | Rollback                                 |
| ---------------------------------------- | ---------------------------------------- |
| Apply **on top** of previously completed | **Remove all evidence** of a transaction |
| **Keep evidence** of the original action | —                                        |

---

## The Two Generals Problem

> The impossibility of reaching a **consensus** over an unreliable communication channel

```
[Army A] ──messengers──→ (Enemies) ←──messengers──[Army B]
```

- 2 armies try to coordinate a siege attack
- Messengers travel through enemy territory and may be killed
- The enemy is the **unreliable network**

### Example

- A sends "attack at 2pm"
- B replies "ok"
- But does B know that A received the "ok" message? → Circle **forever**
- → Infinite acknowledgement chain

> **We can never reach 100% consensus**

---

## Delivery Guarantees

> We can't have **"Exactly Once"** delivery — it's impossible

Should satisfy with **"At most once"** or **"At least once"**

### "At Most Once" — 0/1

- **No retry** philosophy → send and that's it, accept potential loss
- If failure occurs → never retry, avoid duplication
- Requires **no storage** of messages

### "At Least Once" — 1/n

- All messages will **eventually be delivered**
- Failure (from message / from ack) will always be retried
- May be delivered **multiple times**, never lost
- Messages need to be **stored**

### "Exactly Once v2" — _Effectively Once_

Not possible in specific cases:

- On network partition or lost message → can't guarantee it was received
- The act of resending on failure creates **potential duplicates**

→ Simulated by using: **"At Least Once" + Deduplication or Idempotence**

```
                         ──Message──→
                         ──Ack──────→ X (fail)
[Sender]                 ──Message──→         [Receiver]
[penny]  ←──Ack──────────────────────         (deduplicate)
    Storage of message
    on both sides
```

---

## MESSAGING PATTERNS

### 2 Profiles

1. Send message to **specialized service** (point-to-point fashion)
2. Each microservice leverages a **pub/sub broker/bus** to decouple

### Case 1 — Point to Point

→ Each service depends on each other **directly**

- API coupling → knows and understands their deps

### Case 2 — Pub/Sub

→ Common **message bus**

- Publisher has **no knowledge** of the subscribers
- Services are coupled to **message format**, not each other
- Complexity really hard to see

> Both can be done **internally** or via a **Message Bus**
