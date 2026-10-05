---
layout: devlog
title: "Tuist Registry: registryEnabled Points at the Registry, --replace-scm-with-registry Actually Uses It"
category: devlog
tags: [swift, spm, swift-package-manager, registry, tuist]
---

**SCM** = Source Control Management, i.e. git. An "SCM dependency" is any package declared by repository URL, which SwiftPM resolves by cloning the repo. A **registry** dependency is declared by an identity like `scope.name` and comes down as a single zip of one version, with no git history.

```swift
.package(url: "https://github.com/pointfreeco/swift-composable-architecture", from: "1.15.0") // SCM
.package(id: "pointfreeco.swift-composable-architecture", from: "1.15.0")                   // registry
```

Tuist has two switches for the registry, and they do different things.

**`registryEnabled: true`** only tells SwiftPM *where* the registry is:

```swift
let tuist = Tuist(
    project: .tuist(
        generationOptions: .options(registryEnabled: true)
    )
)
```

During `tuist generate` it writes the registry config into `App.xcworkspace/xcshareddata/swiftpm/configuration/`, so you no longer have to run `tuist registry setup` yourself. That's all it does. Only packages you declared with `.package(id:)` go to the registry, and every `.package(url:)` is still cloned from git.

**`--replace-scm-with-registry`** rewrites the SCM dependencies themselves:

```swift
let tuist = Tuist(
    project: .tuist(
        installOptions: .options(
            passthroughSwiftPackageManagerArguments: ["--replace-scm-with-registry"]
        )
    )
)
```

On `tuist install` (or plain `swift package --replace-scm-with-registry resolve`), SwiftPM asks the registry about every git URL in the graph, transitive dependencies included:

```
GET /identifiers?url=https://github.com/pointfreeco/swift-composable-architecture
→ 200 { "identifiers": ["pointfreeco.swift-composable-architecture"] }
```

If the registry knows the URL, SwiftPM downloads the package from the registry. If the registry doesn't know it (404), that one package falls back to git. Your manifests can keep their URLs. For Xcode's own package integration, the same switch is a per-machine default, `IDEPackageDependencySCMToRegistryTransformation = useRegistryIdentityAndSources`, which `tuist registry setup` writes for you.

**Why mixing the two styles breaks without the flag.** SwiftPM tells packages apart by *identity*, not by what's inside them. For a URL dependency the identity is the last path component, so the TCA URL becomes `swift-composable-architecture`. For a registry dependency the identity is the `scope.name` string, so the same package becomes `pointfreeco.swift-composable-architecture`. Those are two different strings, so SwiftPM treats them as two unrelated packages.

Now say your app adds TCA by registry ID, and some library you depend on declares TCA by GitHub URL in *its* `Package.swift`. You can't edit that file. SwiftPM ends up with two copies of the same package in the graph, each with a target named `ComposableArchitecture`:

```
multiple packages ('pointfreeco.swift-composable-architecture', 'swift-composable-architecture')
declare targets with a conflicting name: 'ComposableArchitecture'
```

Even when the names don't collide, you can get two independently resolved versions of the "same" library linked into one app. With `--replace-scm-with-registry`, the library's URL goes through `/identifiers?url=`, gets mapped to `pointfreeco.swift-composable-architecture`, and becomes the *same node* in the graph as your registry dependency. SwiftPM then has one package with both sets of version requirements to satisfy, resolves one version, and builds one module. (There's also a narrower flag, `--use-registry-identity-for-scm`, which only does this identity mapping and still fetches over git.)

**Takeaway:** `registryEnabled` makes the registry *available*, and `--replace-scm-with-registry` makes the whole graph *use* it. Turning on only the first and switching a few dependencies to `.package(id:)` is the easiest way to end up with the duplicate-package error above, because the transitive URL dependencies stay on git under a different identity. Turn on both.
