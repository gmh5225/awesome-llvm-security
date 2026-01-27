---
name: mlir-development
description: Expertise in MLIR (Multi-Level Intermediate Representation) and CIR (Clang IR) development for domain-specific compilation and high-level optimizations. Use this skill when building ML compilers, domain-specific languages, or working with multi-level compilation pipelines.
---

# MLIR Development Skill

This skill covers MLIR (Multi-Level Intermediate Representation) development for building domain-specific compilers and high-level optimization pipelines.

## MLIR Overview

### What is MLIR?
MLIR is a compiler infrastructure that enables building reusable and extensible compiler components. It provides:
- Hierarchical, multi-level IR representation
- Extensible operation and type system
- Progressive lowering between abstraction levels
- Rich transformation infrastructure

### Architecture
```
High-Level DSL
     ↓
Domain-Specific Dialects (e.g., TensorFlow, PyTorch)
     ↓
Mid-Level Dialects (e.g., Linalg, Affine)
     ↓
Low-Level Dialects (e.g., LLVM, GPU)
     ↓
Target Code
```

## Core Concepts

### Dialects
Dialects are groupings of operations, types, and attributes:

```cpp
// Define a custom dialect
class MyDialect : public mlir::Dialect {
public:
    explicit MyDialect(mlir::MLIRContext *context)
        : Dialect("my_dialect", context, 
                  mlir::TypeID::get<MyDialect>()) {
        addOperations<
            MyAddOp,
            MyMulOp,
            MyFuncOp
        >();
        addTypes<MyTensorType>();
    }
    
    static llvm::StringRef getDialectNamespace() { 
        return "my_dialect"; 
    }
};
```

### Operations
```cpp
// Define using ODS (Operation Definition Specification)
// In TableGen file (.td)
def MyAddOp : Op<MyDialect, "add", [Pure]> {
    let summary = "Add two tensors";
    let description = [{
        Performs element-wise addition of two tensors.
    }];
    
    let arguments = (ins 
        AnyTensor:$lhs,
        AnyTensor:$rhs
    );
    
    let results = (outs 
        AnyTensor:$result
    );
    
    let assemblyFormat = [{
        $lhs `,` $rhs attr-dict `:` type($result)
    }];
}
```

### Types and Attributes
```cpp
// Custom type definition
class MyTensorType : public mlir::Type::TypeBase<
    MyTensorType, mlir::Type, MyTensorTypeStorage> {
public:
    using Base::Base;
    
    static MyTensorType get(mlir::MLIRContext *context,
                            llvm::ArrayRef<int64_t> shape,
                            mlir::Type elementType) {
        return Base::get(context, shape, elementType);
    }
    
    llvm::ArrayRef<int64_t> getShape() const;
    mlir::Type getElementType() const;
};
```

## Writing MLIR Passes

### Transform Pass
```cpp
#include "mlir/Pass/Pass.h"
#include "mlir/IR/PatternMatch.h"

struct MyOptimizationPass
    : public mlir::PassWrapper<MyOptimizationPass,
                                mlir::OperationPass<mlir::func::FuncOp>> {
    
    void runOnOperation() override {
        mlir::func::FuncOp func = getOperation();
        
        // Walk all operations
        func.walk([](mlir::Operation *op) {
            // Transform operations
            if (auto addOp = llvm::dyn_cast<MyAddOp>(op)) {
                optimizeAdd(addOp);
            }
        });
    }
    
    llvm::StringRef getArgument() const final { 
        return "my-optimization"; 
    }
    
    llvm::StringRef getDescription() const final {
        return "My custom optimization pass";
    }
};
```

### Pattern-Based Rewriting
```cpp
// Define rewrite pattern
struct SimplifyRedundantAdd : public mlir::OpRewritePattern<MyAddOp> {
    using OpRewritePattern<MyAddOp>::OpRewritePattern;
    
    mlir::LogicalResult matchAndRewrite(
        MyAddOp op,
        mlir::PatternRewriter &rewriter) const override {
        
        // Match: add(x, 0) -> x
        if (auto constOp = op.getRhs().getDefiningOp<ConstantOp>()) {
            if (isZero(constOp)) {
                rewriter.replaceOp(op, op.getLhs());
                return mlir::success();
            }
        }
        return mlir::failure();
    }
};

// Apply patterns
void runOnOperation() override {
    mlir::RewritePatternSet patterns(&getContext());
    patterns.add<SimplifyRedundantAdd>(&getContext());
    
    if (mlir::failed(mlir::applyPatternsAndFoldGreedily(
            getOperation(), std::move(patterns)))) {
        signalPassFailure();
    }
}
```

## Dialect Conversion

