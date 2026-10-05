---
layout: devlog
title: "SwiftPM Asks for a Different Package.swift per Swift Version, and Your Registry Has to Answer"
category: devlog
tags: [swift, spm, swift-package-manager, registry]
---

A package can ship more than one manifest. Next to `Package.swift` you can drop version-specific ones, and SwiftPM picks the most specific match for the toolchain it's running on:

```
Package.swift            // fallback
Package@swift-5.9.swift  // used by Swift 5.9.x
Package@swift-6.swift    // used by any Swift 6.x
```

With git-based dependencies that just works, because SwiftPM checks out the whole repo and looks at the files itself. With a **package registry** it never sees the file list. It has to ask the server, which means the registry has to implement that part of the spec (SE-0292):

```
GET /{scope}/{name}/{version}/Package.swift
→ 200, body = Package.swift
  Link: <…/Package.swift?swift-version=6>; rel="alternate";
        filename="Package@swift-6.swift"; swift-tools-version="6.0"

GET /{scope}/{name}/{version}/Package.swift?swift-version=6
→ 200, body = Package@swift-6.swift
  (or 303 See Other → plain Package.swift if there isn't one)
```

The `Link` header lists the alternates that exist, and the `swift-version` query parameter fetches one of them. If a home-grown registry ignores the query param and always serves the plain `Package.swift`, resolution still "works", just against the wrong manifest. You get the fallback's tools version, platforms and dependencies, and you end up with build errors that never show up when the same package comes from git.

**Takeaway:** if you run your own registry (or proxy one), serving `Package.swift` isn't enough. Advertise `Package@swift-*.swift` files in the `Link` header and honor `?swift-version=`, or any package with versioned manifests will resolve differently than it does from git.
