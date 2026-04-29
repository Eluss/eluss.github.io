---
layout: devlog
title: "Task.yield() — Cooperative Pause, Not a Background Escape"
category: devlog
tags: [swift, concurrency, async, ios]
---
`await Task.yield()` suspends the current task and lets the scheduler run other pending work before resuming.

```swift
func crunchNumbers() async {
    for i in 0..<1_000_000 {
        process(i)
        if i.isMultiple(of: 1000) {
            await Task.yield() // give other tasks a turn
        }
    }
}
```

Without the yield, a long CPU-bound loop holds the thread uninterrupted. Other tasks in the same cooperative thread pool can starve. Yielding periodically makes the loop cooperative.

**It also acts as a cancellation checkpoint:**

```swift
func crunchNumbers() async throws {
    for i in 0..<1_000_000 {
        try Task.checkCancellation()
        process(i)
        if i.isMultiple(of: 1000) {
            await Task.yield()
        }
    }
}
```

After resuming from `yield`, the task checks cancellation on the next `checkCancellation()` call. Without any suspension point, a cancelled task runs to completion anyway.

---

**Red flags**

`Task.yield()` is not a background escape. On `@MainActor`, the task resumes back on the main thread. The main thread is freed only for the brief window between suspend and reschedule — not long enough for heavy computation to stop being a problem.

There's also no fairness guarantee — if nothing else is waiting, `yield` resumes immediately. It's a hint to the scheduler, not a timed pause.

**The testing trap**

`Task.yield()` sometimes shows up in tests to "flush" unstructured tasks:

```swift
func testSomething() async {
    var result = 0
    Task { result = 42 }
    await Task.yield()
    XCTAssertEqual(result, 42) // passes... sometimes
}
```

This is a code smell. `yield` gives the scheduled task a chance to run, but offers no delivery guarantee — the test is racing the scheduler. It can pass locally and fail on CI under load.

The real fix is to not lose the handle:

```swift
func testSomething() async {
    var result = 0
    let task = Task { result = 42 }
    await task.value
    XCTAssertEqual(result, 42) // deterministic
}
```

If you see `Task.yield()` in a test, it's almost always papering over an unstructured task that should either be awaited directly or replaced with structured concurrency.

The rule: use `yield` to keep CPU-bound loops cooperative and cancellation-responsive. In tests, treat it as a red flag.
