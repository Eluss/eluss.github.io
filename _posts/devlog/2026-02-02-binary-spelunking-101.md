---
layout: devlog
title: "Binary Spelunking 101: nm, otool, strings"
category: devlog
tags: [security, macos, binary analysis, reverse engineering]
---
Some tools for peeking inside compiled binaries. Let's compile a simple Swift file:

```swift
class UserAuthenticator {
    private let apiKey = "sk_live_abc123secret"
    private let apiEndpoint = "https://api.myapp.com/v1/auth"

    func authenticate(username: String, password: String) -> Bool {
        return user == "admin" && pass == "supersecret123"
    }
}

class PaymentProcessor {
    let merchantId = "merchant_prod_xyz789"
    func processPayment(amount: Double) -> Bool { ... }
}
```

**strings** - extract readable text:
```bash
$ strings BinaryDemo | grep -iE "(secret|http|merchant|admin)"
sk_live_abc123secret
https://api.myapp.com/v1/auth
admin
supersecret123
merchant_prod_xyz789
```

**nm** - list symbols (functions, classes, globals):
```bash
$ nm BinaryDemo | grep Payment | head -5
00000001000013ec t _$s10BinaryDemo16PaymentProcessorC07processC06amountSbSd_tF
00000001000013b8 t _$s10BinaryDemo16PaymentProcessorC10merchantIdSSvg
...

$ nm BinaryDemo | xcrun swift-demangle | grep Payment | head -3
00000001000013ec t BinaryDemo.PaymentProcessor.processPayment(amount: Swift.Double) -> Swift.Bool
00000001000013b8 t BinaryDemo.PaymentProcessor.merchantId.getter : Swift.String
```

**otool -L** - linked libraries:
```bash
$ otool -L BinaryDemo
BinaryDemo:
  /usr/lib/libSystem.B.dylib (...)
  /System/Library/Frameworks/Foundation.framework/Versions/C/Foundation (...)
  /usr/lib/swift/libswiftCore.dylib (...)
```

*Note: This binary was compiled with plain `swiftc` - no symbol stripping, obfuscation, or App Store encryption (FairPlay).*