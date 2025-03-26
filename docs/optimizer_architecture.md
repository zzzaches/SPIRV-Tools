# SPIRV-Tools Optimizer Architecture

The SPIRV-Tools optimizer uses a modular pass-based architecture to transform and optimize SPIR-V modules. The key components are:

## IRContext

The IRContext class serves as the central data structure, containing:

- The SPIR-V module being optimized
- Analysis results (e.g. def-use chains, control flow graph)
- Utility functions for manipulating the IR

It manages the lifecycle of analyses, invalidating and rebuilding them as the IR is modified.

## PassManager

The PassManager class is responsible for:

- Maintaining the list of optimization passes to run
- Executing passes in sequence on the IR
- Handling pass dependencies and analysis preservation

## Optimization Passes

Individual optimization passes inherit from the Pass base class and implement the Run() method to transform the IR. Passes can:

- Query analyses from the IRContext
- Modify the IR using IRContext helper methods  
- Preserve or invalidate analyses as appropriate

## Optimization Pipeline

```
              ┌─────────────────┐
              │    IRContext    │
              └─────────────────┘
                       │
                       ▼
              ┌─────────────────┐
              │   PassManager   │
              └─────────────────┘
                       │
                       ▼
        ┌───────────────────────────┐
        │                           │
  ┌─────┴─────┐             ┌───────┴───────┐
  │  Pass 1   │      ...    │    Pass N     │
  └─────┬─────┘             └───────┬───────┘
        │                           │
        └───────────────────────────┘
                       │
                       ▼
            Optimized SPIR-V Module
```

The PassManager runs a sequence of passes on the IRContext containing the SPIR-V module. Each pass can query and update analyses in the IRContext as needed. The result is an optimized SPIR-V module.

This architecture allows for flexible composition of optimization passes while managing analysis lifecycles and dependencies between passes.