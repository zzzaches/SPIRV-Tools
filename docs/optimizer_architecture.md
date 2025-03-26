# SPIRV-Tools Optimizer Architecture

The SPIRV-Tools optimizer is designed to perform optimizations and transformations on SPIR-V modules. The key components of the optimizer architecture are:

1. IRContext
2. Pass Manager 
3. Optimization Passes

## IRContext

The IRContext is the central data structure that maintains the SPIR-V module and all associated analyses. It provides:

- Access to the SPIR-V module instructions
- Management of analyses like def-use chains, control flow graph, etc.
- Utility functions for manipulating the IR

Key features:

- Lazy analysis - Analyses are only computed when requested
- Analysis invalidation - Tracks which analyses are valid after IR changes
- Centralized IR manipulation - All changes to the IR go through the IRContext

## Pass Manager

The PassManager is responsible for:

- Maintaining the list of passes to run
- Executing passes in order
- Handling analysis invalidation between passes

Key methods:

- AddPass() - Adds a pass to be run
- Run() - Executes all added passes on the given IRContext

## Optimization Passes 

Individual optimization passes inherit from the Pass base class and implement:

- Run() - Performs the actual optimization on the IR
- Analysis requirements - Specifies which analyses are required/preserved

Passes communicate with the rest of the optimizer solely through the IRContext.

## Optimization Pipeline

```
                  +-------------+
                  |             |
                  | Pass Manager|
                  |             | 
                  +------+------+
                         |
                         v
+----------+    +--------+----------+    +-----------+
|          |    |                   |    |           |
| Pass 1   +--->|     IRContext     +---->  Pass 2   |
|          |    |                   |    |           | 
+----------+    +-------------------+    +-----------+
                    ^           ^
                    |           |
                +---+----+  +---+----+
                |        |  |        |
                |Analysis|  |Analysis|
                |   A    |  |   B    |
                |        |  |        |
                +--------+  +--------+
```

The PassManager runs passes sequentially on the IRContext. Each pass can query analyses from the IRContext as needed. The IRContext handles invalidating and recomputing analyses between passes.

This architecture allows for modular passes that can be easily combined into optimization pipelines, with efficient handling of analyses.