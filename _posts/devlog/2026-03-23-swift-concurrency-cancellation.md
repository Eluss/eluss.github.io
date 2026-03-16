---
layout: devlog
title: "Cancellation-Sensitivity in Swift Concurrency"
category: devlog
tags: [swift, concurrency, async, ios]
---
Task cancellation in Swift concurrency is cooperative. Cancelling a task sets a flag — it doesn't stop execution. Your code has to check.

**The flag:**

```swift
let task = Task {
    for item in largeDataset {
        if Task.isCancelled { break }
        process(item)
    }
}

task.cancel()
```

Without the `isCancelled` check, `task.cancel()` has no effect until the task finishes naturally.

**`checkCancellation()` as an early exit:**

```swift
func processAll(_ items: [Item]) async throws {
    for item in items {
        try Task.checkCancellation() // throws CancellationError if cancelled
        await process(item)
    }
}
```

`checkCancellation()` throws `CancellationError`, which propagates up and unwinds the call stack. Useful when you're deep in a chain and don't want to thread a return value back up manually.

**Structured propagation:**

Cancellation flows down the task tree automatically. Cancelling a parent cancels all its children.

```swift
let parent = Task {
    async let a = fetchUsers()
    async let b = fetchPosts()
    return try await (a, b) // cancel parent → both children cancelled
}

parent.cancel()
```

Child tasks created with `async let` or inside `TaskGroup` inherit cancellation. Unstructured tasks (`Task { }`) do not — they're detached from the parent's cancellation scope.

**Cancellation-sensitive suspension points:**

`try await Task.sleep(for:)` throws on cancellation:

```swift
func poll() async throws {
    while true {
        try await Task.sleep(for: .seconds(5)) // exits immediately when cancelled
        await refresh()
    }
}
```

`URLSession.data(for:)` also responds to cancellation — it cancels the underlying network request and throws. Most system async APIs that involve waiting are cancellation-sensitive.

**`withTaskCancellationHandler` for non-async cleanup:**

When you're wrapping a callback-based API, the async body may be suspended and never reach a `checkCancellation` call. Use this to hook cancellation explicitly:

```swift
func fetchData() async throws -> Data {
    try await withTaskCancellationHandler {
        try await withCheckedThrowingContinuation { continuation in
            let request = legacyFetch { result in
                continuation.resume(with: result)
            }
        }
    } onCancel: {
        request.cancel() // called immediately when task is cancelled
    }
}
```

The `onCancel` handler runs synchronously on cancellation, from whatever thread cancelled the task. Keep it short and thread-safe.
