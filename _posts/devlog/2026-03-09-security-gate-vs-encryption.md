---
layout: devlog
title: "Security as a UI Gate vs. Data-Level Encryption"
category: devlog
tags: [security, ios, encryption, architecture]
---
Two architectures for protecting user data behind a PIN. One is fundamentally broken, the other isn't.

**Architecture A: The UI Gate**

```
User enters PIN
        ↓
Compare PIN to stored value
        ↓
    Match? ──→ YES → showPhotos()
        │
        └──→ NO  → denyAccess()
```

The data sits unencrypted on disk. The PIN is just a boolean check. An attacker who can modify the runtime (hooking `denyAccess` to call `showPhotos` instead) bypasses the entire system without knowing the PIN.

**Architecture B: PIN-Derived Encryption**

```
User enters PIN
        ↓
key = PBKDF2(pin, salt, 100000 iterations)
        ↓
AES-256-GCM-Decrypt(key, encrypted_data)
        ↓
    Success? ──→ YES → display decrypted data
        │
        └───→ NO  → decryption fails, garbage output
```

There's no comparison to bypass. The PIN *is* the encryption key (after derivation). Wrong PIN = wrong key = undecryptable data.

**How PBKDF2 key derivation works:**

```swift
import CommonCrypto

func deriveKey(pin: String, salt: Data) -> Data {
    var key = Data(count: 32) // 256-bit key
    let pinData = pin.data(using: .utf8)!

    key.withUnsafeMutableBytes { keyPtr in
        pinData.withUnsafeBytes { pinPtr in
            salt.withUnsafeBytes { saltPtr in
                CCKeyDerivationPBKDF(
                    CCPBKDFAlgorithm(kCCPBKDF2),
                    pinPtr.baseAddress, pinData.count,
                    saltPtr.baseAddress, salt.count,
                    CCPseudoRandomAlgorithm(kCCPRFHmacAlgSHA256),
                    100_000, // iterations — makes brute force slow
                    keyPtr.baseAddress, 32
                )
            }
        }
    }
    return key
}
```

PIN `"1234"` with one salt produces `a7f3b2c1...`. PIN `"1235"` with the same salt produces `9e41d0ff...`. Completely different keys, completely different decryption results.

**Argon2 as a PBKDF2 alternative:**

PBKDF2 is CPU-hard only — each guess costs CPU time. Argon2 (RFC 9106, winner of the Password Hashing Competition) adds memory-hardness on top of that. Each guess requires allocating large amounts of RAM, which makes GPU and ASIC brute-force attacks impractical since memory is expensive to parallelize.