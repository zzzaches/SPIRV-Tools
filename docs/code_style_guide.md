# SPIRV-Tools Code Style Guide

This document outlines the coding conventions and best practices for contributing C++ code to the SPIRV-Tools project.

## Naming Conventions

### File Names
- Use lowercase with underscores for file names (e.g., `ir_context.h`, `pass_manager.cpp`)
- Header files use the `.h` extension, source files use `.cpp`

### Variable Names  
- Use lowercase with underscores for variable names (e.g., `def_use_mgr_`, `consumer_`)
- Member variables should end with an underscore (e.g., `module_`, `unique_id_`)
- Constants and enums should be ALL_CAPS with underscores (e.g., `kAnalysisNone`, `SPV_ENV_UNIVERSAL_1_2`)

### Function Names
- Use CamelCase for function names (e.g., `BuildDefUseManager()`, `GetMemberName()`)
- Getters can omit the "Get" prefix if brief (e.g., `consumer()` instead of `GetConsumer()`)

### Class Names  
- Use CamelCase for class names (e.g., `IRContext`, `PassManager`)

### Namespace Names
- Use lowercase for namespaces (e.g., `spvtools::opt`)

## Formatting

- Use 2 space indentation (no tabs)
- Place opening braces on the same line as the statement
- Place else statements on the same line as the closing brace
- Use spaces around operators and after commas
- Limit lines to 80 characters where possible
- Use C++11 style for null pointer (`nullptr`)
- Use `//` for single line comments, `/* */` for multi-line

## Best Practices

- Use `const` for variables and parameters that don't change
- Prefer enums for named constants instead of macros
- Use `override` keyword for virtual function overrides
- Initialize member variables in the initializer list
- Use `std::unique_ptr` for exclusive ownership of dynamically allocated objects
- Prefer `enum class` over plain enums to avoid name conflicts
- Use `explicit` for single argument constructors to prevent implicit conversions
- Avoid using exceptions, use error codes or status objects instead
- Use forward declarations to reduce header dependencies where possible

## Header Files

- Use `#pragma once` for header guards
- Order includes as: related header, C system headers, C++ system headers, other library headers, project headers
- Use angle brackets for system and library includes, quotes for project includes

## Classes

- Declare public members first, then protected, then private
- Use the `explicit` keyword for single argument constructors
- Prefer composition over inheritance where possible
- Make destructors virtual if the class is meant to be inherited from

## Functions

- Keep functions short and focused on a single task
- Use early returns to simplify logic
- Pass objects by const reference when possible, unless passing by value is more efficient

## Error Handling

- Use `assert` for internal logic checks that should never fail
- Return status codes or use `Optional<T>` for recoverable errors
- Use logging for diagnostic information (via the `MessageConsumer`)

## Memory Management

- Prefer stack allocation over heap allocation
- Use smart pointers (`std::unique_ptr`, `std::shared_ptr`) for dynamic allocation
- Avoid raw `new` and `delete` operations

## Miscellaneous

- Use `auto` when it improves readability, but be mindful of overuse
- Prefer pre-increment (`++i`) over post-increment (`i++`) when either will do
- Use range-based for loops when possible
- Use `nullptr` instead of `NULL` or `0` for null pointers

Remember to keep the code consistent with the existing SPIRV-Tools codebase and follow any project-specific conventions not explicitly mentioned here.