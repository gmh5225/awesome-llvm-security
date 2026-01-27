---
name: dynamic-instrumentation
description: Expertise in LLVM-based dynamic binary instrumentation, runtime tracing, and program monitoring. Use this skill when implementing runtime analysis tools, code coverage systems, profilers, or dynamic security monitors.
---

# Dynamic Instrumentation Skill

This skill covers dynamic binary instrumentation (DBI), runtime tracing, and program monitoring using LLVM infrastructure.

## Dynamic Binary Instrumentation Overview

### What is DBI?
Dynamic Binary Instrumentation allows modifying program behavior at runtime without source code access:
- Insert analysis code at arbitrary points
- Monitor program execution
- Modify control flow and data

### LLVM-Based DBI Tools
- **QBDI**: QuarkslaB Dynamic Binary Instrumentation
- **Instrew**: Fast instrumentation through LLVM lifting
- **binopt**: Runtime optimization of binary code

## QBDI (QuarkslaB DBI)

### Basic Usage
```cpp
#include <QBDI.h>

// Callback function for instrumentation
QBDI::VMAction onInstruction(QBDI::VMInstanceRef vm, 
                              QBDI::GPRState *gprState,
                              QBDI::FPRState *fprState, 
                              void *data) {
    // Get current instruction info
    const QBDI::InstAnalysis *inst = vm.getInstAnalysis();
    
    printf("Executing: 0x%lx - %s %s\n", 
           inst->address, 
           inst->mnemonic, 
           inst->operandsStr);
    
    return QBDI::VMAction::CONTINUE;
}

int main() {
    QBDI::VM vm;
    
    // Get current stack
    uint8_t *fakestack;
    QBDI::allocateVirtualStack(vm.getGPRState(), 0x100000, &fakestack);
    
    // Add instrumentation callback
    vm.addCodeCB(QBDI::PREINST, onInstruction, nullptr);
    
    // Run target function
    QBDI::rword retval;
    vm.call(&retval, (QBDI::rword)targetFunction, {arg1, arg2});
    
    return 0;
}
```

### Memory Access Tracking
```cpp
QBDI::VMAction onMemoryAccess(QBDI::VMInstanceRef vm,
                               QBDI::GPRState *gprState,
                               QBDI::FPRState *fprState,
                               void *data) {
    // Get memory accesses for current instruction
    std::vector<QBDI::MemoryAccess> memAccesses = vm.getMemoryAccess();
    
    for (const auto &access : memAccesses) {
        const char* type = (access.type == QBDI::MEMORY_READ) ? "READ" : "WRITE";
        printf("%s: addr=0x%lx, size=%d, value=0x%lx\n",
               type, access.accessAddress, access.size, access.value);
    }
    
    return QBDI::VMAction::CONTINUE;
}

// Register callback for memory access events
vm.addMemAccessCB(QBDI::MEMORY_READ_WRITE, onMemoryAccess, nullptr);
```

### Instruction Filtering
```cpp
// Only instrument specific instruction ranges
vm.addCodeRangeCB(startAddr, endAddr, QBDI::PREINST, callback, nullptr);

// Instrument specific modules
vm.addCodeAddrCB(targetAddr, QBDI::PREINST, callback, nullptr);

// Remove instrumentation dynamically
vm.deleteInstrumentation(callbackId);
```

## Instrew - LLVM Lifting DBI

### Concept
Instrew lifts binary code to LLVM IR at runtime, enabling:
- High-level optimizations on binary code
- Efficient instrumentation through LLVM passes
- JIT recompilation with modifications

### Architecture
```
Binary → Rellume Lifter → LLVM IR → Custom Passes → JIT → Execute
                              ↓
                     [Instrumentation Passes]
```

## Compile-Time Instrumentation

### LLVM IR Instrumentation Pass
```cpp
struct InstrumentationPass : public llvm::PassInfoMixin<InstrumentationPass> {
    llvm::PreservedAnalyses run(llvm::Module &M,
                                 llvm::ModuleAnalysisManager &MAM) {
        auto &Ctx = M.getContext();
        
        // Declare instrumentation functions
        auto *VoidTy = llvm::Type::getVoidTy(Ctx);
        auto *Int64Ty = llvm::Type::getInt64Ty(Ctx);
        
        auto *LogFuncTy = llvm::FunctionType::get(VoidTy, {Int64Ty}, false);
        auto LogFunc = M.getOrInsertFunction("__log_bb", LogFuncTy);
        
        for (auto &F : M) {
            for (auto &BB : F) {
                // Insert at beginning of each basic block
                llvm::IRBuilder<> Builder(&*BB.getFirstInsertionPt());
                
                auto *BBAddr = llvm::ConstantInt::get(
                    Int64Ty, reinterpret_cast<uint64_t>(&BB));
                Builder.CreateCall(LogFunc, {BBAddr});
            }
        }
        
        return llvm::PreservedAnalyses::none();
    }
};
```

### SanitizerCoverage
Built-in LLVM coverage instrumentation:

```bash
# Enable coverage instrumentation
clang -fsanitize-coverage=trace-pc-guard source.c

# Edge coverage
clang -fsanitize-coverage=edge source.c

# Trace comparisons
clang -fsanitize-coverage=trace-cmp source.c
```

```cpp
// Implement coverage callbacks
extern "C" void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
    if (!*guard) return;
    
    void *PC = __builtin_return_address(0);
    printf("Edge: guard=%u, PC=%p\n", *guard, PC);
}

extern "C" void __sanitizer_cov_trace_pc_guard_init(
    uint32_t *start, uint32_t *stop) {
    
    static uint32_t N = 0;
    for (uint32_t *x = start; x < stop; x++) {
        *x = ++N;
    }
    printf("Total edges: %u\n", N);
}
```

