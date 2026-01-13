---
layout: devlog
title: "#require in Swift Testing"
category: devlog
tags: [swift, testing, swift testing]
source: https://developer.apple.com/documentation/testing/require(_:_:sourcelocation:)-5l63q
source_title: "Apple Documentation"
---
The `#require` macro in Swift Testing safely unwraps optionals and throws if nil, stopping the test immediately. It replaces the old `XCTUnwrap` pattern.

**Before (XCTest):**
```swift
func testUserName() throws {
    let user = try XCTUnwrap(fetchUser())
    XCTAssertEqual(user.name, "John")
}
```

**After (Swift Testing):**
```swift
@Test func userName() throws {
    let user = try #require(fetchUser())
    #expect(user.name == "John")
}
```

It also works with boolean conditions - the test fails if the condition is false:
```swift
@Test func adminAccess() throws {
    let user = try #require(fetchUser())
    try #require(user.isAdmin)  // Fails test if not admin
    // Continue with admin-only tests...
}
```

The key difference from `#expect`: `#require` stops execution on failure, while `#expect` records the failure but continues. Use `#require` when subsequent code depends on the condition being true.
