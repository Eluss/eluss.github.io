---
layout: devlog
title: "Swift 6.4: One private var No Longer Kills Your Memberwise Init"
category: devlog
tags: [swift, swift6, structs]
---

Up to Swift 6.3, adding one `private` property with a default value made the synthesized memberwise init `private` as well:

```swift
struct Profile {
    var name: String
    var age: Int
    private var cache = [String: Int]()
}

Profile(name: "Ela", age: 30)
// Swift 6.3: ❌ 'Profile' initializer is inaccessible due to 'private' protection level
// Swift 6.4: ✅
```

The fix used to be writing the whole `init(name:age:)` by hand, just to get around a property the caller never needs to touch. SE-0502 changes the rule: a property that **has an initial value** and is **less accessible** than the other properties is left out of the memberwise init. `var x: Int?` counts as having an initial value too, since it's implicitly `nil`.

The rule has limits:

```swift
struct Profile {
    var name: String
    private var id: Int   // no default value
}
// still a private init(name:id:). There's no way to build it without `id`
```

- The memberwise init is still `internal` at most, so it does nothing for `public` structs.
- The old signature lives on as a compatibility overload, so existing calls like `Profile(name:age:cache:)` in the same file keep compiling.
- One way it can break: if you've already written `init(name:age:)` in an extension, you now get a redeclaration error, because the compiler synthesizes the same signature.

**Takeaway:** in 6.4 you can add `private` caches, flags and other bookkeeping properties to a struct without losing its memberwise init. Go through the hand-written inits that existed only because of one `private var` and delete them.
