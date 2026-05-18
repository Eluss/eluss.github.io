---
layout: devlog
title: "OSAllocatedUnfairLock — A Lock That Owns Its State"
category: devlog
tags: [swift, concurrency, threading, ios]
---
`OSAllocatedUnfairLock` is a low-level mutex added in iOS 16 / macOS 13. Unlike `NSLock`, it's a **struct** — but you can copy it freely because the actual lock storage is heap-allocated (hence "allocated"). The struct is just a handle.

The defining design choice: the protected value lives *inside* the lock, not alongside it. You get it only by going through `withLock`:

```swift
let counter = OSAllocatedUnfairLock(initialState: 0)

counter.withLock { value in
    value += 1
}

let snapshot = counter.withLock { $0 }
```

Compare that to `NSLock`, where nothing stops you from reading the value without locking:

```swift
var count = 0
let lock = NSLock()

lock.lock()
count += 1
lock.unlock()

print(count) // compiles fine — lock is bypassed entirely
```

**"Unfair"** means threads don't get the lock in arrival order. The OS picks freely, which avoids the overhead of maintaining a queue. In practice this gives it much lower latency than `NSLock` for uncontended cases.

**It's non-recursive.** Calling `withLock` again from inside `withLock` on the same lock is a deadlock, not a re-entry.

**`Sendable` conformance**

`OSAllocatedUnfairLock` is `Sendable` when `State: Sendable`. Because the state is only reachable through `withLock`, the compiler can verify that sharing the lock across concurrency boundaries is safe — no `@unchecked Sendable` needed.

This makes it a clean fit for Swift 6 strict concurrency. A class that holds mutable state and an `NSLock` forces you to suppress the warning yourself:

```swift
// Swift 6 — compiler complains
class Counter: Sendable {
    private var value = 0        // ⚠️ stored property 'value' of 'Sendable'-conforming class is mutable
    private let lock = NSLock()
}
```

With `OSAllocatedUnfairLock` the state is inside the lock, so the compiler is satisfied without any suppression:

```swift
final class Counter: Sendable {
    private let lock = OSAllocatedUnfairLock(initialState: 0) // ✅

    func increment() { lock.withLock { $0 += 1 } }
    func value() -> Int { lock.withLock { $0 } }
}
```

**Takeaway:** reach for `OSAllocatedUnfairLock` when you need a fast, struct-friendly mutex and want the compiler to make it hard to access shared state without holding the lock.
