# SPIR-V Validator Internals

This document describes the internal workings of the SPIR-V validator, including how it processes different SPIR-V instructions and performs various validation checks.

## Overview

The SPIR-V validator is responsible for verifying that a SPIR-V module adheres to the rules and constraints defined in the SPIR-V specification. It performs a series of checks on the structure, instructions, and semantics of the SPIR-V code.

## Key Components

- IRContext: Manages the in-memory representation of the SPIR-V module and provides access to various analyses.
- InstructionFolder: Performs constant folding and propagation.
- FeatureManager: Tracks capabilities and extensions used in the module.
- Various analysis passes (e.g. DefUseManager, TypeManager, DecorationManager)

## Validation Process

1. Parse the SPIR-V binary into an in-memory representation.
2. Perform structural validation checks (e.g. section ordering, instruction layout).
3. Build and populate the IRContext.
4. Validate individual instructions:
   - Check operand types and counts
   - Verify IDs are defined before use
   - Ensure instructions are valid for the current shader stage and execution model
5. Perform global analyses and checks:
   - Control flow analysis
   - Data flow analysis
   - Type checking
   - Decoration consistency

## Instruction Processing

For each instruction, the validator:

1. Decodes the instruction opcode and operands.
2. Verifies the instruction is allowed in the current context (e.g. function vs. global scope).
3. Checks operand types against the instruction's requirements.
4. Performs instruction-specific validation logic.
5. Updates relevant analyses (e.g. def-use chains, type information).

## Extending the Validator

To extend the validator for new features or instructions:

1. Update the SPIR-V grammar (spirv.core.grammar.json) to include new instructions or operands.
2. Add new capability and extension handling in FeatureManager.
3. Implement instruction-specific validation logic in the appropriate validation functions.
4. Update existing analyses and passes to handle new instructions or semantics.
5. Add new validation passes if required for complex feature interactions.
6. Extend test suite to cover new validation scenarios.

## Configuration Options

The validator behavior can be customized using the spv_validator_options_t structure, which allows relaxing certain rules or enabling experimental features.