---
layout: devlog
title: "Actor Reentrancy"
category: devlog
tags: [swift, concurrency, actors, ios]
---
Actors prevent concurrent access to their state — but suspension points release the actor. Other tasks can mutate state while you're awaiting.

```swift
actor Cache {
    var store: [String: Data] = [:]

    func load(_ key: String) async throws -> Data {
        if let cached = store[key] { return cached }

        let data = try await fetch(key) // actor released here

        store[key] = data // another task may have already written this
        return data
    }
}
```

Two callers with the same key both pass the `store[key]` check, both suspend on `fetch`, and both write the result. Redundant network requests at best, a race at worst.

**Fix: re-check after the await:**

```swift
let data = try await fetch(key)

if store[key] == nil {
    store[key] = data
}
```

**Fix: track in-flight requests:**

```swift
actor Cache {
    var store: [String: Data] = [:]
    var inFlight: [String: Task<Data, Error>] = [:]

    func load(_ key: String) async throws -> Data {
        if let cached = store[key] { return cached }
        if let task = inFlight[key] { return try await task.value }

        let task = Task { try await fetch(key) }
        inFlight[key] = task
        let data = try await task.value
        store[key] = data
        inFlight[key] = nil
        return data
    }
}
```

One fetch per key, regardless of how many callers race to `load` simultaneously.

The rule: never assume actor state is unchanged across an `await`.
