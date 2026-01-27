---
name: llvm-learning
description: Comprehensive learning resources and tutorials for LLVM, Clang, and compiler development. Use this skill when helping users learn LLVM internals, find educational resources, or understand compiler concepts.
---

# LLVM Learning Skill

This skill provides curated learning paths and resources for mastering LLVM, Clang, and compiler development.

## Learning Paths

### Beginner Path
1. **Start with Kaleidoscope**: Official LLVM tutorial building a simple language
2. **Understand LLVM IR**: Learn the intermediate representation
3. **Write Simple Passes**: Transform IR with custom passes
4. **Explore Clang**: Understand C/C++ frontend

### Intermediate Path
1. **Deep Dive into Optimization**: Study built-in optimization passes
2. **Backend Development**: Target code generation
3. **Analysis Frameworks**: Pointer analysis, dataflow
4. **Clang Tooling**: LibTooling, AST matchers

### Advanced Path
1. **MLIR**: Multi-level IR for domain-specific compilation
2. **JIT Compilation**: ORC JIT framework
3. **Security Features**: Sanitizers, CFI implementation
4. **Contribute to LLVM**: Patches, reviews, community

## Official Resources

### Documentation
- **LLVM Docs**: https://llvm.org/docs
- **Clang Docs**: https://clang.llvm.org/docs
- **LLVM Language Reference**: Complete IR specification
- **Programmer's Manual**: Core APIs and usage

### Tutorials
- **Kaleidoscope**: https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/index.html
- **Writing an LLVM Pass**: https://llvm.org/docs/WritingAnLLVMPass.html
- **Building LLVM**: https://llvm.org/docs/GettingStarted.html

## Community Resources

### Books and Guides
- **Learn LLVM 12** by Kai Nacke: Comprehensive practical guide
- **LLVM Techniques, Tips, and Best Practices**: Advanced patterns
- **Getting Started with LLVM Core Libraries**: Foundation concepts
- **LLVM Essentials**: Quick start guide

### Video Resources
- **LLVM Developer Meetings**: Recorded talks on YouTube
- **LLVM Weekly**: Newsletter with latest developments
- **EuroLLVM/US LLVM**: Conference recordings

### GitHub Learning Projects
```
# Highly recommended starter projects
banach-space/llvm-tutor     # Comprehensive pass examples
hunterzju/llvm-tutorial     # Step-by-step tutorials
jauhien/iron-kaleidoscope   # Kaleidoscope in Rust
bigconvience/llvm-ir-in-action  # IR examples
```

## LLVM IR Essentials

### Basic Structure
```llvm
; Module level
@global_var = global i32 42

; Function definition
define i32 @add(i32 %a, i32 %b) {
entry:
    %sum = add i32 %a, %b
    ret i32 %sum
}

; External declaration
declare i32 @printf(i8*, ...)
```

### Common Instructions
```llvm
; Arithmetic
%result = add i32 %a, %b
%result = sub i32 %a, %b
%result = mul i32 %a, %b

; Memory
%ptr = alloca i32
store i32 %value, i32* %ptr
%loaded = load i32, i32* %ptr

; Control flow
br i1 %cond, label %then, label %else
br label %next

; Function calls
%retval = call i32 @function(i32 %arg)
```

### Type System
```llvm
; Integer types: i1, i8, i16, i32, i64, i128
; Floating point: half, float, double
; Pointers: i32*, [10 x i32]*, { i32, i64 }*
; Arrays: [10 x i32]
; Structs: { i32, i64, i8* }
; Vectors: <4 x i32>
```

## Writing LLVM Passes

### New Pass Manager (Recommended)
```cpp
#include "llvm/Passes/PassPlugin.h"
#include "llvm/Passes/PassBuilder.h"

struct MyPass : public llvm::PassInfoMixin<MyPass> {
    llvm::PreservedAnalyses run(llvm::Function &F,
                                 llvm::FunctionAnalysisManager &FAM) {
        for (auto &BB : F) {
            for (auto &I : BB) {
                llvm::errs() << I << "\n";
            }
        }
        return llvm::PreservedAnalyses::all();
    }
};

// Plugin registration
extern "C" LLVM_ATTRIBUTE_WEAK ::llvm::PassPluginLibraryInfo
llvmGetPassPluginInfo() {
    return {LLVM_PLUGIN_API_VERSION, "MyPass", LLVM_VERSION_STRING,
            [](llvm::PassBuilder &PB) {
                PB.registerPipelineParsingCallback(
                    [](llvm::StringRef Name, llvm::FunctionPassManager &FPM,
                       llvm::ArrayRef<llvm::PassBuilder::PipelineElement>) {
                        if (Name == "my-pass") {
                            FPM.addPass(MyPass());
                            return true;
                        }
                        return false;
                    });
            }};
}
```