## Runtime Tracing

### Function Tracing
```cpp
// Compile with: clang -finstrument-functions source.c

extern "C" {
    void __cyg_profile_func_enter(void *func, void *caller) {
        Dl_info info;
        if (dladdr(func, &info)) {
            printf("ENTER: %s\n", info.dli_sname);
        }
    }
    
    void __cyg_profile_func_exit(void *func, void *caller) {
        Dl_info info;
        if (dladdr(func, &info)) {
            printf("EXIT: %s\n", info.dli_sname);
        }
    }
}
```

### XRay Instrumentation
LLVM's built-in instrumentation framework:

```bash
# Enable XRay
clang -fxray-instrument -fxray-instruction-threshold=1 source.c
```

```cpp
// Custom XRay handler
[[clang::xray_always_instrument]]
void my_function() {
    // Function will always be instrumented
}

// Runtime control
__xray_patch();    // Enable instrumentation
__xray_unpatch();  // Disable instrumentation
```

## Performance Profiling

### Block Frequency
```cpp
struct BlockProfiler : public llvm::PassInfoMixin<BlockProfiler> {
    llvm::PreservedAnalyses run(llvm::Function &F,
                                 llvm::FunctionAnalysisManager &FAM) {
        auto &BFI = FAM.getResult<llvm::BlockFrequencyAnalysis>(F);
        
        for (auto &BB : F) {
            auto Freq = BFI.getBlockFreq(&BB);
            llvm::errs() << BB.getName() << ": " << Freq.getFrequency() << "\n";
        }
        
        return llvm::PreservedAnalyses::all();
    }
};
```

### Sampling Profiler Integration
```cpp
// Use with perf or similar
// Map addresses back to source using debug info

void interpretProfile(const std::string &profilePath) {
    // Parse profile data
    // Map samples to LLVM IR/source locations
    // Generate optimization hints
}
```

## System Call Monitoring

### SysCallStubber
Intercept and monitor system calls:

```cpp
// Hook system calls at LLVM IR level
struct SyscallMonitor : public llvm::PassInfoMixin<SyscallMonitor> {
    llvm::PreservedAnalyses run(llvm::Module &M,
                                 llvm::ModuleAnalysisManager &MAM) {
        for (auto &F : M) {
            for (auto &BB : F) {
                for (auto &I : BB) {
                    if (auto *Call = llvm::dyn_cast<llvm::CallInst>(&I)) {
                        if (isSyscallWrapper(Call)) {
                            instrumentSyscall(Call);
                        }
                    }
                }
            }
        }
        return llvm::PreservedAnalyses::none();
    }
};
```

## eBPF Integration

### bpfcov - Code Coverage with eBPF
```cpp
// eBPF program for coverage collection
SEC("uprobe/target_function")
int trace_function(struct pt_regs *ctx) {
    u64 addr = PT_REGS_IP(ctx);
    
    // Record coverage
    u32 *count = bpf_map_lookup_elem(&coverage_map, &addr);
    if (count) {
        __sync_fetch_and_add(count, 1);
    }
    
    return 0;
}
```

## Taint Tracking

### Dynamic Taint Analysis
```cpp
// Shadow memory for taint tracking
class TaintTracker {
    std::unordered_map<void*, TaintInfo> shadowMemory;
    
public:
    void markTainted(void *addr, size_t size, TaintSource source) {
        for (size_t i = 0; i < size; i++) {
            shadowMemory[(char*)addr + i] = {source, true};
        }
    }
    
    bool isTainted(void *addr) {
        return shadowMemory.count(addr) && shadowMemory[addr].tainted;
    }
    
    void propagateTaint(void *dst, void *src, size_t size) {
        for (size_t i = 0; i < size; i++) {
            if (isTainted((char*)src + i)) {
                markTainted((char*)dst + i, 1, shadowMemory[(char*)src + i].source);
            }
        }
    }
};
```

## Best Practices

1. **Minimize Overhead**: Only instrument necessary code paths
2. **Buffer Events**: Batch event logging to reduce I/O
3. **Use Sampling**: Full tracing is expensive, sample for production
4. **Thread Safety**: Ensure instrumentation is thread-safe
5. **Symbol Resolution**: Use debug info for meaningful output

## Integration Patterns

### Fuzzer Integration
```cpp
// Coverage-guided fuzzing with instrumentation
void fuzzerCallback(uint8_t *data, size_t size) {
    // Reset coverage
    __sanitizer_cov_reset_coverage();
    
    // Run target
    targetFunction(data, size);
    
    // Collect coverage
    uint8_t *coverage = __sanitizer_cov_get_coverage();
    feedbackToFuzzer(coverage);
}
```

### Debugging Integration
```cpp
// Breakpoint-like instrumentation
void onBreakpoint(void *addr, void *context) {
    // Dump registers
    // Inspect memory
    // Allow continue/step
}
```

## Resources

See Dynamic Binary Instrumentation, Monitor, and eBPF sections in README.md for related tools and projects.

## Getting Detailed Information

When you need detailed and up-to-date resource links, tool lists, or project references, fetch the latest data from:

```
https://raw.githubusercontent.com/gmh5225/awesome-llvm-security/refs/heads/main/README.md
```

This README contains comprehensive curated lists of:
- Dynamic Binary Instrumentation tools (DBI section)
- Runtime monitoring and tracing tools (Monitor section)
- eBPF-related projects and resources
