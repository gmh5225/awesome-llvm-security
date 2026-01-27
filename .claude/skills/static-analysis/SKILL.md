---
name: static-analysis
description: Expertise in LLVM-based static analysis including dataflow analysis, pointer analysis, taint tracking, and program verification. Use this skill when implementing security scanners, bug finders, code quality tools, or performing program analysis research.
---

# Static Analysis Skill

This skill covers static program analysis techniques using LLVM infrastructure for security research, vulnerability detection, and code quality assessment.

## Analysis Categories

### Dataflow Analysis
- **Forward Analysis**: Track values from definitions to uses
- **Backward Analysis**: Track from uses back to definitions
- **May/Must Analysis**: Conservative vs precise approximations

### Control Flow Analysis
- **Dominator Trees**: Identify code dominance relationships
- **Post-Dominator Trees**: Control dependence analysis
- **Loop Analysis**: Detect and characterize loops

### Pointer Analysis
- **Flow-Insensitive**: Andersen's, Steensgaard's algorithms
- **Flow-Sensitive**: Track pointer values at each program point
- **Context-Sensitive**: Distinguish calling contexts

## LLVM Analysis Infrastructure

### Using Built-in Analyses
```cpp
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/DominatorTree.h"

void analyze(llvm::Function& F, llvm::FunctionAnalysisManager& FAM) {
    // Get dominator tree
    auto& DT = FAM.getResult<llvm::DominatorTreeAnalysis>(F);
    
    // Get loop info
    auto& LI = FAM.getResult<llvm::LoopAnalysis>(F);
    
    // Get alias analysis
    auto& AA = FAM.getResult<llvm::AAManager>(F);
    
    // Check if two pointers may alias
    llvm::AliasResult AR = AA.alias(Ptr1, Ptr2);
}
```

### Implementing Custom Analysis
```cpp
class TaintAnalysis {
    std::set<llvm::Value*> taintedValues;
    
public:
    void markTainted(llvm::Value* V) {
        taintedValues.insert(V);
    }
    
    bool isTainted(llvm::Value* V) {
        return taintedValues.count(V) > 0;
    }
    
    void propagate(llvm::Instruction* I) {
        // Propagate taint through operations
        for (auto& Op : I->operands()) {
            if (isTainted(Op)) {
                markTainted(I);
                break;
            }
        }
    }
};
```

## Taint Analysis

### Source-Sink Model
```cpp
// Define taint sources (user input, network, files)
bool isTaintSource(llvm::CallInst* CI) {
    llvm::Function* F = CI->getCalledFunction();
    if (!F) return false;
    
    static const std::set<std::string> sources = {
        "read", "recv", "fread", "getenv", "gets", "scanf"
    };
    return sources.count(F->getName().str()) > 0;
}

// Define sensitive sinks (SQL queries, system calls, format strings)
bool isSensitiveSink(llvm::CallInst* CI) {
    llvm::Function* F = CI->getCalledFunction();
    if (!F) return false;
    
    static const std::set<std::string> sinks = {
        "system", "exec", "printf", "strcpy", "memcpy", "sql_query"
    };
    return sinks.count(F->getName().str()) > 0;
}
```

### Interprocedural Analysis
- Track taint across function boundaries
- Handle indirect calls via call graph analysis
- Context-sensitivity for precision

## Pointer Analysis Frameworks

### Using Andersen's Analysis
```cpp
#include "llvm/Analysis/CFLAndersAliasAnalysis.h"

// Check may-alias relationship
void checkAlias(llvm::AAResults& AA, llvm::Value* P1, llvm::Value* P2) {
    switch (AA.alias(P1, P2)) {
        case llvm::AliasResult::NoAlias:
            // Definitely don't alias
            break;
        case llvm::AliasResult::MayAlias:
            // Might alias
            break;
        case llvm::AliasResult::MustAlias:
            // Always alias
            break;
    }
}
```

### Points-To Sets
```cpp
// Track what each pointer may point to
class PointsToAnalysis {
    std::map<llvm::Value*, std::set<llvm::Value*>> pointsTo;
    
public:
    void addPointsTo(llvm::Value* ptr, llvm::Value* target) {
        pointsTo[ptr].insert(target);
    }
    
    const std::set<llvm::Value*>& getPointsTo(llvm::Value* ptr) {
        return pointsTo[ptr];
    }
};
```

## Dependency Analysis

### Data Dependency Graph (DDG)
```cpp
#include "llvm/Analysis/DDG.h"

void buildDDG(llvm::Function& F, llvm::FunctionAnalysisManager& FAM) {
    auto& LI = FAM.getResult<llvm::LoopAnalysis>(F);
    
    for (auto* L : LI) {
        llvm::DataDependenceGraph DDG(*L, FAM.getResult<llvm::AAManager>(F));
        
        for (auto& Node : DDG) {
            // Analyze data dependencies in loop
        }
    }
}
```

### Program Slicing
- Forward slicing: Find all statements affected by a variable
- Backward slicing: Find all statements affecting a variable
- Use for debugging, testing, and security analysis

## Major Static Analysis Tools

### Comprehensive Frameworks
- **Phasar**: Industrial-strength LLVM-based analysis framework
- **SVF**: Scalable pointer analysis and value-flow analysis
- **Joern**: Code analysis platform with graph queries
- **CodeChecker**: Clang-based static analysis integration

### Specialized Tools
- **clam**: Abstract interpretation framework
- **dg**: Dependence graph construction
- **Semgrep**: Pattern-based multi-language analysis

## Security Analysis Applications

### Vulnerability Detection
```cpp
// Buffer overflow detection
void checkBufferAccess(llvm::GetElementPtrInst* GEP) {
    // Get array size if available
    llvm::Type* SourceType = GEP->getSourceElementType();
    if (auto* AT = llvm::dyn_cast<llvm::ArrayType>(SourceType)) {
        uint64_t arraySize = AT->getNumElements();
        
        // Check if index might exceed bounds
        llvm::Value* Index = GEP->getOperand(2);
        // Perform range analysis on index
    }
}
```

### Use-After-Free Detection
- Track allocation/deallocation points
- Build reaching definitions for pointers
- Flag uses after free calls

### Information Flow
- Track sensitive data (passwords, keys, PII)
- Detect leaks to logs, network, or untrusted sinks
- Implement declassification rules

## Best Practices

1. **Soundness vs Completeness**: Understand trade-offs
2. **Scalability**: Use summaries for large codebases
3. **Path Sensitivity**: Balance precision and performance
4. **False Positive Management**: Prioritize actionable findings
5. **Incremental Analysis**: Cache results for faster re-analysis

## Integration with IDEs

### Clangd/LSP Integration
- Provide real-time analysis feedback
- Integrate with editor diagnostics
- Support quick-fix suggestions

### CI/CD Integration
- Run analysis on pull requests
- Block merges on critical findings
- Track analysis trends over time

## Resources

See Static Analysis and Clang Plugins sections in README.md for comprehensive tool listings and research references.

## Getting Detailed Information

When you need detailed and up-to-date resource links, tool lists, or project references, fetch the latest data from:

```
https://raw.githubusercontent.com/gmh5225/awesome-llvm-security/refs/heads/main/README.md
```

This README contains comprehensive curated lists of:
- Static analysis frameworks (Static Analysis section)
- Pointer analysis and taint tracking tools
- Program verification and bug finding tools
- Clang-based code analysis plugins
