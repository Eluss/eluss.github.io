---
layout: devlog
title: "Swift 6.2 Span: Safe, Non-Owning Buffer Views"
category: devlog
tags: [swift, performance, memory, swift6]
---
`Span<T>` is Swift 6.2's answer to `UnsafeBufferPointer`: a bounds-checked, non-owning view into a contiguous region of memory — with lifetime safety enforced by the compiler.

```swift
let numbers = [1, 2, 3, 4, 5]
let span: Span<Int> = numbers.span   // zero-copy view, no allocation

print(span[0])      // 1
print(span.count)   // 5

for value in span {
    process(value)
}
```

The key property is that `Span` is `~Escapable` — it cannot outlive the storage it views. The compiler rejects any attempt to escape it:

```swift
func leakedSpan() -> Span<Int> {   // ❌ Span<Int> is non-escapable
    let numbers = [1, 2, 3, 4, 5]
    return numbers.span             // compile error: span would outlive numbers
}
```

This makes the entire class of dangling-pointer bugs a compile-time error rather than a runtime crash. Compare with the old approach:

```swift
// Before: no safety net
func riskyWork(buffer: UnsafeBufferPointer<Int>) { ... }
numbers.withUnsafeBufferPointer { riskyWork(buffer: $0) }  // easy to misuse

// Swift 6.2: lifetime tracked by the compiler
func safeWork(span: Span<Int>) { ... }
safeWork(span: numbers.span)
```

For byte-level access there is `RawSpan`, and `MutableSpan<T>` covers write-enabled views (with the same lifetime guarantees).

**Takeaway:** prefer `Span<T>` over `UnsafeBufferPointer` whenever you need a performant, zero-copy read over contiguous memory — the `~Escapable` constraint gives you the speed without the footguns.
