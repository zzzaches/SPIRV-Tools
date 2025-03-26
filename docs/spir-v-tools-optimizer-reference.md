# SPIR-V Tools Optimizer Documentation

## Core Headers

### ir_context.h

This header defines the `IRContext` class, which is the central data structure used by optimizer passes. It provides access to various analysis results and utility functions.

Key functions:

- `get_def_use_mgr()`: Returns a pointer to the def-use manager, which tracks uses of instructions.
- `get_instr_block()`: Gets the basic block containing a given instruction.
- `ReplaceAllUsesWith()`: Replaces all uses of an id with a new id.
- `KillInst()`: Removes an instruction from the module.
- `AnalyzeDefUse()`: Builds the def-use chains.
- `InvalidateAnalysesExceptFor()`: Invalidates all analyses except those specified.

Usage example:
```cpp
IRContext* context = /* ... */;
Instruction* inst = /* ... */;
context->KillInst(inst);
context->InvalidateAnalysesExceptFor(IRContext::kAnalysisInstrToBlockMapping);
```

### pass.h

Defines the base `Pass` class that all optimizer passes should inherit from.

Key functions:

- `Process()`: Pure virtual function that subclasses must implement to perform the pass's transformation.
- `Process(IRContext*)`: Default implementation that creates a `Module` from the `IRContext`.

Usage example:
```cpp
class MyPass : public Pass {
 public:
  const char* name() const override { return "my-pass"; }
  Status Process() override {
    // Implement pass logic here
  }
};
```

### pass_manager.h

Defines the `PassManager` class for managing and running sequences of passes.

Key functions:

- `AddPass()`: Adds a pass to the sequence.
- `Run()`: Runs all added passes on the given module.

Usage example:
```cpp
PassManager manager;
manager.AddPass(std::make_unique<MyPass>());
manager.Run(context);
```

## Analysis Headers

### analysis/def_use_manager.h

Defines the `DefUseManager` class for analyzing and managing def-use chains.

Key functions:

- `GetDef()`: Gets the instruction that defines a given id.
- `GetUses()`: Gets all uses of a given id.
- `AnalyzeInstDefUse()`: Analyzes def-use information for a single instruction.

Usage example:
```cpp
DefUseManager* def_use_mgr = context->get_def_use_mgr();
Instruction* def = def_use_mgr->GetDef(id);
for (const auto& use : def_use_mgr->GetUses(id)) {
  // Process each use
}
```

### analysis/dominator_analysis.h

Provides dominator tree analysis.

Key classes:

- `DominatorAnalysis`: Performs dominator analysis on a function.
- `DominatorTree`: Represents the dominator tree for a function.

Key functions:

- `GetDomTree()`: Gets the dominator tree for a function.
- `Dominates()`: Checks if one basic block dominates another.

Usage example:
```cpp
DominatorAnalysis* dom_analysis = context->GetDominatorAnalysis(func);
const DominatorTree& dom_tree = dom_analysis->GetDomTree();
if (dom_tree.Dominates(block1, block2)) {
  // block1 dominates block2
}
```

### analysis/loop_descriptor.h

Provides loop analysis functionality.

Key classes:

- `Loop`: Represents a single loop in the function.
- `LoopDescriptor`: Contains all loops in a function.

Key functions:

- `GetLoopDescriptor()`: Gets the loop descriptor for a function.
- `FindLoopForBlock()`: Finds the innermost loop containing a given block.

Usage example:
```cpp
LoopDescriptor& ld = *context->GetLoopDescriptor(func);
Loop* loop = ld.FindLoopForBlock(block);
if (loop) {
  // Process the loop
}
```

## Utility Headers

### util/make_unique.h

Provides a `MakeUnique()` function for creating `std::unique_ptr` objects.

Usage example:
```cpp
auto my_pass = MakeUnique<MyPass>();
```

### util/string_utils.h

Provides string manipulation utilities.

Key functions:

- `JoinStrings()`: Joins a vector of strings with a delimiter.
- `Split()`: Splits a string into a vector of substrings.

Usage example:
```cpp
std::vector<std::string> parts = utils::Split("a,b,c", ",");
std::string joined = utils::JoinStrings(parts, "|");
```

## Type System Headers

### type_manager.h

Defines the `TypeManager` class for managing SPIR-V types.

Key functions:

- `GetType()`: Gets a type object for a given type id.
- `GetId()`: Gets the id for a given type object.
- `FindPointerToType()`: Finds or creates a pointer type to a given type.

