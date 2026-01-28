---
layout: devlog
title: "Debug Symbols: The dSYM Double-Edged Sword"
category: devlog
tags: [debugging, security, xcode, dsym, dwarf]
---
dSYM bundles contain DWARF debug info - essential for crash symbolication, dangerous if leaked.

**Source:**
```swift
class SecretManager {                                    // line 3
    let apiKey = "sk_live_9a8b7c6d5e4f"                  // line 4

    func validateLicense(_ license: String) -> Bool {    // line 6
        return license == "XXXX-YYYY-ZZZZ"               // line 7
    }
}

let manager = SecretManager()                            // line 11
print(manager.validateLicense("test"))                   // line 12
```

**Generate dSYM:**
```bash
$ swiftc -g -c Demo.swift -o Demo.o
$ swiftc -g Demo.o -o Demo
$ dsymutil Demo -o Demo.dSYM
```

**What `--debug-info` contains:**

`DW_TAG_*` entries describe your code structure:
```bash
$ dwarfdump --debug-info Demo.dSYM

DW_TAG_structure_type                    # class/struct definition
  DW_AT_name          ("SecretManager")

DW_TAG_subprogram                        # function definition
  DW_AT_name          ("validateLicense")
  DW_AT_decl_file     ("/Users/dev/MyApp/Demo.swift")
  DW_AT_decl_line     (6)

DW_TAG_formal_parameter                  # function parameter
  DW_AT_name          ("license")
```

**Address → source mapping with `--lookup`:**

Given a crash address, find exact source location:
```bash
$ dwarfdump --lookup 0x100000dac Demo.dSYM
Line info: file 'Demo.swift', line 6    # func validateLicense

$ dwarfdump --lookup 0x100000de0 Demo.dSYM
Line info: file 'Demo.swift', line 7, column 27    # return statement

$ dwarfdump --lookup 0x100000b50 Demo.dSYM
Line info: file 'Demo.swift', line 11   # let manager = ...
```

Full source paths, class names, function names, parameter names, exact line/column - a roadmap to your code.
