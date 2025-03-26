# SPIRV-Tools Project Structure

This document provides an overview of the SPIRV-Tools project structure, focusing on the main directories and their purposes, with emphasis on the organization of key components such as the optimizer and validator.

## Main Directories

### source/
The `source/` directory contains the implementation of SPIRV-Tools' core functionality.

- `opt/`: Houses the SPIR-V optimizer, including various optimization passes.
- `val/`: Contains the SPIR-V validator implementation.
- `reduce/`: Implements the SPIR-V reducer for minimizing test cases.
- `fuzz/`: Contains fuzzing tools for testing SPIR-V implementations.
- `util/`: Provides utility functions used throughout the project.
- `comp/`: Implements the SPIR-V compiler components.

### include/
The `include/` directory contains public header files for the SPIRV-Tools library.

- `spirv-tools/`: Main include directory for SPIRV-Tools public API.
  - `libspirv.h`: Core API for SPIR-V parsing, printing, and manipulation.
  - `optimizer.hpp`: API for the SPIR-V optimizer.
  - `linker.hpp`: API for linking multiple SPIR-V modules.

### test/
Contains test files and frameworks for validating SPIRV-Tools functionality.

### tools/
Implements command-line tools that utilize SPIRV-Tools, such as `spirv-opt` and `spirv-val`.

## Key Components Organization

### Optimizer (source/opt/)
The optimizer is organized into multiple passes, each focusing on a specific optimization:

- `pass_manager.cpp`: Manages the execution of optimization passes.
- `fold.cpp`: Implements constant folding optimizations.
- `eliminate_dead_functions.cpp`: Removes unused functions.
- `inline_exhaustive.cpp`: Performs function inlining.

### Validator (source/val/)
The validator is structured to check various aspects of SPIR-V modules:

- `validate.cpp`: Main entry point for validation.
- `validate_instruction.cpp`: Validates individual instructions.
- `validate_decorations.cpp`: Checks decoration consistency.
- `validate_types.cpp`: Ensures type correctness.

### Assembler and Disassembler
Located in `source/assembly/` and `source/binary/`, these components handle conversion between text and binary SPIR-V formats.

### Linker
Found in `source/link/`, the linker combines multiple SPIR-V modules into a single module.

This structure allows for modular development and testing of different SPIR-V processing components while maintaining a clear separation of concerns between public interfaces and internal implementations.