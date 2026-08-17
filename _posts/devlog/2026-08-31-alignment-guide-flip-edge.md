---
layout: devlog
title: "alignmentGuide Trick from WWDC's Shaders Talk: Point One Edge at Another"
category: devlog
tags: [swiftui, layout, wwdc]
---

From the WWDC shaders session — a throwaway layout trick buried in a demo, worth stealing on its own. `.alignmentGuide` doesn't just nudge a guide by an offset, you can redefine it to *return a different guide entirely*, which lets you pin one view's top edge to another view's bottom.

```swift
Text(line.text)
    .overlay(alignment: .bottomLeading) {
        Text(line.formattedTimestamp)
            .alignmentGuide(.bottom) { $0[.top] }
    }
```

The overlay is anchored `.bottomLeading`, so normally its `.bottom` guide lines up with the base text's bottom edge — stacking the timestamp *inside*, over the last line. Overriding `.bottom` to resolve to `$0[.top]` (the overlay's own top edge) tricks the alignment math: the position SwiftUI computes for "bottom" is now wherever the timestamp's *top* is, which pushes the whole timestamp view upward until its top edge sits where its bottom used to — landing it flush above the text instead of overlapping it.

**Takeaway:** `alignmentGuide`'s closure can return any `Dimension` off the view's `ViewDimensions`, not just an offset from the guide you named. Swap in a different edge when you need to flip which side of an overlay touches the anchor, instead of reaching for manual offsets or a separate `VStack`.
