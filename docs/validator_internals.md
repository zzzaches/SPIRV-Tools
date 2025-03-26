# SPIR-V Validator Internals

This document describes the internal workings of the SPIR-V validator, including how it processes different SPIR-V instructions and performs various validation checks.

## Overview

The SPIR-V validator is implemented in the SPIRV-Tools project. Its main purpose is to verify that a given SPIR-V module adheres to the SPIR-V specification and any additional validation rules.

## Key Components

- IRContext: Manages the internal representation of the SPIR-V module and provides access to various analysis results.
- CFG: Represents the control flow graph of the module.
- DefUseManager: Tracks definitions and uses of IDs in the module.
- DecorationManager: Manages decorations applied to instructions.
- TypeManager: Handles type-related information and checks.

## Validation Process

1. The validator creates an IRContext containing the SPIR-V module.
2. It performs a series of structural checks on the module layout and header.
3. For each instruction, the validator:
   - Verifies the instruction's format and operands.
   - Checks for consistency with previously seen instructions.
   - Performs instruction-specific validation rules.
4. The validator uses various analyses (e.g., CFG, def-use) to perform global checks across the module.

## Instruction Processing

The validator processes instructions by:
1. Decoding the instruction using the SPIR-V grammar.
2. Checking operand types and values against the specification.
3. Updating internal state (e.g., def-use information, type information).
4. Performing instruction-specific validation checks.

## Validation Checks

The validator performs numerous checks, including but not limited to:
- Type consistency
- Control flow correctness
- Declaration and definition rules
- Execution model constraints
- Memory model and storage class rules
- Decoration consistency

## Extending the Validator

To extend the validator for new features or instructions:

1. Update the SPIR-V grammar (if necessary) in the SPIRV-Headers project.
2. Add new validation rules in the appropriate validation pass files.
3. Implement any new analyses required for the validation rules.
4. Update the IRContext and other supporting classes as needed.
5. Add test cases to ensure the new validation rules are working correctly.

When adding new validation rules, consider:
- Where in the validation process the check should occur.
- What additional information or analyses are required.
- How to provide clear and helpful error messages for validation failures.