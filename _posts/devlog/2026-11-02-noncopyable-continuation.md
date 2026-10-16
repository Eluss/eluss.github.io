---
layout: devlog
title: "Swift 6.4: Continuation Makes Double-Resume a Compile Error"
category: devlog
tags: [swift, concurrency, swift6, noncopyable]
---

Until now, bridging callbacks to `async` meant picking your poison. `UnsafeContinuation` costs nothing, but resuming twice is undefined behavior and forgetting to resume hangs the task forever, silently. `CheckedContinuation` catches both mistakes, but only at runtime, and it pays for that with an allocation and atomic operations. SE-0528 adds a third option, a `~Copyable` `Continuation`:

```swift
let data = try await withContinuation(of: Data.self, throwing: NetworkError.self) { continuation in
    client.fetch { result in
        continuation.resume(with: result)
    }
}
```

Every `resume` is `consuming`, so once you've called it the binding is gone:

```swift
func complete(with value: String) {
    if let continuation {
        continuation.resume(returning: value)
        continuation.resume(returning: value) // ❌ 'continuation' used after consuming use
    }
}
```

Dropping it without resuming hits a `deinit` that calls `fatalError`. A hang you'd never notice becomes a crash with a clear message:

```swift
func cancel() {
    self.continuation = nil // 💥 "The continuation was dropped without resuming."
}
```

| | Unsafe | Checked | `Continuation` |
|---|---|---|---|
| Double resume | undefined behavior | runtime trap | **compile error** |
| Missing resume | silent hang | runtime warning | runtime trap |
| Overhead | none | allocation + atomics | none |

Two smaller wins:
- `of:` and `throwing:` put the types at the call site, so you no longer need a `(continuation: CheckedContinuation<Data, Never>) in` annotation.
- Thanks to typed throws, one function covers all three cases. `Never` needs no `try`, `NetworkError.self` gives you a typed error, and `(any Error).self` works like the old `withCheckedThrowingContinuation`.

The catch: a noncopyable value can't be captured by an escaping closure, and it can't be consumed in two places. An API with separate `onSuccess` and `onFailure` callbacks won't compile. For that case, wrap it with `let checked = CheckedContinuation(continuation)`. That's exactly why `Checked` and `Unsafe` aren't being deprecated.

**Takeaway:** make `withContinuation(of:)` your default for new bridging code, and fall back to `CheckedContinuation(c)` only when the callback API makes you capture it in more than one place.
