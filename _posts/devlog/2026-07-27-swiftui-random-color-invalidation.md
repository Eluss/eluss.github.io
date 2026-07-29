---
layout: devlog
title: "SwiftUI Group Lab: Spot Invalidations with a Random Background Color"
category: devlog
tags: [swiftui, performance, ios, debugging]
---
*Notes from a SwiftUI Group Lab Q&A — a summary of what the panel said, not my own take.*

Body invalidation in SwiftUI is invisible by default — a view can re-evaluate far more often than you'd guess, with nothing on screen to show it. The panel's trick: give the view a random background color on every `body` evaluation, so each re-render literally flashes a different color.

```swift
struct RowView: View {
    let item: Item

    var body: some View {
        Text(item.title)
            .padding()
            .background(Color(
                red: .random(in: 0...1),
                green: .random(in: 0...1),
                blue: .random(in: 0...1)
            ))
    }
}
```

Since the color is generated inside `body`, it changes on every invalidation. A row that should only redraw when its own data changes but instead flickers on every parent update, scroll, or unrelated state change becomes obvious at a glance — no Instruments session required.

**Takeaway:** wrap a suspect view's background in a random color during development to turn invisible re-renders into a visible signal, then strip it out before shipping.
