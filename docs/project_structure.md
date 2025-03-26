# SPIRV-Tools Project Structure

This document provides an overview of the SPIRV-Tools project structure, focusing on the main directories and their purposes, with emphasis on the organization of key components such as the optimizer and validator.

## Main Directories

### source/
The `source/` directory contains the implementation of SPIRV-Tools' core functionality.

Key subdirectories:
- `opt/`: Contains the SPIR-V optimizer implementation.
- `val/`: Houses the SPIR-V validator implementation.
- `reduce/`: Implements the SPIR-V reducer functionality.
- `fuzz/`: Contains fuzzing tools for testing SPIR-V.
- `util/`: Provides utility functions used across the project.

### include/
The `include/` directory contains public header files for the SPIRV-Tools library.

Key subdirectories:
- `spirv-tools/`: Contains the main public API headers.
- `spirv-tools/opt/`: Houses headers for the optimizer.
- `spirv-tools/val/`: Contains headers for the validator.

### test/
The `test/` directory contains test cases and fixtures for the project.

### tools/
The `tools/` directory contains command-line tools built on top of the SPIRV-Tools library.

## Key Components Organization

### Optimizer (source/opt/)
The optimizer is organized into several passes, each implemented in its own file. The main optimizer driver is implemented in `optimizer.cpp`.

Key files:
- `pass_manager.h`: Manages the execution of optimization passes.
- `instruction.h`: Represents SPIR-V instructions for optimization.
- Various pass implementations (e.g., `dead_branch_elim_pass.cpp`, `eliminate_dead_functions_pass.cpp`).

### Validator (source/val/)
The validator is structured around different validation rules and checks.

Key files:
- `validate.cpp`: Main entry point for validation.
- `validation_state.h`: Maintains the state during validation.
- Various validation rule implementations (e.g., `validate_decorations.cpp`, `validate_instructions.cpp`).

### Assembler and Disassembler
Located in `source/assembly/`, these components handle the textual representation of SPIR-V.

### Binary Parser
Found in `source/binary.cpp`, this component is responsible for parsing SPIR-V binary format.

## Build System
The project uses CMake for its build system, with the main `CMakeLists.txt` file in the root directory and additional configuration files in subdirectories.

This structure allows for modular development and testing of different SPIR-V processing components while maintaining a cohesive overall project organization.