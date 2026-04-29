---
layout: devlog
title: "Task Scheduling Order and Actor Isolation"
category: devlog
tags: [swift, concurrency, async, ios, mainactor]
---
This is an academic example to illustrate scheduling order — not a pattern worth copying. Using `[weak self]` with `guard let self` inside a `Task` is questionable in practice: `guard let self` immediately promotes the weak reference back to a strong one, so you're not really guarding against anything meaningful for the duration of the task body. In real code, prefer structured concurrency, just let the task capture `self` strongly or use weak and check if it exists after suspension points.

That said, the scheduling behavior this reveals is worth understanding.

```swift
class ViewModel {
    func work() {
        Task { [weak self] in
            guard let self else { print("No self"); return }
            doWork()
        }
    }
}

var vm: ViewModel? = ViewModel()
vm?.work()
vm = nil
```

**Without `@MainActor`:** `work()` is nonisolated, so the task runs on the cooperative thread pool. It may start executing before the main thread processes `vm = nil`. Self is still alive — the guard passes silently.

**With `@MainActor in`:** the task is queued on the main actor. The main thread finishes its current job first — including `vm = nil` — then picks up the task. By that point self is nil, and the guard catches it.

```swift
Task { [weak self] in           // thread pool — races with vm = nil on main thread
    guard let self else { ... } // self may still be alive
}

Task { @MainActor [weak self] in  // queued after current main actor job completes
    guard let self else { ... }   // vm = nil has already run
}
```

The point isn't weak self — it's that **a task bound to the main actor can only run after the main actor finishes its current work**. Switching actor context changes the scheduling order, which changes what state the world is in when the task body starts.

The rule: `@MainActor in` doesn't change capture semantics — it changes when the task runs relative to everything else on the main actor.
