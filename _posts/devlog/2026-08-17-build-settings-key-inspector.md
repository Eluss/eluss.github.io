---
layout: devlog
title: "Xcode Build Settings: Quick Help Shows You the Actual Key"
category: devlog
tags: [xcode, build-settings]
---

Build Settings shows friendly names like "Swift Language Version," but `.xcconfig` files and `xcodebuild` need the raw key. Option-click the setting (or select it and open the Quick Help inspector, ⌥⌘2/⌘⌥3) and the key shows up right there.

```
Swift Language Version → SWIFT_VERSION
Enable Bitcode          → ENABLE_BITCODE
Other Linker Flags      → OTHER_LDFLAGS
```

```
// Debug.xcconfig
SWIFT_VERSION = 5.0
```

**Takeaway:** don't guess or grep the internet for a setting's underlying name — option-click it in the Build Settings editor and Quick Help gives you the exact key to drop into an `.xcconfig`.
