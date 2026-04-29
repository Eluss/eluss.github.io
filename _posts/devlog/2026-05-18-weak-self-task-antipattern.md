---
layout: devlog
title: "[weak self] + guard let self in a Task — A Pattern Worth Dropping"
category: devlog
tags: [swift, concurrency, async, ios]
---
With completion handlers, `[weak self]` + `guard let self` was a clean pattern. The closure was stored externally, created a retain cycle, and the body was short and synchronous. Weak capture prevented the cycle; the guard was a cheap nil check at the top.

```swift
// Callbacks — this made sense
service.fetch { [weak self] result in
    guard let self else { return }
    self.update(result) // synchronous, done immediately
}
```

The pattern got carried into Swift concurrency by reflex, and it's worth questioning it there.

```swift
func load() {
    Task { [weak self] in
        guard let self else { return }
        let data = await fetchData()
        let processed = await process(data)
        update(processed)
    }
}
```

`guard let self` at the top promotes the weak reference back to a strong one immediately. Self is now strongly retained for the entire task — across every suspension point, for however long the task runs. The weak capture only did work for the instant before the guard.

More importantly: `Task { }` doesn't get stored by anyone externally the way a completion handler does, so there's rarely a retain cycle to break in the first place.

**What to do instead**

If the task represents work that belongs to the object, just capture strongly:

```swift
func load() {
    Task {
        let data = await fetchData()
        update(data) // self kept alive for as long as the work runs — that's fine
    }
}
```

If you want to discard results when the object is gone by the time the async work finishes, check at the point where it matters — after the await:

```swift
func load() {
    Task { [weak self] in
        let data = await fetchData() // do the work regardless
        guard let self else { return } // discard if object is gone by now
        update(data)
    }
}
```