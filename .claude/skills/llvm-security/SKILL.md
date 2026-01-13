---
name: llvm-security
description: Expertise in LLVM security features including sanitizers, hardening techniques, exploit mitigations, and secure compilation. Use this skill when implementing security-focused compiler features, analyzing vulnerabilities, or hardening applications.
---

# LLVM Security Skill

This skill covers LLVM-based security features, sanitizers, hardening mechanisms, and secure software development practices.

## Sanitizers

### AddressSanitizer (ASan)
Detects memory errors: buffer overflow, use-after-free, use-after-scope.

```bash
# Compile with ASan
clang -fsanitize=address -g program.c -o program

# Key features
# - Stack buffer overflow detection
# - Heap buffer overflow detection  
# - Use-after-free detection
# - Memory leak detection
```

### MemorySanitizer (MSan)
Detects uninitialized memory reads.

```bash
clang -fsanitize=memory -g program.c -o program
```

### ThreadSanitizer (TSan)
Detects data races in multithreaded programs.

```bash
clang -fsanitize=thread -g program.c -o program
```

### UndefinedBehaviorSanitizer (UBSan)
Detects undefined behavior at runtime.

```bash
clang -fsanitize=undefined -g program.c -o program

# Specific checks
clang -fsanitize=signed-integer-overflow,null program.c
```

### Custom Sanitizer Development
```cpp
// Implementing custom memory tracking
extern "C" void __asan_poison_memory_region(void const volatile *addr, size_t size);
extern "C" void __asan_unpoison_memory_region(void const volatile *addr, size_t size);

class SecureAllocator {
public:
    void* allocate(size_t size) {
        // Add red zones around allocation
        void* ptr = malloc(size + 2 * REDZONE_SIZE);
        __asan_poison_memory_region(ptr, REDZONE_SIZE);
        __asan_poison_memory_region((char*)ptr + REDZONE_SIZE + size, REDZONE_SIZE);
        return (char*)ptr + REDZONE_SIZE;
    }
};
```

## Hardening Techniques

### Stack Protection
```bash
# Stack canaries
clang -fstack-protector-strong program.c

# Stack clash protection
clang -fstack-clash-protection program.c

# Safe stack (separate stacks for safe/unsafe data)
clang -fsanitize=safe-stack program.c
```

### Control Flow Integrity (CFI)
```bash
# Forward-edge CFI
clang -fsanitize=cfi -flto program.c

# Specific CFI schemes
clang -fsanitize=cfi-vcall      # Virtual call checks
clang -fsanitize=cfi-nvcall     # Non-virtual member call checks
clang -fsanitize=cfi-icall      # Indirect call checks
```

### Shadow Call Stack
```bash
# Backward-edge protection (return address protection)
clang -fsanitize=shadow-call-stack program.c
```

### Position Independent Executables
```bash
# Full ASLR support
clang -fPIE -pie program.c

# Position independent code for shared libraries
clang -fPIC -shared library.c -o library.so
```

## Symbolic Execution

### Integration with KLEE
```cpp
// Mark symbolic inputs
#include <klee/klee.h>

int main() {
    int input;
    klee_make_symbolic(&input, sizeof(input), "input");
    
    if (input > 0) {
        // Path 1
    } else {
        // Path 2
    }
    return 0;
}
```

### SymCC (Symbolic Execution via Compilation)
Compile-time instrumentation for symbolic execution:
- Faster than IR interpretation
- Supports complex real-world programs
- Integrates with fuzzing workflows

### Symbolic Analysis Tools
- **Caffeine**: LLVM-based symbolic executor
- **SymSan**: Symbolic execution + sanitizers
- **Haybale**: Rust-based LLVM symbolic executor

## Security-Focused Analysis

### Type Checking at Runtime
```cpp
// LLVM TypeSanitizer concepts
// Track type information through allocations
struct TypeInfo {
    const char* typeName;
    size_t typeSize;
    uint64_t typeHash;
};

void checkType(void* ptr, TypeInfo expected) {
    TypeInfo* actual = getTypeInfo(ptr);
    if (actual->typeHash != expected.typeHash) {
        reportTypeMismatch(ptr, actual, expected);
    }
}
```

### Memory Leak Detection
```cpp
// LeakSanitizer integration
extern "C" void __lsan_do_leak_check();
extern "C" void __lsan_disable();
extern "C" void __lsan_enable();

// Custom leak tracking
class PreciseLeakSanitizer {
    std::unordered_map<void*, AllocationInfo> allocations;
    
public:
    void recordAlloc(void* ptr, size_t size, const char* file, int line) {
        allocations[ptr] = {size, file, line, getStackTrace()};
    }
    
    void recordFree(void* ptr) {
        allocations.erase(ptr);
    }
    
    void reportLeaks() {
        for (auto& [ptr, info] : allocations) {
            fprintf(stderr, "Leak: %zu bytes at %s:%d\n", 
                    info.size, info.file, info.line);
        }
    }
};
```

## Exploit Mitigation Implementation

### Return Address Protection
```llvm
; Shadow stack concept in LLVM IR
define void @protected_function() {
entry:
    %return_addr = call ptr @llvm.returnaddress(i32 0)
    call void @shadow_stack_push(ptr %return_addr)
    
    ; Function body...
    
    %saved_addr = call ptr @shadow_stack_pop()
    %current_addr = call ptr @llvm.returnaddress(i32 0)
    %match = icmp eq ptr %saved_addr, %current_addr
    br i1 %match, label %safe_return, label %attack_detected
    
safe_return:
    ret void
    
attack_detected:
    call void @abort()
    unreachable
}
```

### Pointer Authentication (ARM)
```cpp
// Using pointer authentication on ARM64
__attribute__((target("sign-return-address")))
void signed_function() {
    // Return address is cryptographically signed
}
```

## Secure Compilation Pipeline

### Build Flags Checklist
```bash
# Comprehensive hardening
CFLAGS="-O2 \
    -fstack-protector-strong \
    -fstack-clash-protection \
    -fcf-protection=full \
    -fPIE \
    -D_FORTIFY_SOURCE=2 \
    -Wformat -Wformat-security \
    -fsanitize=cfi -flto"

LDFLAGS="-pie \
    -Wl,-z,relro \
    -Wl,-z,now \
    -Wl,-z,noexecstack"
```

### Compiler Security Checks
- `-Wformat-security`: Format string vulnerabilities
- `-Warray-bounds`: Array bounds violations
- `-Wshift-overflow`: Shift operation overflows
- `-Wnull-dereference`: Null pointer dereferences

## Fuzzing Integration

### libFuzzer
```cpp
// Fuzz target template
extern "C" int LLVMFuzzerTestOneInput(const uint8_t *Data, size_t Size) {
    // Parse/process Data
    processInput(Data, Size);
    return 0;
}
```

### Sanitizer + Fuzzer Combination
```bash
# Comprehensive fuzzing setup
clang -fsanitize=fuzzer,address,undefined \
      -fno-omit-frame-pointer \
      -g fuzz_target.c -o fuzzer
```

## Windows-Specific Security

### Control Flow Guard (CFG)
```bash
clang-cl /guard:cf program.c
```

### SEH (Structured Exception Handling)
- LLVM supports Windows SEH
- Use for secure exception handling
- Integrate with security monitoring

## Resources

See Security Features, Sanitizer, and Symbolic Execution sections in README.md for comprehensive tool listings.
