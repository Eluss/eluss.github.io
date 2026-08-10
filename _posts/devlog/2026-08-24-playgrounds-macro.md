---
layout: devlog
title: "#Playground: Inline Playgrounds Without Leaving Your Source File"
category: devlog
tags: [xcode, swift, playgrounds, macros]
---

`import Playgrounds` unlocks the `#Playground` macro — drop it straight into a regular `.swift` file and Xcode runs it as a live playground canvas next to your code, no separate `.playground` bundle needed.

```swift
import Playgrounds

#Playground {
    let items = fetchItems()
    let total = items.reduce(0) { $0 + $1.price }
}
```

Open the file in Xcode and the canvas (⌥⌘Return) shows each statement's result inline, same as classic Playgrounds, but scoped to that block and living right beside the production code it's exercising — so it can call your app's real types and functions directly, no copy-pasting into a standalone playground.

**Takeaway:** for quick "let me just try this" experiments against existing app code, reach for `#Playground { }` instead of spinning up a separate `.playground` file — it stays version-controlled with the code it tests and gets deleted in the same PR when you're done.