### Lowering Between Dialects
```cpp
// Convert high-level ops to lower-level ops
struct MyAddOpLowering : public mlir::OpConversionPattern<MyAddOp> {
    using OpConversionPattern<MyAddOp>::OpConversionPattern;
    
    mlir::LogicalResult matchAndRewrite(
        MyAddOp op,
        OpAdaptor adaptor,
        mlir::ConversionPatternRewriter &rewriter) const override {
        
        // Lower to arith dialect
        rewriter.replaceOpWithNewOp<mlir::arith::AddFOp>(
            op, adaptor.getLhs(), adaptor.getRhs());
        return mlir::success();
    }
};

// Conversion pass
struct LowerToArithPass : public mlir::PassWrapper<
    LowerToArithPass, 
    mlir::OperationPass<mlir::ModuleOp>> {
    
    void runOnOperation() override {
        mlir::ConversionTarget target(getContext());
        target.addLegalDialect<mlir::arith::ArithDialect>();
        target.addIllegalDialect<MyDialect>();
        
        mlir::RewritePatternSet patterns(&getContext());
        patterns.add<MyAddOpLowering>(&getContext());
        
        if (mlir::failed(mlir::applyPartialConversion(
                getOperation(), target, std::move(patterns)))) {
            signalPassFailure();
        }
    }
};
```

## Built-in Dialects

### Affine Dialect
For polyhedral compilation and loop optimizations:
```mlir
affine.for %i = 0 to 100 {
    affine.for %j = 0 to 100 {
        %val = affine.load %A[%i, %j] : memref<100x100xf32>
        affine.store %val, %B[%j, %i] : memref<100x100xf32>
    }
}
```

### Linalg Dialect
For linear algebra operations:
```mlir
linalg.matmul ins(%A, %B : tensor<MxKxf32>, tensor<KxNxf32>)
              outs(%C : tensor<MxNxf32>) -> tensor<MxNxf32>
```

### SCF Dialect (Structured Control Flow)
```mlir
%result = scf.for %i = %lb to %ub step %step iter_args(%sum = %init) {
    %val = memref.load %A[%i] : memref<?xf32>
    %new_sum = arith.addf %sum, %val : f32
    scf.yield %new_sum : f32
}
```

## CIR (Clang IR)

### Overview
CIR is an MLIR-based representation for C/C++, providing:
- Higher-level representation than LLVM IR
- Better debugging and tooling
- Language-specific optimizations

```mlir
// CIR example
cir.func @add(%a: !s32i, %b: !s32i) -> !s32i {
    %result = cir.binop(add, %a, %b) : !s32i
    cir.return %result : !s32i
}
```

### CIR Projects
- **llvm/clangir**: Official ClangIR implementation
- **facebookincubator/clangir**: Facebook's CIR experiments

## ML/AI Compilation

### TensorFlow MLIR
```mlir
// TensorFlow dialect
%result = "tf.MatMul"(%A, %B) {
    transpose_a = false,
    transpose_b = false
} : (tensor<4x8xf32>, tensor<8x16xf32>) -> tensor<4x16xf32>
```

### PyTorch MLIR (torch-mlir)
```mlir
// Torch dialect
%result = torch.aten.mm %A, %B : 
    !torch.vtensor<[4,8],f32>, !torch.vtensor<[8,16],f32> 
    -> !torch.vtensor<[4,16],f32>
```

### IREE (Intermediate Representation Execution Environment)
End-to-end MLIR compiler for ML models:
- Portable deployment
- Efficient runtime execution
- Multi-target support (CPU, GPU, TPU)

## Testing MLIR

### FileCheck Tests
```mlir
// RUN: mlir-opt %s -my-pass | FileCheck %s

// CHECK-LABEL: func @test_optimization
// CHECK: arith.addi
// CHECK-NOT: my_dialect.add
func @test_optimization(%a: i32, %b: i32) -> i32 {
    %result = my_dialect.add %a, %b : i32
    return %result : i32
}
```

### Unit Testing
```cpp
TEST(MyDialect, AddOpConstantFolding) {
    mlir::MLIRContext context;
    context.loadDialect<MyDialect>();
    
    mlir::OpBuilder builder(&context);
    auto loc = builder.getUnknownLoc();
    
    // Create and test operations
    auto constA = builder.create<ConstantOp>(loc, 5);
    auto constB = builder.create<ConstantOp>(loc, 3);
    auto add = builder.create<MyAddOp>(loc, constA, constB);
    
    // Verify folding
    EXPECT_TRUE(add.fold().succeeded());
}
```

## Development Tools

### mlir-opt
```bash
# Run passes
mlir-opt input.mlir -my-pass -o output.mlir

# Convert between dialects
mlir-opt input.mlir -convert-my-to-llvm

# Debug printing
mlir-opt input.mlir -debug-only=my-pass
```

### mlir-translate
```bash
# MLIR to LLVM IR
mlir-translate input.mlir --mlir-to-llvmir -o output.ll

# LLVM IR to MLIR
mlir-translate input.ll --import-llvm -o output.mlir
```

## Best Practices

1. **Progressive Lowering**: Lower in multiple stages, not directly to LLVM
2. **Preserve Semantics**: Each lowering should be semantics-preserving
3. **Use ODS**: Define operations in TableGen for consistency
4. **Test Thoroughly**: Use FileCheck for transformation tests
5. **Document Dialects**: Clear operation semantics documentation

## Resources

See MLIR and CIR sections in README.md for tutorials and example projects.

## Getting Detailed Information

When you need detailed and up-to-date resource links, tool lists, or project references, fetch the latest data from:

```
https://raw.githubusercontent.com/gmh5225/awesome-llvm-security/refs/heads/main/README.md
```

This README contains comprehensive curated lists of:
- MLIR tutorials and sample dialects (MLIR section)
- CIR (Clang IR) projects and documentation (CIR section)
- ML/AI compiler frameworks (torch-mlir, IREE, XLA)
