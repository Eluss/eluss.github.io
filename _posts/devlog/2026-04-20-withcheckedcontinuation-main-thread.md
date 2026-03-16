---
layout: devlog
title: "withCheckedContinuation Runs Synchronously"
category: devlog
tags: [swift, concurrency, async, ios]
---
`withCheckedContinuation`'s closure runs synchronously on the caller's thread before the task suspends. From `@MainActor`, that's the main thread.

```swift
@MainActor
func bad() async -> Int {
    await withCheckedContinuation { continuation in
        print(Thread.isMainThread) // true
        Thread.sleep(forTimeInterval: 2) // freezes UI for 2s
        continuation.resume(returning: 1)
    }
}
```

The task never gets a chance to suspend until `resume()` is called. While the closure blocks, the main run loop is stuck — no UI updates, no timers, nothing.

**Fix: dispatch the blocking work, return immediately:**

```swift
@MainActor
func fixed() async -> Int {
    await withCheckedContinuation { continuation in
        DispatchQueue.global(qos: .userInitiated).async {
            Thread.sleep(forTimeInterval: 2) // background thread
            continuation.resume(returning: 1)
        }
        // closure returns here → task suspends → main thread is free
    }
}
```

The closure exits right away, the task suspends, and the main thread is released. The continuation resumes from the background queue when the work finishes.

The rule: inside a continuation closure, only schedule work — never do it.
