---
layout: devlog
title: "Swift 6.4: Result(catching:) Finally Takes async Closures"
category: devlog
tags: [swift, concurrency, swift6]
---

`Result(catching:)` has been around since Swift 5, but only for synchronous code. Pretty much every codebase with async code ended up writing the same little extension by hand. SE-0530 adds it to the standard library:

```swift
let result = await Result {
    try await api.fetchProfile()
}
```

It's most useful when you want to keep an error around as a value instead of throwing it right away, for example when you run several requests in parallel and want *each* outcome, not just the first failure:

```swift
async let profile = Result { try await api.fetchProfile() }
async let feed    = Result { try await api.fetchFeed() }

let (p, f) = await (profile, feed)  // one failing doesn't cancel or hide the other
```

Two details in the signature are worth knowing:
- It's `throws(Failure)`, so with a typed-throws closure you get `Result<Profile, APIError>` instead of `Result<Profile, any Error>`.
- It's `nonisolated(nonsending)`. The closure runs on the caller's actor, so calling it from `@MainActor` doesn't hop off the main thread and doesn't need a `Sendable` closure.

It's also back-deployable, so you can delete your own copy of `extension Result { init(catching body: () async throws -> Success) async }` without raising your deployment target.

**Takeaway:** in async code, `await Result { try await … }` is now the built-in way to turn an error into a value. Delete the extension your project wrote by hand.
