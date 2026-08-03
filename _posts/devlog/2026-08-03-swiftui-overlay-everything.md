---
layout: devlog
title: "SwiftUI Group Lab: There's No API for 'On Top of Everything'"
category: devlog
tags: [swiftui, ios, presentation]
---
*Notes from a SwiftUI Group Lab Q&A — a summary of what the panel said, not my own take.*

Question: how do you present a full-screen overlay above *everything*, including whatever sheet or full-screen-cover is already up — say, a forced login screen after a network disconnect? Short answer from the panel: SwiftUI doesn't have an API for that, and that's arguably on purpose.

Presentations in SwiftUI form a hierarchy. From the root, one `fullScreenCover` on top is easy:

```swift
struct RootView: View {
    @State private var showLogin = false

    var body: some View {
        ContentView()
            .fullScreenCover(isPresented: $showLogin) {
                LoginView()
            }
    }
}
```

But if something else already presented a sheet or cover deeper in the hierarchy, you can't just reach in from outside and stack another presentation above it — you'd need to track every active presentation yourself and coordinate through that, since SwiftUI has no built-in registry of "what's currently on top."

When that coordination isn't worth building, the panel's fallback is to drop to UIKit: with the UIKit scene lifecycle you get direct access to `UIWindowScene`, so you can spin up a new `UIWindow`, give it a `UIHostingController` root, and position it above everything — SwiftUI content included.

```swift
final class OverlayWindowController {
    private var overlayWindow: UIWindow?

    func showOverlay(in scene: UIWindowScene) {
        let window = UIWindow(windowScene: scene)
        window.rootViewController = UIHostingController(rootView: LoginView())
        window.windowLevel = .alert + 1
        window.makeKeyAndVisible()
        overlayWindow = window
    }
}
```

The catch: this doesn't scale past one team doing it. If two parts of the app both decide their thing "always needs to be on top," it degenerates into whichever `UIWindow` was made last winning — a distributed layering decision with no source of truth, which is exactly the kind of bug SwiftUI's presentation model is designed to prevent.

**Takeaway:** the panel's real advice was to question the design brief before reaching for this. A forced full-screen cover over an arbitrary presentation state is disruptive to the user regardless of how it's implemented — often the better fix is unwinding the navigation/presentation stack to a base state and restoring it after login, rather than layering on top of whatever's currently shown. If you do need a global overlay, a single coordinated owner (not ad hoc `UIWindow`s from wherever) is what keeps it from becoming a last-window-wins race.
