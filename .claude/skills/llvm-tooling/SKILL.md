---
name: llvm-tooling
description: Expertise in LLVM tooling development including Clang plugins, LLDB debugger extensions, Clangd/LSP, and LibTooling. Use this skill when building source code analysis tools, refactoring tools, debugger extensions, or IDE integrations.
---

# LLVM Tooling Skill

This skill covers development of tools using LLVM/Clang infrastructure for source code analysis, debugging, and IDE integration.

## Clang Plugin Development

### Plugin Architecture
Clang plugins are dynamically loaded libraries that extend Clang's functionality during compilation.

```cpp
#include "clang/Frontend/FrontendPluginRegistry.h"
#include "clang/AST/ASTConsumer.h"
#include "clang/AST/RecursiveASTVisitor.h"

class MyVisitor : public clang::RecursiveASTVisitor<MyVisitor> {
public:
    bool VisitFunctionDecl(clang::FunctionDecl *FD) {
        llvm::outs() << "Found function: " << FD->getName() << "\n";
        return true;
    }
};

class MyConsumer : public clang::ASTConsumer {
    MyVisitor Visitor;
public:
    void HandleTranslationUnit(clang::ASTContext &Context) override {
        Visitor.TraverseDecl(Context.getTranslationUnitDecl());
    }
};

class MyPlugin : public clang::PluginASTAction {
protected:
    std::unique_ptr<clang::ASTConsumer> CreateASTConsumer(
        clang::CompilerInstance &CI, llvm::StringRef) override {
        return std::make_unique<MyConsumer>();
    }
    
    bool ParseArgs(const clang::CompilerInstance &CI,
                   const std::vector<std::string> &args) override {
        return true;
    }
};

static clang::FrontendPluginRegistry::Add<MyPlugin>
    X("my-plugin", "My custom plugin description");
```

### Running Clang Plugins
```bash
# Build plugin
clang++ -shared -fPIC -o MyPlugin.so MyPlugin.cpp \
    $(llvm-config --cxxflags --ldflags)

# Run plugin
clang -Xclang -load -Xclang ./MyPlugin.so \
      -Xclang -plugin -Xclang my-plugin \
      source.cpp
```

## LibTooling

### Standalone Tools
```cpp
#include "clang/Tooling/CommonOptionsParser.h"
#include "clang/Tooling/Tooling.h"
#include "clang/ASTMatchers/ASTMatchFinder.h"

using namespace clang::tooling;
using namespace clang::ast_matchers;

// Define matcher
auto functionMatcher = functionDecl(hasName("targetFunction")).bind("func");

// Callback handler
class FunctionCallback : public MatchFinder::MatchCallback {
public:
    void run(const MatchFinder::MatchResult &Result) override {
        if (auto *FD = Result.Nodes.getNodeAs<FunctionDecl>("func")) {
            llvm::outs() << "Found: " << FD->getQualifiedNameAsString() << "\n";
        }
    }
};

int main(int argc, const char **argv) {
    auto ExpectedParser = CommonOptionsParser::create(argc, argv, MyCategory);
    ClangTool Tool(ExpectedParser->getCompilations(),
                   ExpectedParser->getSourcePathList());
    
    FunctionCallback Callback;
    MatchFinder Finder;
    Finder.addMatcher(functionMatcher, &Callback);
    
    return Tool.run(newFrontendActionFactory(&Finder).get());
}
```

### AST Matchers Reference
```cpp
// Declaration matchers
functionDecl()           // Match function declarations
varDecl()               // Match variable declarations
recordDecl()            // Match struct/class/union
fieldDecl()             // Match class fields
cxxMethodDecl()         // Match C++ methods
cxxConstructorDecl()    // Match constructors

// Expression matchers
callExpr()              // Match function calls
binaryOperator()        // Match binary operators
unaryOperator()         // Match unary operators
memberExpr()            // Match member access

// Narrowing matchers
hasName("name")         // Filter by name
hasType(asString("int")) // Filter by type
isPublic()              // Filter by access specifier
hasAnyParameter(...)    // Match parameters

// Traversal matchers
hasDescendant(...)      // Match anywhere in subtree
hasAncestor(...)        // Match in parent chain
has(...)                // Match direct children
forEach(...)            // Match all children
```

## LLDB Extension Development

