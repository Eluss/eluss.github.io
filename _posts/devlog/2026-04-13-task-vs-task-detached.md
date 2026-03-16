---
layout: devlog
title: "Task vs Task.detached"
category: devlog
tags: [swift, concurrency, async, ios]
---
Both create unstructured tasks. Three things differ.

**Actor isolation**

`Task` inherits the actor context of its creator. `Task.detached` is always nonisolated.

```swift
actor Counter {
    var value = 0

    func increment() {
        Task { value += 1 }          // runs on Counter actor ✓
        Task.detached { value += 1 } // compile error — crosses actor boundary ✗
    }
}
```

**Task locals**

`Task` inherits task-local values from the calling context. `Task.detached` starts with no task locals.

```swift
@TaskLocal static var requestID: String = ""

TaskLocal.$requestID.withValue("abc") {
    Task { print(requestID) }          // "abc"
    Task.detached { print(requestID) } // ""
}
```

**Priority**

`Task` inherits the priority of its creator. `Task.detached` defaults to `.medium` unless specified explicitly.

```swift
Task { }                                    // inherits caller priority
Task.detached { }                           // .medium
Task.detached(priority: .background) { }    // explicit
```

---

When you want fire-and-forget work that's aware of its context — use `Task`. When you explicitly want isolation from the current actor and environment — use `Task.detached`.
