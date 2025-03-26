# SPIRV-Tools Optimizer Documentation

## Table of Contents
1. Introduction
2. Basic Concepts
3. Adding and Modifying Variables
4. Creating Additional Shader Arguments
5. OpenCL vs Vulkan Dialects
6. Modifying Control Flow
7. Existing Optimizer Passes
8. Creating New Optimizer Passes
9. Important Optimizer Functions
10. Best Practices

## 1. Introduction

The SPIRV-Tools optimizer is a powerful framework for analyzing and transforming SPIR-V shaders. This documentation aims to provide a comprehensive guide for users who want to modify existing shaders or create new optimization passes.

## 2. Basic Concepts

SPIR-V is a binary intermediate representation for graphics shaders and compute kernels. The SPIRV-Tools optimizer works on this representation, allowing various transformations and optimizations.

Key concepts:
- Instructions: Basic units of SPIR-V code
- Basic Blocks: Sequences of instructions
- Functions: Collections of basic blocks
- Modules: Top-level containers for SPIR-V code

## 3. Adding and Modifying Variables

### Adding Variables

To add a new variable:

1. Create a new `OpVariable` instruction
2. Set the storage class and type
3. Add the variable to the module's global variables

Example:
```cpp
spv::Op opcode = spv::Op::OpVariable;
uint32_t result_type = context->get_type_mgr()->GetId(type);
uint32_t storage_class = static_cast<uint32_t>(spv::StorageClass::Function);
uint32_t new_var_id = context->TakeNextId();

std::vector<uint32_t> words{opcode, result_type, new_var_id, storage_class};
context->module()->AddGlobalValue(spv::Op::OpVariable, new_var_id, result_type, words);
```

### 3.2 Modifying Variables

To modify an existing variable:

1. Locate the variable's defining instruction
2. Update the instruction's operands

Example:
```cpp
auto* var_inst = context->get_def_use_mgr()->GetDef(variable_id);
var_inst->SetOperand(2, new_storage_class);
```

## 4. Creating Additional Shader Arguments

### Adding Input/Output Variables

1. Create a new variable with `Input` or `Output` storage class
2. Add appropriate decorations (e.g., `Location`)

Example:
```cpp
uint32_t var_id = context->TakeNextId();
context->module()->AddGlobalValue(spv::Op::OpVariable, var_id, type_id,
    {spv::Op::OpVariable, type_id, var_id, static_cast<uint32_t>(spv::StorageClass::Input)});

context->get_decoration_mgr()->AddDecoration(var_id, spv::Decoration::Location, {location});
```

### 4.2 Adding Uniform Buffer Objects (UBOs)

1. Create a struct type for the UBO
2. Create a variable with `Uniform` storage class
3. Add `Block` decoration to the struct
4. Add `Binding` and `DescriptorSet` decorations to the variable

Example:
```cpp
uint32_t struct_id = context->TakeNextId();
context->module()->AddType(spv::Op::OpTypeStruct, struct_id, {member_type_ids});

uint32_t ptr_type_id = context->get_type_mgr()->FindPointerToType(struct_id, spv::StorageClass::Uniform);
uint32_t var_id = context->TakeNextId();
context->module()->AddGlobalValue(spv::Op::OpVariable, var_id, ptr_type_id,
    {spv::Op::OpVariable, ptr_type_id, var_id, static_cast<uint32_t>(spv::StorageClass::Uniform)});

context->get_decoration_mgr()->AddDecoration(struct_id, spv::Decoration::Block);
context->get_decoration_mgr()->AddDecoration(var_id, spv::Decoration::Binding, {binding});
context->get_decoration_mgr()->AddDecoration(var_id, spv::Decoration::DescriptorSet, {set});
```

## 5. OpenCL vs Vulkan Dialects

### 5.1 OpenCL-specific Features

- Kernel functions (`OpEntryPoint` with `Kernel` execution model)
- `__local`, `__global`, `__constant` address spaces
- `get_global_id`, `get_local_id` built-in functions

Example (adding a kernel function):
```cpp
uint32_t func_id = context->TakeNextId();
context->AddEntryPoint(spv::ExecutionModel::Kernel, func_id, "kernel_name", {});
```

### Vulkan-specific Features

- Shader stages (Vertex, Fragment, Compute, etc.)
- Specialized descriptor sets and bindings
- Push constants

Example (adding a Vulkan shader entry point):
```cpp
uint32_t func_id = context->TakeNextId();
context->AddEntryPoint(spv::ExecutionModel::GLCompute, func_id, "main", {});
```

## 6. Modifying Control Flow

### 6.1 Adding Basic Blocks

1. Create a new basic block
2. Add instructions to the block
3. Update control flow instructions (e.g., `OpBranch`)

Example:
```cpp
auto* function = context->GetFunction(func_id);
auto* new_block = function->AddBasicBlock();
new_block->AddInstruction(MakeUnique<Instruction>(context, spv::Op::OpLabel, 0, context->TakeNextId(), {}));

// Add more instructions to the block
// ...

// Update control flow
auto* previous_block = /* get previous block */;
previous_block->AddInstruction(MakeUnique<Instruction>(
    context, spv::Op::OpBranch, 0, 0, {{SPV_OPERAND_TYPE_ID, {new_block->id()}}}));
```

