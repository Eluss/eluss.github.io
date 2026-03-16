---
layout: devlog
title: "withThrowingTaskGroup"
category: devlog
tags: [swift, concurrency, async, ios]
---
`withThrowingTaskGroup` runs child tasks concurrently and cancels the remaining ones the moment any single child throws.

```swift
let results = try await withThrowingTaskGroup(of: Data.self) { group in
    for url in urls {
        group.addTask { try await URLSession.shared.data(from: url).0 }
    }

    var collected: [Data] = []
    for try await data in group {
        collected.append(data)
    }
    return collected
}
```

All `addTask` closures start immediately and run in parallel. The `for try await` loop collects results as they finish — not in submission order.

**First throw cancels the group:**

```swift
group.addTask { throw URLError(.badURL) } // this one fails
group.addTask { try await slowFetch() }   // gets cancelled
```

The error propagates out of the `for try await` loop, the group cancels remaining tasks, and `withThrowingTaskGroup` rethrows. You don't get partial results.

**Collecting partial results instead:**

If you want to keep whatever succeeded, catch inside `addTask`:

```swift
group.addTask {
    try? await URLSession.shared.data(from: url).0 // nil on failure
}
```

Or use `withTaskGroup` (non-throwing) with a `Result` return type:

```swift
withTaskGroup(of: Result<Data, Error>.self) { group in
    group.addTask { await Result { try await fetch(url) } }
}
```

**`addTaskUnlessCancelled`:**

Skips adding the task if the group is already cancelled — useful when building the task list in a loop after some tasks have already failed:

```swift
for url in urls {
    guard group.addTaskUnlessCancelled(operation: { try await fetch(url) })
    else { break }
}
```
