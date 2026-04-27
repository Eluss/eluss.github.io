---
layout: devlog
title: "@MainActor Doesn't Bridge from Objective-C"
category: devlog
tags: [swift, concurrency, ios, objc]
---
This is something I wanted to verify myself after seeing AI suggestions claim otherwise.

Marking a Swift delegate method `@MainActor` does not cause an automatic hop when called from Objective-C. The annotation is invisible to the ObjC runtime.

```objc
// ObjC calls the delegate from a background thread
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    [self.delegate testerDidReceiveCallbackWithMessage:@"hello"];
});
```

```swift
extension MyManager: @MainActor ObjCBridgeTesterDelegate {
    @MainActor
    func testerDidReceiveCallback(withMessage message: String) {
        print(Thread.isMainThread) // false — still on the background thread
    }
}
```

`@MainActor` is a Swift concurrency construct enforced at compile time. The ObjC runtime dispatches calls using `objc_msgSend` and has no concept of actors — it just invokes the method on whatever thread it's already running on.