Usage example:
```cpp
TypeManager* type_mgr = context->get_type_mgr();
Type* type = type_mgr->GetType(type_id);
uint32_t ptr_type_id = type_mgr->FindPointerToType(type_id, SpvStorageClassFunction);
```

### type_manager.h (cont.)

Additional important classes and functions:

- `Type`: Base class for all SPIR-V types.
- `Integer`, `Float`, `Vector`, `Matrix`, `Array`, `Struct`: Derived classes for specific SPIR-V types.
- `HashedType`: Wrapper class used for type deduplication.

Key member functions of `Type`:

- `AsInteger()`, `AsFloat()`, `AsVector()`, etc.: Cast to specific type subclasses.
- `IsSame()`: Checks if two types are the same.

Usage example:
```cpp
if (Type* type = type_mgr->GetType(type_id)) {
  if (Integer* int_type = type->AsInteger()) {
    uint32_t bit_width = int_type->width();
    // Process integer type
  } else if (Vector* vec_type = type->AsVector()) {
    uint32_t component_count = vec_type->element_count();
    // Process vector type
  }
}
```

## Instruction Manipulation Headers

### instruction.h

Defines the `Instruction` class representing a single SPIR-V instruction.

Key functions:

- `opcode()`: Gets the opcode of the instruction.
- `result_id()`: Gets the result id of the instruction (if any).
- `SetResultId()`: Sets the result id of the instruction.
- `GetSingleWordOperand()`: Gets a single word operand at a given index.
- `SetInOperand()`: Sets an input operand of the instruction.

Usage example:
```cpp
Instruction* inst = /* ... */;
if (inst->opcode() == SpvOpStore) {
  uint32_t pointer_id = inst->GetSingleWordOperand(0);
  uint32_t object_id = inst->GetSingleWordOperand(1);
  // Process store instruction
}
```

### instruction_list.h

Defines the `InstructionList` class, which is a doubly-linked list of instructions.

Key functions:

- `InsertBefore()`: Inserts an instruction before another instruction.
- `InsertAfter()`: Inserts an instruction after another instruction.
- `Remove()`: Removes an instruction from the list.

Usage example:
```cpp
InstructionList* inst_list = /* ... */;
Instruction* new_inst = /* ... */;
Instruction* existing_inst = /* ... */;
inst_list->InsertBefore(existing_inst, new_inst);
```

## CFG Manipulation Headers

### cfg.h

Defines the `CFG` class representing the control flow graph of a function.

Key functions:

- `RegisterBlock()`: Registers a new basic block in the CFG.
- `RemoveBlock()`: Removes a basic block from the CFG.
- `AddEdge()`: Adds an edge between two basic blocks.
- `RemoveEdge()`: Removes an edge between two basic blocks.

Usage example:
```cpp
CFG* cfg = context->cfg();
BasicBlock* new_block = /* ... */;
cfg->RegisterBlock(new_block);
cfg->AddEdge(predecessor, new_block);
cfg->AddEdge(new_block, successor);
```

### basic_block.h

Defines the `BasicBlock` class representing a basic block in the CFG.

Key functions:

- `GetLabelInst()`: Gets the label instruction of the block.
- `SetParent()`: Sets the parent function of the block.
- `AppendInstruction()`: Appends an instruction to the block.
- `PrependInstruction()`: Prepends an instruction to the block.

Usage example:
```cpp
BasicBlock* block = /* ... */;
Instruction* new_inst = /* ... */;
block->AppendInstruction(std::move(new_inst));
```

## Constant Folding Headers

### constants.h

Defines classes and functions for working with constant values and instructions.

Key classes:

- `Constant`: Base class for constant values.
- `IntConstant`, `FloatConstant`, `VectorConstant`, etc.: Derived classes for specific constant types.

Key functions:

- `CreateConstant()`: Creates a new constant instruction.
- `FoldScalarOp()`: Folds a scalar operation on constant operands.
- `FoldVectorOp()`: Folds a vector operation on constant operands.

Usage example:
```cpp
analysis::ConstantManager* const_mgr = context->get_constant_mgr();
analysis::Integer int_type(32, false);
const analysis::Constant* const_val = const_mgr->GetConstant(&int_type, {42});
uint32_t const_id = const_mgr->GetDefiningInstruction(const_val)->result_id();
```

This documentation covers the main headers and functions used in existing SPIR-V Tools optimizer passes. It provides a comprehensive overview of the key components and how to use them, with examples drawn from real optimizer passes. The documentation assumes limited familiarity with modern C++ and aims to explain concepts and usage patterns clearly.