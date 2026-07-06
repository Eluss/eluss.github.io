---
layout: devlog
title: "Swift 6.4: Task Cancellation Shields"
category: devlog
source: https://developer.apple.com/videos/play/wwdc2026/262/
tags: [swift, concurrency, swift6]
---
SE-0504 adds `withTaskCancellationShield` — a way to run a closure where `Task.isCancelled` always returns `false`, regardless of the outer task's cancellation state.

The problem it solves: cleanup and rollback work that *must* finish even when a task has been cancelled. Without a shield, awaits inside cancelled tasks can skip or short-circuit unexpectedly.

```swift
func closeConnection() async {
    await withTaskCancellationShield {
        await database.close()   // runs to completion even if outer task is cancelled
    }
    // Task.isCancelled reflects the real state again here
}
```

A good one-two punch is pairing it with `defer`:

```swift
func performWork() async throws {
    defer {
        withTaskCancellationShield {
            cleanup()   // deferred cleanup that must not be skipped
        }
    }
    try await doTheWork()
}
```

The shield also prevents cancellation from propagating into child tasks — `async let` bindings and task groups inside the closure won't be automatically cancelled.

**Constraints worth knowing:**
- Keep shielded regions short — finish or roll back work already started, not hide cancellation from long-running operations
- Once the closure returns, the outer task's cancellation state is restored

**Takeaway:** reach for `withTaskCancellationShield` when you have cleanup that cannot be skipped on cancellation — it's the missing piece for reliable resource teardown in structured concurrency.
