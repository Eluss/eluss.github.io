---
layout: devlog
title: "Swift ownership: borrowing, consuming, inout"
category: devlog
tags: [swift, swift 5.9, structs, enums]
source: https://github.com/swiftlang/swift-evolution/blob/main/proposals/0390-noncopyable-structs-and-enums.md
source_title: "Proposal: SE-0390"
---
Swift 5.9 introduced ownership modifiers for non-copyable types (`~Copyable`). Here's the difference:

```swift
struct DBConnection: ~Copyable {
    mutating func open() { /* ... */ }
    mutating func close() { /* ... */ }
    func query(_ sql: String) { /* ... */ }
}
```

**borrowing** - read-only access, caller keeps ownership:
```swift
func inspect(_ connection: borrowing DBConnection) {
    connection.query("SELECT 1")  // OK - read only
    // connection.open()          // Error - can't mutate
}
```

**consuming** - takes ownership, caller loses the value:
```swift
func runAndClose(_ connection: consuming DBConnection) {
    connection.open()
    connection.close()
    // connection is destroyed here
}

runAndClose(db)
// db.query("...")  // Error - db was consumed
```

**inout** - mutable borrow, caller keeps ownership:
```swift
func reopen(_ connection: inout DBConnection) {
    connection.open()  // OK - can mutate
}
// db still usable after call
```

Useful for modeling resources like DB connections, file handles, or locks where you want compile-time guarantees against use-after-close bugs.