### Running Custom Passes
```bash
# Build as shared library
clang++ -shared -fPIC -o MyPass.so MyPass.cpp `llvm-config --cxxflags --ldflags`

# Run with opt
opt -load-pass-plugin=./MyPass.so -passes="my-pass" input.ll -o output.ll
```

## Clang Development

### AST Exploration
```bash
# Dump AST
clang -Xclang -ast-dump -fsyntax-only source.cpp

# AST as JSON
clang -Xclang -ast-dump=json -fsyntax-only source.cpp
```

### LibTooling Basics
```cpp
#include "clang/Tooling/Tooling.h"
#include "clang/ASTMatchers/ASTMatchers.h"
#include "clang/ASTMatchers/ASTMatchFinder.h"

using namespace clang::ast_matchers;

// Match all function declarations
auto matcher = functionDecl().bind("func");

class Callback : public MatchFinder::MatchCallback {
    void run(const MatchFinder::MatchResult &Result) override {
        if (auto *FD = Result.Nodes.getNodeAs<FunctionDecl>("func")) {
            llvm::outs() << "Found: " << FD->getName() << "\n";
        }
    }
};
```

### AST Matchers Reference
```cpp
// Common matchers
functionDecl()               // Match function declarations
varDecl()                    // Match variable declarations
binaryOperator()             // Match binary operators
callExpr()                   // Match function calls
ifStmt()                     // Match if statements

// Narrowing matchers
hasName("foo")               // Match by name
hasType(asString("int"))     // Match by type
isPrivate()                  // Match private members

// Traversal matchers
hasDescendant(...)           // Match in subtree
hasAncestor(...)             // Match in parent tree
hasBody(...)                 // Match function body
```

## Development Tools

### Essential Commands
```bash
# Compile to LLVM IR
clang -S -emit-llvm source.c -o source.ll

# Optimize IR
opt -O2 source.ll -o optimized.ll

# Disassemble bitcode
llvm-dis source.bc -o source.ll

# Assemble to bitcode
llvm-as source.ll -o source.bc

# Generate native code
llc source.ll -o source.s

# View CFG
opt -passes=view-cfg source.ll
```

### Debugging LLVM IR
```bash
# With debug info
clang -g -S -emit-llvm source.c

# Debug IR transformation
opt -debug-pass=Structure -O2 source.ll

# Verify IR validity
opt -passes=verify source.ll
```

## Common Analysis APIs

```cpp
// Dominator Tree
#include "llvm/Analysis/Dominators.h"
DominatorTree &DT = FAM.getResult<DominatorTreeAnalysis>(F);
if (DT.dominates(BB1, BB2)) { /* BB1 dominates BB2 */ }

// Loop Analysis
#include "llvm/Analysis/LoopInfo.h"
LoopInfo &LI = FAM.getResult<LoopAnalysis>(F);
for (Loop *L : LI) { /* process loops */ }

// Alias Analysis
#include "llvm/Analysis/AliasAnalysis.h"
AAResults &AA = FAM.getResult<AAManager>(F);
AliasResult R = AA.alias(Ptr1, Ptr2);

// Call Graph
#include "llvm/Analysis/CallGraph.h"
CallGraph &CG = MAM.getResult<CallGraphAnalysis>(M);
```

## Practice Projects

### Beginner
1. Instruction counter pass
2. Function call logger
3. Dead code finder

### Intermediate
1. Simple constant propagation
2. Branch probability estimator
3. Memory access pattern analyzer

### Advanced
1. Custom optimization pass
2. Security vulnerability detector
3. Domain-specific language frontend

## Community

### Getting Help
- **LLVM Discourse**: https://discourse.llvm.org
- **LLVM Discord**: Real-time chat
- **LLVM Mailing Lists**: llvm-dev, cfe-dev

### Contributing
- **Phabricator**: Code reviews (being migrated to GitHub)
- **GitHub**: https://github.com/llvm/llvm-project
- **Bug Tracker**: https://github.com/llvm/llvm-project/issues

## Resources Index

For a comprehensive list of tutorials, books, and example projects, see the LLVM Tutorial and Clang Tutorial sections in the main README.md.

## Getting Detailed Information

When you need detailed and up-to-date resource links, tutorials, or learning materials, fetch the latest data from:

```
https://raw.githubusercontent.com/gmh5225/awesome-llvm-security/refs/heads/main/README.md
```

This README contains comprehensive curated lists of:
- LLVM tutorials and learning resources (LLVM Tutorial section)
- Clang tutorials and AST guides (Clang Tutorial section)
- C++ modern programming guides (CPP Tutorial section)
- Compiler and security books
