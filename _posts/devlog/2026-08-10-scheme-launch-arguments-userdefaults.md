---
layout: devlog
title: "Xcode Schemes: Launch Arguments Silently Override UserDefaults"
category: devlog
tags: [xcode, ios, debugging, userdefaults]
---

`Edit Scheme → Run → Arguments → Arguments Passed On Launch` isn't just for `-AppleLanguage`-style system flags. Anything typed there as `-key value` gets registered into `NSArgumentDomain`, and that domain outranks everything else in `UserDefaults`' search order — including values you explicitly wrote with `.set()`.

```
-disableAnimations YES
-apiEnvironment staging
```

```swift
UserDefaults.standard.bool(forKey: "disableAnimations") // true, no code change needed
UserDefaults.standard.string(forKey: "apiEnvironment")   // "staging"
```

Even if the app has previously called `UserDefaults.standard.set(false, forKey: "disableAnimations")` and that's sitting on disk, the scheme argument wins on every launch — it doesn't persist, it just shadows whatever's stored for the life of that process.

Non-string values need plist syntax, quoted so the shell doesn't eat the parens:

```
-featureFlags '(chat, offlineSync)'
```

**Takeaway:** for temporary overrides — feature flags, locale testing, pointing at a different API environment — duplicate the scheme and set the argument there instead of adding `#if DEBUG` branches or hand-editing defaults in the simulator. Switching schemes becomes the toggle.
