---
layout: devlog
title: "The Compilation Pipeline: Where Obfuscation Lives"
category: devlog
tags: [security, clang, llvm, swift, obfuscation]
---
```
       ObjC                     Swift
         ↓                        ↓
    Preprocessor                  │
         ↓                        ↓
        AST                      AST
         │                        ↓
         │                    Raw SIL
         │                        ↓
         │                  Canonical SIL
         ↓                        ↓
      LLVM IR                  LLVM IR
         ↓                        ↓
     Assembly                 Assembly
         ↓                        ↓
    Object (.o)              Object (.o)
         └────────┬───────────────┘
                  ↓ linker
            Executable
```

View each stage:
```bash
# ObjC
clang -E file.m                    # preprocessed
clang -Xclang -ast-dump file.m     # AST
clang -S -emit-llvm file.m         # LLVM IR
clang -S file.m                    # assembly
clang -c file.m                    # object
clang file.o -o file               # executable

# Swift
swiftc -dump-ast file.swift        # AST
swiftc -emit-silgen file.swift     # raw SIL
swiftc -emit-sil file.swift        # canonical SIL
swiftc -emit-ir file.swift         # LLVM IR
swiftc -S file.swift               # assembly
swiftc -c file.swift               # object
swiftc file.o -o file              # executable
```

**Where can obfuscation happen?**

| Level | Tools | Notes |
|-------|-------|-------|
| Source | Swift Shield | Renames symbols before compile |
| LLVM IR | OLLVM, Hikari | Language agnostic, most common |

Stock compilers don't obfuscate - you need additional tooling.