### 6.2 Adding Loops

1. Create loop header, body, and continue blocks
2. Add phi instructions for loop variables
3. Add branch instructions for loop control

Example:
```cpp
auto* function = context->GetFunction(func_id);
auto* header = function->AddBasicBlock();
auto* body = function->AddBasicBlock();
auto* continue_block = function->AddBasicBlock();
auto* merge = function->AddBasicBlock();

// Add instructions to blocks
// ...

// Add loop control instructions
header->AddInstruction(MakeUnique<Instruction>(
    context, spv::Op::OpLoopMerge, 0, 0, 
    {{SPV_OPERAND_TYPE_ID, {merge->id()}}, {SPV_OPERAND_TYPE_ID, {continue_block->id()}}, 
     {SPV_OPERAND_TYPE_LOOP_CONTROL, {spv::LoopControlMask::None}}}));
```

### Adding Conditionals

1. Create condition, true, and false blocks
2. Add branch instructions for conditional control

Example:
```cpp
auto* function = context->GetFunction(func_id);
auto* condition = function->AddBasicBlock();
auto* true_block = function->AddBasicBlock();
auto* false_block = function->AddBasicBlock();
auto* merge = function->AddBasicBlock();

// Add instructions to blocks
// ...

// Add conditional control instructions
condition->AddInstruction(MakeUnique<Instruction>(
    context, spv::Op::OpSelectionMerge, 0, 0, 
    {{SPV_OPERAND_TYPE_ID, {merge->id()}}, {SPV_OPERAND_TYPE_SELECTION_CONTROL, {spv::SelectionControlMask::None}}}));
condition->AddInstruction(MakeUnique<Instruction>(
    context, spv::Op::OpBranchConditional, 0, 0, 
    {{SPV_OPERAND_TYPE_ID, {condition_var_id}}, {SPV_OPERAND_TYPE_ID, {true_block->id()}}, 
     {SPV_OPERAND_TYPE_ID, {false_block->id()}}}));
```

## 7. Existing Optimizer Passes

### 7.1 Dead Code Elimination (DCE)

Removes instructions that don't contribute to the shader's output.

Usage:
```cpp
context->RegisterPass(CreateDeadCodeEliminationPass());
```

### Common Subexpression Elimination (CSE)

Identifies and removes redundant computations.

Usage:
```cpp
context->RegisterPass(CreateCommonSubexpressionEliminationPass());
```

### Aggressive Dead Code Elimination (ADCE)

A more aggressive version of DCE that can remove entire functions.

Usage:
```cpp
context->RegisterPass(CreateAggressiveDCEPass());
```

### 7.4 Scalar Replacement of Aggregates

Breaks down aggregate types (structs, arrays) into scalar variables.

Usage:
```cpp
context->RegisterPass(CreateScalarReplacementPass());
```

### 7.5 Loop Unrolling

Replaces loops with repeated code sequences.

Usage:
```cpp
context->RegisterPass(CreateLoopUnrollPass());
```

## 8. Creating New Optimizer Passes

To create a new optimizer pass:

1. Create a new class inheriting from `Pass`
2. Implement the `Process()` method
3. Register the pass with the optimizer

Example:
```cpp
class MyCustomPass : public Pass {
 public:
  const char* name() const override { return "my-custom-pass"; }
  Status Process() override {
    // Implement your pass logic here
    return Status::SuccessWithoutChange;
  }
};

// Usage
context->RegisterPass(MakeUnique<MyCustomPass>());
```

## 9. Important Optimizer Functions

### 9.1 Context Management

- `IRContext::get_module()`: Access the current SPIR-V module
- `IRContext::get_def_use_mgr()`: Access the def-use manager
- `IRContext::get_instr_block()`: Get the basic block containing an instruction

### 9.2 Type Management

- `IRContext::get_type_mgr()`: Access the type manager
- `TypeManager::GetId()`: Get the ID of a type
- `TypeManager::GetType()`: Get the type for an ID

### 9.3 Decoration Management

- `IRContext::get_decoration_mgr()`: Access the decoration manager
- `DecorationManager::AddDecoration()`: Add a decoration to an ID
- `DecorationManager::RemoveDecoration()`: Remove a decoration from an ID

### 9.4 Instruction Manipulation

- `Instruction::SetOpcode()`: Change an instruction's opcode
- `Instruction::SetResultId()`: Set the result ID of an instruction
- `Instruction::SetInOperand()`: Set an input operand of an instruction

## 10. Best Practices

1. Always use the IRContext to access and modify the SPIR-V module
2. Maintain the SSA (Static Single Assignment) form when modifying code
3. Update the def-use manager after modifying instructions
4. Use the type manager to create and manage types consistently
5. Properly handle decorations when modifying variables or types
6. Verify the validity of the SPIR-V module after applying passes
7. Use existing utility functions and passes when possible to avoid duplication
8. Document your passes thoroughly, including their purpose and limitations
9. Write unit tests for your passes to ensure correctness
10. Be mindful of different SPIR-V versions and capabilities when writing passes

By following this documentation, users should be able to effectively use the SPIRV-Tools optimizer to modify existing shaders and create new optimization passes.