### Python Commands
```python
import lldb

def my_command(debugger, command, result, internal_dict):
    """Custom LLDB command implementation"""
    target = debugger.GetSelectedTarget()
    process = target.GetProcess()
    thread = process.GetSelectedThread()
    frame = thread.GetSelectedFrame()
    
    # Get variable value
    var = frame.FindVariable("my_var")
    result.AppendMessage(f"my_var = {var.GetValue()}")

def __lldb_init_module(debugger, internal_dict):
    debugger.HandleCommand(
        'command script add -f my_module.my_command my_cmd')
```

### Scripted Process
```python
class MyScriptedProcess(lldb.SBScriptedProcess):
    def __init__(self, target, args):
        super().__init__(target, args)
        
    def get_thread_with_id(self, tid):
        # Return thread info
        pass
    
    def get_registers_for_thread(self, tid):
        # Return register context
        pass
    
    def read_memory_at_address(self, addr, size):
        # Read memory
        pass
```

### LLDB GUI Extensions
- **visual-lldb**: GUI frontend for LLDB
- **vegvisir**: Browser-based LLDB GUI
- **lldbg**: Lightweight native GUI
- **CodeLLDB**: VSCode debugger extension

## Clangd / Language Server Protocol

### Custom LSP Extensions
```cpp
// Implementing custom code actions
class MyCodeActionProvider {
public:
    std::vector<CodeAction> getCodeActions(
        const TextDocumentIdentifier &File,
        const Range &Range,
        const CodeActionContext &Context) {
        
        std::vector<CodeAction> Actions;
        
        // Add custom refactoring action
        CodeAction Action;
        Action.title = "Extract to function";
        Action.kind = "refactor.extract";
        
        // Define workspace edits
        WorkspaceEdit Edit;
        // ... populate edits
        Action.edit = Edit;
        
        Actions.push_back(Action);
        return Actions;
    }
};
```

### Clangd Configuration
```yaml
# .clangd configuration file
CompileFlags:
  Add: [-xc++, -std=c++17]
  Remove: [-W*]

Diagnostics:
  ClangTidy:
    Add: [modernize-*, performance-*]
    Remove: [modernize-use-trailing-return-type]

Index:
  Background: Build
```

## Source-to-Source Transformation

### Clang Rewriter
```cpp
#include "clang/Rewrite/Core/Rewriter.h"

class MyRewriter : public RecursiveASTVisitor<MyRewriter> {
    Rewriter &R;
    
public:
    MyRewriter(Rewriter &R) : R(R) {}
    
    bool VisitFunctionDecl(FunctionDecl *FD) {
        // Add comment before function
        R.InsertTextBefore(FD->getBeginLoc(), "// Auto-generated\n");
        
        // Replace function name
        R.ReplaceText(FD->getLocation(), 
                      FD->getName().size(), "new_name");
        
        return true;
    }
};
```

### RefactoringTool
```cpp
#include "clang/Tooling/Refactoring.h"

class MyRefactoring : public RefactoringCallback {
public:
    void run(const MatchFinder::MatchResult &Result) override {
        // Generate replacements
        Replacement Rep(
            *Result.SourceManager,
            CharSourceRange::getTokenRange(Range),
            "new_text");
        
        Replacements.insert(Rep);
    }
};
```

## Notable Tools Built with LLVM Tooling

### Analysis Tools
- **cppinsights**: C++ template instantiation visualization
- **ClangBuildAnalyzer**: Build time analysis
- **clazy**: Qt-specific static analysis

### Refactoring Tools
- **clang-rename**: Symbol renaming
- **clang-tidy**: Linting and auto-fixes
- **include-what-you-use**: Include optimization

### Code Generation
- **classgen**: Extract type info for IDA
- **constexpr-everything**: Auto-apply constexpr
- **clang-expand**: Inline function expansion

## Best Practices

1. **Use AST Matchers**: More readable than manual traversal
2. **Preserve Formatting**: Use Rewriter carefully to maintain style
3. **Handle Macros**: Be aware of macro expansion locations
4. **Test Thoroughly**: Edge cases in C++ are numerous
5. **Provide Good Diagnostics**: Clear error messages improve usability

## Resources

See Clang Plugins, Clangd/Language Server, and LLDB sections in README.md for comprehensive tool listings.

## Getting Detailed Information

When you need detailed and up-to-date resource links, tool lists, or project references, fetch the latest data from:

```
https://raw.githubusercontent.com/gmh5225/awesome-llvm-security/refs/heads/main/README.md
```

This README contains comprehensive curated lists of:
- Clang plugins and extensions (Clang Plugins section)
- Language server implementations (Clangd/Language Server section)
- LLDB debugger tools and GUIs (LLDB section)
- Static analysis and refactoring tools
