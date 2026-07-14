---
layout: devlog
title: "Swift Testing: Test.cancel()"
category: devlog
tags: [swift, testing, swift6]
---
Swift Testing 2.0 (Swift 6.4) adds `Test.cancel()` — a way to programmatically cancel the currently running test, marking it as cancelled rather than failed.

```swift
@Test func featureRequiresDevice() async throws {
    guard await DeviceCapability.isSupported else {
        Test.current?.cancel()
        return
    }
    // test body runs only when supported
}
```

This is different from `#require` (which records a failure) or throwing a `Skip` (which marks the test skipped). `Test.cancel()` cancels the underlying task — the test shows up as cancelled in the report, signalling "not applicable" rather than "broken."

Because it goes through Swift concurrency cancellation, any `try await` after the call will throw `CancellationError` — so a bare `cancel()` followed by `return` is the safest pattern.

Cancellation also propagates to child tests in a suite:

```swift
@Test func suite() async {
    if ProcessInfo.processInfo.environment["CI"] == nil {
        Test.current?.cancel()
        return
    }
    // child tests run only in CI
}
```

**Takeaway:** use `Test.cancel()` when a test is conditionally irrelevant — not wrong, just not applicable. Reserve `#require` for preconditions that *should* hold, and keep `cancel()` for environment or capability gates.
