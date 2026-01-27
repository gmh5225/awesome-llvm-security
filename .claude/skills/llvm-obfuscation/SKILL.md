---
name: llvm-obfuscation
description: Expertise in LLVM-based code obfuscation techniques including OLLVM, control flow flattening, string encryption, virtualization, and anti-analysis methods. Use this skill when working on code protection, anti-reverse engineering, or implementing custom obfuscation passes.
---

# LLVM Code Obfuscation Skill

This skill provides comprehensive knowledge of LLVM-based code obfuscation frameworks and techniques for software protection and anti-reverse engineering.

## Core Obfuscation Techniques

### Control Flow Obfuscation
- **Control Flow Flattening (CFF)**: Transform structured control flow into a single dispatcher loop with state machine
- **Bogus Control Flow (BCF)**: Insert opaque predicates and dead code paths
- **CFG Randomization**: Randomize basic block ordering and add fake edges

### Data Obfuscation
- **String Encryption**: Encrypt string literals at compile-time, decrypt at runtime
- **Constant Substitution**: Replace constants with complex expressions
- **Variable Splitting**: Split variables into multiple components

### Code Transformation
- **Instruction Substitution**: Replace standard instructions with equivalent complex sequences
- **MBA (Mixed Boolean-Arithmetic)**: Use mixed boolean-arithmetic expressions for obfuscation
- **Virtualization (VMP)**: Convert code into custom bytecode executed by embedded VM

## Major OLLVM Frameworks

### Classic OLLVM
- **Original OLLVM**: https://github.com/obfuscator-llvm/obfuscator
- Features: BCF, CFF, Instruction Substitution, String Encryption

### Modern Variants
- **Hikari**: Advanced features including function wrapper, anti-class-dump
- **Pluto-Obfuscator**: Well-maintained with MBA, indirect branch, global encryption
- **Arkari**: Modern implementation with enhanced features
- **o-mvll**: Mobile-focused obfuscator for iOS/Android

### Specialized Tools
- **IR VMP**: GANGE666/xVMP, NiTianErXing666/SmallVmp for virtualization
- **Warbird**: Microsoft's commercial obfuscation technology

## Implementation Guidelines

### Creating Custom LLVM Obfuscation Pass

```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"

class MyObfuscationPass : public llvm::FunctionPass {
public:
    static char ID;
    MyObfuscationPass() : FunctionPass(ID) {}
    
    bool runOnFunction(llvm::Function &F) override {
        // Implement obfuscation logic
        for (auto &BB : F) {
            for (auto &I : BB) {
                // Transform instructions
            }
        }
        return true;
    }
};
```

### Best Practices
1. **Preserve Semantics**: Ensure transformations don't break program correctness
2. **Randomization**: Use seeded random number generators for reproducible builds
3. **Layered Approach**: Combine multiple obfuscation techniques
4. **Performance Balance**: Consider runtime overhead vs protection level
5. **Testing**: Extensive testing across different inputs and platforms

## Toolchain Integration

### NDK Integration
- OLLVM with Android NDK (r17-r23+)
- Examples: android-ndk-aarch64-host-LLVM6.0-Ollvm-Armariris

### Compiler Toolchains
- ollvm-mingw: Windows cross-compilation
- ollvm-rust: Rust toolchain integration
- Swift integration: swift-Ollvm11

## Anti-Deobfuscation Considerations

When implementing obfuscation:
- Consider resistance to symbolic execution (SymCC, KLEE)
- Add protection against pattern matching deobfuscators
- Implement anti-debugging checks
- Use dynamic dispatch to hinder static analysis

## Resources

Refer to the main README.md for a comprehensive list of OLLVM implementations and related tools.

## Getting Detailed Information

When you need detailed and up-to-date resource links, tool lists, or project references, fetch the latest data from:

```
https://raw.githubusercontent.com/gmh5225/awesome-llvm-security/refs/heads/main/README.md
```

This README contains comprehensive curated lists of:
- 80+ OLLVM implementations and forks (OLLVM section)
- MSVC Warbird obfuscation tools (MSVC Warbird section)
- IR-based VMP and virtualization projects
- NDK integration examples for different versions
