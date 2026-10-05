---
layout: devlog
title: "Swift 6.4: You Can Finally await Inside defer"
category: devlog
tags: [swift, concurrency, swift6]
---

SE-0493 lifts the old "no `await` in `defer`" rule. In an `async` function, the cleanup can now be async too:

```swift
func sync(with server: Server) async throws {
    let session = try await server.openSession()
    defer { await session.close() }

    try await session.upload(pendingChanges)
    try await session.download(remoteChanges)
}
```

Before this, you had two bad options. You could copy `await session.close()` onto every exit path, including each `throw` and early `return`, and hope nobody adds a new one later. Or you could write `defer { Task { await session.close() } }`, which starts an unstructured task that outlives the function, runs at some later point, and isn't guaranteed to finish before the next session opens.

How it works:
- The `defer` body is awaited implicitly when the scope exits. You only write `await` on the calls inside it, not on the `defer` itself.
- The function suspends only at the `await`s written inside the body, so a synchronous `defer` behaves exactly as before.
- It only works in an `async` context. `defer { await g() }` in a sync function is still a compile error. Inside a closure, it makes the closure inferred as `async`.
- Cancellation gets **no special treatment**. If the task was cancelled (which is often why you're exiting), the awaited cleanup runs in that cancelled task. Any cancellation-aware call inside it, like `Task.sleep`, a `URLSession` request or a `try Task.checkCancellation()`, can bail out early.

**Takeaway:** async cleanup goes in `defer` now, not in a `Task { }` inside `defer`. Keep in mind that a cancelled task stays cancelled inside `defer`, so cleanup that must finish needs a cancellation shield (next week).
