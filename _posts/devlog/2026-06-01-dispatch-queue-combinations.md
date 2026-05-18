---
layout: devlog
title: "GCD Queue Combinations: What Serial/Concurrent and Sync/Async Actually Control"
category: devlog
tags: [swift, gcd, concurrency, threading, ios]
---
Two independent axes, four combinations. They're easy to conflate because the names sound similar, but they control completely different things.

**Serial vs. concurrent** — how many tasks the *queue* runs at once.  
**Sync vs. async** — whether the *caller* blocks until the task finishes.

```swift
let serial     = DispatchQueue(label: "serial")
let concurrent = DispatchQueue(label: "concurrent", attributes: .concurrent)
```

---

**Serial + sync** — caller blocks, queue runs one task at a time.

```swift
serial.sync { work() } // caller waits; next line runs only after work() returns
serial.sync { moreWork() }
// work() always finishes before moreWork() starts
```

Useful when you need a result immediately and can afford to block the calling thread. Deadlocks if you call `serial.sync` from a task already running on `serial`.

---

**Serial + async** — caller returns immediately, queue still runs one task at a time.

```swift
serial.async { work() }
serial.async { moreWork() }
// caller continues; work() finishes before moreWork() starts, but caller doesn't wait
```

The classic pattern for thread-safe state mutation without blocking. All mutations are serialized; no locks needed.

---

**Concurrent + sync** — caller blocks until *its* task finishes; other tasks on the queue may overlap.

```swift
concurrentQueue.async { longTask() }  // already running
concurrentQueue.sync  { quickTask() } // caller blocks for quickTask(); longTask() may still be running
```

Rarely the right tool. If you need the result synchronously, you probably want a serial queue or just inline the work.

---

**Concurrent + async** — caller returns immediately, tasks may run in parallel.

```swift
for item in items {
    concurrent.async { process(item) } // all items may process simultaneously
}
```

Maximum throughput for independent work. Order of completion is not guaranteed.

---

**The mental model:**

| | Sync | Async |
|---|---|---|
| **Serial** | blocks caller, ordered | non-blocking, ordered |
| **Concurrent** | blocks caller, may overlap others | non-blocking, may overlap |

Sync/async is about the *submitter's* thread. Serial/concurrent is about the *queue's* parallelism. They compose independently.
