---
layout: devlog
title: "What Mangled Swift Names Actually Leak"
category: devlog
tags: [swift, security, reverse engineering, binary analysis]
---
Let's compare what the same class looks like in a compiled binary.

**Swift source:**
```swift
class UserController {
    func authenticate(with password: String) -> Bool {
        return password == "secret"
    }
}
```

**ObjC source:**
```objc
@interface UserController : NSObject
- (BOOL)authenticateWithPassword:(NSString *)password;
@end
```

**ObjC in binary** - plaintext:
```bash
$ nm ObjCAuth | grep -i user
0000000100000928 t -[UserController authenticateWithPassword:]
00000001000080c8 S _OBJC_CLASS_$_UserController
```

**Swift in binary** - mangled but decodable:
```bash
$ nm SwiftAuth | grep -i user
0000000100000cfc t _$s9SwiftAuth14UserControllerC12authenticate4withSbSS_tF
...

$ nm SwiftAuth | grep -i user | xcrun swift-demangle
0000000100000cfc t SwiftAuth.UserController.authenticate(with: Swift.String) -> Swift.Bool
```

The mangled symbol `_$s9SwiftAuth14UserControllerC12authenticate4withSbSS_tF` encodes:
- **Module**: `SwiftAuth` (9 chars)
- **Class**: `UserController` (14 chars, `C` = class)
- **Method**: `authenticate`
- **Label**: `with`
- **Types**: `Sb` = Bool, `SS` = String

Both expose your API surface. Swift just requires one extra step.
