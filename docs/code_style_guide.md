# SPIRV-Tools Code Style Guide

This document outlines the code style guidelines for contributing to the SPIRV-Tools project. Following these conventions helps maintain consistency and readability across the codebase.

## Naming Conventions

### General
- Use meaningful and descriptive names for variables, functions, and classes
- Use PascalCase for class/struct names: `class IRContext`
- Use snake_case for variable and function names: `def_use_mgr_`
- Use UPPER_CASE for constants and macros: `kAnalysisNone`

### File Names
- Use snake_case for file names: `instruction.h`, `pass_manager.cpp`

### Member Variables  
- Use trailing underscore for private member variables: `module_`

## Formatting

### Indentation
- Use 2 spaces for indentation, not tabs

### Line Length
- Limit lines to 80 characters where possible

### Braces
- Place opening braces on the same line for functions and control structures:

```cpp
if (condition) {
  // code
}

void MyFunction() {
  // code  
}
```

### Spaces
- Place a space after `if`, `for`, `while`, and `switch`
- Place a space before and after binary operators: `a + b`
- No space between function name and opening parenthesis: `MyFunction()`

## Best Practices

### Comments
- Use `//` for single line comments
- Use `/* */` for multi-line comments
- Add comments for complex logic or non-obvious code

### Header Files
- Use `#ifndef` include guards in header files:

```cpp
#ifndef SOURCE_OPT_INSTRUCTION_H_
#define SOURCE_OPT_INSTRUCTION_H_

// Code

#endif  // SOURCE_OPT_INSTRUCTION_H_
```

### Forward Declarations
- Use forward declarations when possible to reduce dependencies

### const Correctness
- Use `const` for variables and parameters that should not be modified

### Memory Management
- Prefer smart pointers (`std::unique_ptr`, `std::shared_ptr`) over raw pointers

### Error Handling
- Use assertions (`assert`) for internal logic checks
- Use error codes or exceptions for recoverable errors

### Namespaces
- Use namespaces to organize code: `namespace spvtools { namespace opt { } }`
- Avoid `using namespace` in header files

### Classes
- Declare public members first, followed by protected and private
- Use the `explicit` keyword for single-parameter constructors to prevent implicit conversions

### Functions
- Keep functions short and focused on a single task
- Use early returns to simplify logic and reduce nesting

### Enums
- Use enum classes when possible for type safety

### Initialization
- Use member initializer lists for constructors

### Include Order
- Group includes in the following order, separated by blank lines:
  1. Related header
  2. C system headers
  3. C++ system headers
  4. Other libraries' headers
  5. Project headers

### File Structure
- Place public API in header files
- Implement non-trivial functions in source files

By following these guidelines, contributors can help maintain a consistent and high-quality codebase for SPIRV-Tools.