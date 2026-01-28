---
layout: devlog
title: "Stripped vs Unstripped: What Survives"
category: devlog
tags: [macos, ios, binary analysis, security, mach-o]
---
Let's see what stripping actually removes.

**Source:**
```swift
class SecretManager {
    let apiKey = "sk_live_9a8b7c6d5e4f"
    let endpoint = "https://api.internal.company.com/v2"

    func validateLicense(_ license: String) -> Bool {
        return license == "XXXX-YYYY-ZZZZ"
    }
}
```

**Unstripped** - full symbol table:
```bash
$ nm MachODemo | xcrun swift-demangle | grep SecretManager
MachODemo.SecretManager.validateLicense(Swift.String) -> Swift.Bool
MachODemo.SecretManager.apiKey.getter : Swift.String
MachODemo.SecretManager.endpoint.getter : Swift.String
MachODemo.SecretManager.__allocating_init() -> MachODemo.SecretManager
...
```

**Stripped** - symbol table removed:
```bash
$ strip MachODemo -o MachODemo_stripped
$ nm MachODemo_stripped | grep SecretManager
(nothing)
```

**But strings still finds everything:**
```bash
$ strings MachODemo_stripped | grep -E "(sk_live|SecretManager|apiKey)"
sk_live_9a8b7c6d5e4f
https://api.internal.company.com/v2
XXXX-YYYY-ZZZZ
SecretManager
apiKey
endpoint
```

Stripping removes the symbol table (function addresses), but hardcoded strings and Swift reflection metadata stay embedded in `__TEXT`. The secrets survive.
