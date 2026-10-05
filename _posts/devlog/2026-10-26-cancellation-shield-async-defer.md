---
layout: devlog
title: "withTaskCancellationShield + async defer: Cleanup That Survives Cancellation"
category: devlog
tags: [swift, concurrency, swift6]
---

Async `defer` (SE-0493) runs cleanup in the same task, and that task may already be cancelled. If the cleanup checks for cancellation, it gets cut short:

```swift
defer { await session.close() }  // close() calls URLSession under the hood → cancelled instantly
```

SE-0504's `withTaskCancellationShield` is the fix. Since `defer` can now `await`, the shield goes straight into it:

```swift
func sync(with server: Server) async throws {
    let session = try await server.openSession()
    defer {
        await withTaskCancellationShield {
            await session.close()  // sees Task.isCancelled == false, runs to completion
        }
    }
    try await session.upload(pendingChanges)
}
```

What the shield actually hides:
- **`Task.isCancelled` (the static one)** returns `false` inside the closure, and so does `Task.checkCancellation()`, which therefore doesn't throw. Once the closure returns, both report the real state again.
- **Cancellation handlers** registered inside the shield don't fire when the outer task is cancelled.
- **Child tasks** (`async let`, task groups) created inside don't inherit the outer cancellation.

What it doesn't hide:
- **`someTask.isCancelled` on a task instance** always reports the real state, because the shield affects what code *inside* the task sees, not the task itself.
- **Explicit cancellation** still works. Calling `group.cancelAll()` inside a shield cancels those child tasks as usual.
- **`Task.hasActiveCancellationShield`** tells you whether you're inside a shield, which is handy for logging "we were cancelled, but kept going".

**Takeaway:** for cleanup that must finish, use `defer { await withTaskCancellationShield { … } }`. Shield only the teardown, never the main work, or cancelling the task stops doing anything.
