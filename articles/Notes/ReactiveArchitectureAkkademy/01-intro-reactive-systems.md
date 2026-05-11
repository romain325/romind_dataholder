# Introduction to Reactive Systems

> !! DISCLAIMER !! this is an OCR of my manual notes, might not be perfect. Check the original notes:
> <https://github.com/romain325/romind_dataholder/blob/main/notes/reac_sys_1.jpg>
> <https://github.com/romain325/romind_dataholder/blob/main/notes/reac_sys_2.jpg>

## Why Reactive?

> **Being able to be responsive all the time**

**"Data at rest"** — The data is not consumed at the time it is injected. It's stored and then consumed later in a batch process.

---

## Case Study: Family Restaurant

- Grew from **1 to 500 locations**, 120 countries
- In need to modernize software → downtime no longer OK, online reservation, etc.
- Actual software: one big **monolith**, everything is tightly coupled
- Online order spike during sports events

### Unresponsive Software

- Gives bad company image — even if it's our 3rd party dependencies and not us, the client doesn't care → we need to isolate
- Bad for a better solution

### Symptoms

- Reservation lookup is slow, slow down at table repartition
- Waiter isn't notified right away when order is ready → it's cold
- Waiter struggles adding order to system
- Customer is tired of the struggle for order → rage quit

### The Goal

- **Scale!** Distribution too
- Consume only the resources necessary for the current load
- Handle failures **discreetly**
- Maintain quality

---

## Reactive Principles

> 📖 [reactivemenifesto.org](https://www.reactivemanifesto.org)

```
┌─────────────────────────────────────────┐
│  ✦ Responsive    → ok timing            │
│  ✦ Resilient     → ok even when failure │
│  ✦ Elastic       → ok when more/less    │
│     Message-driven → async non-blocking │
└─────────────────────────────────────────┘
```

### Responsive

Always answer in a timely manner.

### Resilient

- Resilience allows responsiveness **despite failures**
- Failures are isolated to a single component
- Recovery is delegated to an external component
- **Replication, Isolation, Containment, Delegation** are the keys

### Elastic

- Fewest possible contention and no central bottlenecks
- Scale up and down **as needed**
- Predictive auto-scaling

### Message-Driven

- Async & non-blocking
- Loose coupling, isolation, location transparency
- Resources are consumed **only while active**

---

## Reactive Programming ≠ Reactive Systems

> To apply Reactive to the **architectural level**

- When doing Reactive Programming, you don't especially have a Reactive System
- Rx programming helps break problems into small discrete steps in **async / non-blocking** via callbacks

---

## Reactive Systems and the Actor Model Paradigm

### Message Driven

Abstractions provide **Elasticity and Resilience** → provided by location transparency for example.

### Fundamental Concepts

- All computation occurs **inside of Actors**
- Each actor has an **address**
- Only communicate via **async messages**

---

## Location Transparency

> Communicate **the same way all the time regardless of location** — which means Local OR Remote is the same.

```
┌─────────────────────────────────────────────┐
│                                             │
│   (ACTOR) ──local──→ (ROUTER) ──remote──→ (Actor) │
│               ↓                             │
│            (ACTOR)                          │
│                         ex: location transparency │
└─────────────────────────────────────────────┘
```

### ⚠️ ≠ Transparent Remoting

- ❌ Remote looks like local → **hide transport errors**
- ✅ Local looks like remote → **never hide problems**

---

## Actor Model vs Other Solutions

> Don't need the actor model — can be replaced by → **Load Balancer, Service Registry, Message Bus**
> → Works but directly at mid/large scale, not school-level
