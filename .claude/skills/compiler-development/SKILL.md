---
name: compiler-development
description: Expertise in compiler development using LLVM infrastructure including frontend design, IR generation, optimization passes, and code generation. Use this skill when building custom programming languages, implementing DSL compilers, or working on compiler internals.
---

# Compiler Development Skill

This skill provides comprehensive knowledge of building compilers and language implementations using the LLVM infrastructure.

## Compiler Architecture Overview

### Classic Three-Phase Design
```
Source Code → Frontend → Middle-End (Optimizer) → Backend → Machine Code
                ↓              ↓                      ↓
             AST/IR      LLVM IR Passes          Target Code
```

## Frontend Development

### Lexical Analysis
```cpp
// Token types for a simple language
enum class TokenKind {
    Identifier,
    Number,
    String,
    Keyword,
    Operator,
    Punctuation,
    EndOfFile
};

struct Token {
    TokenKind kind;
    std::string value;
    SourceLocation location;
};
```

### Parser Implementation
- Recursive Descent: Easy to implement, good error messages
- Operator Precedence Parsing: Efficient for expression parsing
- LALR/LR: Use tools like Bison for complex grammars

### AST Design
```cpp
class Expr {
public:
    virtual ~Expr() = default;
    virtual llvm::Value* codegen() = 0;
};

class BinaryExpr : public Expr {
    std::unique_ptr<Expr> LHS, RHS;
    char Op;
public:
    llvm::Value* codegen() override {
        llvm::Value* L = LHS->codegen();
        llvm::Value* R = RHS->codegen();
        
        switch (Op) {
            case '+': return Builder.CreateFAdd(L, R, "addtmp");
            case '-': return Builder.CreateFSub(L, R, "subtmp");
            case '*': return Builder.CreateFMul(L, R, "multmp");
            case '/': return Builder.CreateFDiv(L, R, "divtmp");
        }
    }
};
```

## LLVM IR Generation

### Module and Context Setup
```cpp
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/IRBuilder.h"

class CodeGen {
    std::unique_ptr<llvm::LLVMContext> Context;
    std::unique_ptr<llvm::Module> Module;
    std::unique_ptr<llvm::IRBuilder<>> Builder;
    
public:
    CodeGen() {
        Context = std::make_unique<llvm::LLVMContext>();
        Module = std::make_unique<llvm::Module>("my_module", *Context);
        Builder = std::make_unique<llvm::IRBuilder<>>(*Context);
    }
};
```

### Function Generation
```cpp
llvm::Function* createFunction(const std::string& name, 
                                llvm::Type* returnType,
                                std::vector<llvm::Type*> params) {
    llvm::FunctionType* FT = llvm::FunctionType::get(returnType, params, false);
    llvm::Function* F = llvm::Function::Create(
        FT, llvm::Function::ExternalLinkage, name, Module.get());
    
    llvm::BasicBlock* BB = llvm::BasicBlock::Create(*Context, "entry", F);
    Builder->SetInsertPoint(BB);
    
    return F;
}
```

## JIT Compilation

### LLVM ORC JIT
```cpp
#include "llvm/ExecutionEngine/Orc/LLJIT.h"

auto JIT = llvm::orc::LLJITBuilder().create();
if (!JIT) {
    handleError(JIT.takeError());
}

// Add module
(*JIT)->addIRModule(llvm::orc::ThreadSafeModule(
    std::move(Module), std::move(Context)));

// Look up symbol and execute
auto Sym = (*JIT)->lookup("main");
auto* MainFn = (int(*)())Sym->getAddress();
int result = MainFn();
```

## Optimization Pass Pipeline

### New Pass Manager (Recommended)
```cpp
#include "llvm/Passes/PassBuilder.h"

void optimizeModule(llvm::Module& M) {
    llvm::PassBuilder PB;
    llvm::LoopAnalysisManager LAM;
    llvm::FunctionAnalysisManager FAM;
    llvm::CGSCCAnalysisManager CGAM;
    llvm::ModuleAnalysisManager MAM;
    
    PB.registerModuleAnalyses(MAM);
    PB.registerCGSCCAnalyses(CGAM);
    PB.registerFunctionAnalyses(FAM);
    PB.registerLoopAnalyses(LAM);
    PB.crossRegisterProxies(LAM, FAM, CGAM, MAM);
    
    llvm::ModulePassManager MPM = PB.buildPerModuleDefaultPipeline(
        llvm::OptimizationLevel::O2);
    MPM.run(M, MAM);
}
```

### Custom Pass Implementation
```cpp
struct MyPass : public llvm::PassInfoMixin<MyPass> {
    llvm::PreservedAnalyses run(llvm::Function& F, 
                                 llvm::FunctionAnalysisManager& FAM) {
        for (auto& BB : F) {
            for (auto& I : BB) {
                // Transform instructions
            }
        }
        return llvm::PreservedAnalyses::none();
    }
};
```

## Language Implementation Patterns

### Memory-Safe Languages
- Use LLVM's memory sanitizer hooks
- Implement bounds checking with GEP introspection
- Reference counting or garbage collection integration

### Type Systems
- Implement type inference during AST construction
- Generate appropriate LLVM types (i32, float, struct, ptr)
- Handle generic types via monomorphization or boxing

### Error Handling
- Generate exception handling via LLVM's landingpad/invoke
- Implement Result/Option types as tagged unions
- Use LLVM's personality functions for unwinding

## Notable Language Implementations

### Systems Languages
- **Rust**: Complex borrow checker, trait system → LLVM
- **Zig**: Comptime evaluation, safety features
- **Carbon**: C++ interop, modern syntax

### Scripting Languages
- **Julia**: JIT-compiled scientific computing
- **Crystal**: Ruby-like syntax, static typing
- **Nim**: Python-like, multi-backend

### Domain-Specific
- **Solidity**: Ethereum smart contracts
- **MLIR**: Multi-level IR for ML/AI workloads
- **Halide**: Image processing DSL

## Development Workflow

1. **Start Simple**: Begin with Kaleidoscope tutorial
2. **Incremental Features**: Add one language feature at a time
3. **Test Extensively**: Unit tests for each compiler phase
4. **Use LLVM Tools**: opt, llc, llvm-dis for debugging IR
5. **Profile and Optimize**: Focus on common code patterns

## Resources

### Official Tutorials
- LLVM Kaleidoscope: Building a language from scratch
- Clang internals: Frontend implementation patterns
- Writing an LLVM Backend: Target code generation

### Community Projects
See DIY Compiler section in README.md for 100+ example implementations across different language paradigms.

## Getting Detailed Information

When you need detailed and up-to-date resource links, tool lists, or project references, fetch the latest data from:

```
https://raw.githubusercontent.com/gmh5225/awesome-llvm-security/refs/heads/main/README.md
```

This README contains comprehensive curated lists of:
- 100+ DIY compiler implementations (DIY Compiler section)
- Toolchain configurations and IDE setup
- Compiler development tutorials and books
