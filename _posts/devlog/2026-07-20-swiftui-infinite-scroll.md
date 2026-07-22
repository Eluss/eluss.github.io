---
layout: devlog
title: "SwiftUI Group Lab: Track Visible Items, Not Scroll Offset"
category: devlog
tags: [swiftui, performance, ios]
---
*Notes from a SwiftUI Group Lab Q&A — a summary of what the panel said, not my own take.*

Treat a scroll view's content offset as an implementation detail — with lazy stacks it's only estimated from the heights of off-screen views and carries no reliable semantic meaning. Trigger UI changes off the views actually on screen instead.

`scrollPosition` tracks which item is currently visible by ID, not by offset — so it stays relative to your data model rather than a pixel value:

```swift
struct FeedView: View {
    @State private var position = ScrollPosition(idType: Item.ID.self)

    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(items) { item in
                    ItemRow(item: item).id(item.id)
                }
            }
        }
        .scrollPosition($position)
    }
}
```

For visibility-based triggers — analytics, impression tracking — `onScrollVisibilityChange(threshold:)` fires when a view crosses a percentage-visible threshold, instead of you computing that from offsets yourself:

```swift
ItemRow(item: item)
    .onScrollVisibilityChange(threshold: 0.8) { isVisible in
        if isVisible { analytics.logImpression(item) }
    }
```

`onGeometryChange` and scroll transitions round out the toolkit: the former replaces `GeometryReader` hacks for reading frame changes, the latter offloads scroll-driven visual effects to the system and is often cheaper than computing them by hand from offset.

**Infinite scrolling**, per the panel, doesn't need any offset math either — put a sentinel view at the end of the list and let its `onAppear` trigger the next page fetch:

```swift
LazyVStack {
    ForEach(items) { item in
        ItemRow(item: item)
    }
    Color.clear
        .frame(height: 1)
        .onAppear { Task { await loadMore() } }
}
```

**Takeaway:** stop reasoning about scroll views in terms of content offset. Whatever you're after — revealing a view at some scroll depth, tracking impressions, or paging in more data — there's an API keyed to the views on screen instead.
