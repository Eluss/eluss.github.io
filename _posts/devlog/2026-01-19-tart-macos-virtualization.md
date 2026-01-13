---
layout: devlog
title: "Tart: macOS VMs like containers"
category: devlog
tags: [macos, ci/cd, virtualization]
source: https://github.com/cirruslabs/tart
source_title: "Tart on GitHub"
---
Tart from Cirrus Labs uses Apple's Virtualization.framework to run macOS/Linux VMs on Apple Silicon. The killer feature: it distributes VM images via OCI registries, so you can pull/push them like Docker images.

```bash
tart clone ghcr.io/cirruslabs/macos-sequoia-xcode:latest my-vm
tart run my-vm
```

Combine with Packer to automate image creation - install Xcode, dependencies, snapshot, and push to your registry:

```bash
tart push my-vm ghcr.io/myorg/macos-ci:v1
```

OCI is only the distribution format - it's not containerizing macOS (that would violate licensing). It just chunks the disk image into layers for efficient transfer and versioning. Great for CI runners.
