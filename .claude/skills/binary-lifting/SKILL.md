---
name: binary-lifting
description: Expertise in binary lifting techniques - converting machine code to LLVM IR for analysis, decompilation, and recompilation. Use this skill when working on reverse engineering, binary analysis, deobfuscation, or converting binaries to higher-level representations.
---

# Binary Lifting Skill

This skill covers techniques and tools for lifting binary executables to LLVM IR, enabling advanced analysis, transformation, and recompilation of existing binaries.

## Core Concepts

### What is Binary Lifting?
Binary lifting is the process of translating low-level machine code (x86, ARM, etc.) into a higher-level intermediate representation (LLVM IR), enabling:
- Static and dynamic analysis
- Deobfuscation and vulnerability research
- Code recompilation and optimization
- Cross-architecture translation

### Lifting Pipeline
```
Binary → Disassembly → IR Generation → Optimization → Analysis/Recompilation
```

## Major Lifting Frameworks

### Production-Grade Tools
- **RetDec** (Avast): Full decompiler with C output, multi-architecture support
- **McSema** (Trail of Bits): x86/x64 to LLVM IR, function recovery
- **revng**: Based on QEMU, supports multiple architectures
- **reopt** (Galois): Focus on correctness and formal methods

### Research/Specialized Tools
- **Rellume**: Fast x86-64 to LLVM lifting for JIT scenarios
- **fcd**: Pattern-based decompiler with optimization passes
- **bin2llvm**: QEMU-based binary to LLVM translator
- **llvm-mctoll**: Microsoft's machine code to LLVM lifter

### Language-Specific Lifters
- **llvm2c/IR->C**: Convert LLVM IR back to C code
- **llvm2cranelift**: LLVM IR to Cranelift IR
- **Leaven**: LLVM IR to Go language
- **masxinlingvonta**: JVM bytecode to LLVM IR

## Implementation Techniques

### Instruction Semantics Translation

```cpp
// Example: Translating x86 ADD to LLVM IR
Value* translateADD(IRBuilder<> &builder, Value* op1, Value* op2) {
    Value* result = builder.CreateAdd(op1, op2, "add_result");
    
    // Update flags (CF, OF, SF, ZF, etc.)
    updateCarryFlag(builder, op1, op2, result);
    updateOverflowFlag(builder, op1, op2, result);
    updateSignFlag(builder, result);
    updateZeroFlag(builder, result);
    
    return result;
}
```

### Control Flow Recovery
1. **Linear Sweep**: Simple but misses code with embedded data
2. **Recursive Descent**: Follow control flow, better coverage
3. **Speculative Disassembly**: Handle indirect jumps/calls
4. **Machine Learning**: Use ML to identify function boundaries

### Handling Indirect Control Flow
- Value Set Analysis (VSA)
- Symbolic execution for jump target resolution
- Type recovery for virtual table reconstruction

## Triton Integration

Triton symbolic execution engine can be used with lifting:

```python
from triton import TritonContext, ARCH, Instruction

ctx = TritonContext(ARCH.X86_64)

# Symbolically execute and extract AST
inst = Instruction(b"\x48\x01\xd8")  # add rax, rbx
ctx.processing(inst)

# Convert Triton AST to LLVM IR
ast = ctx.getRegisterAst(ctx.registers.rax)
llvm_ir = triton_ast_to_llvm(ast)
```

## Deobfuscation via Lifting

### Approach
1. Lift obfuscated binary to LLVM IR
2. Apply optimization passes to simplify
3. Use custom passes for specific obfuscation patterns
4. Re-emit cleaned code

### Useful Optimization Passes
- Dead Store Elimination (DSE)
- Global Value Numbering (GVN)
- Constant Propagation
- Instruction Combining
- Loop Simplification

### VMP/VM Handler Recovery
- Identify dispatcher patterns
- Extract VM bytecode semantics
- Convert handlers to native IR
- Example: TicklingVMProtect for VMProtect analysis

## Best Practices

1. **Architecture Support**: Handle endianness, calling conventions, ABI differences
2. **Memory Modeling**: Accurate memory layout for global/stack variables
3. **External Dependencies**: Handle library calls and system calls
4. **Validation**: Compare execution traces of original vs lifted code
5. **Incremental Lifting**: Support partial program analysis

## Dynamic Binary Lifting

### Runtime Translation
- **Instrew**: Fast instrumentation through LLVM
- **QBDI**: QuarkslaB Dynamic Binary Instrumentation
- **binopt**: Runtime optimization of binary code

### JIT Recompilation
Lift frequently executed code paths for runtime optimization:
- Profile-guided lifting
- Hot path detection
- Speculative optimization

## Resources

For a complete list of lifting tools and research papers, refer to the LIFT section in the main README.md.

## Getting Detailed Information

When you need detailed and up-to-date resource links, tool lists, or project references, fetch the latest data from:

```
https://raw.githubusercontent.com/gmh5225/awesome-llvm-security/refs/heads/main/README.md
```

This README contains comprehensive curated lists of:
- Binary lifting frameworks and tools (LIFT section)
- Related research papers and documentation
- Implementation examples and tutorials
