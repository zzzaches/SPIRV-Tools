# SPIRV-Tools Testing Framework

SPIRV-Tools uses a custom testing framework for writing and running tests. This document explains how to write and run tests for new optimization passes and validator rules.

## Test Types

The project uses several types of tests:

1. Unit tests - Test individual components/functions
2. End-to-end tests - Test full optimization passes or validation flows
3. Fuzz tests - Use randomly generated inputs to test for crashes/errors

## Writing Tests

### Unit Tests 

Unit tests are written using the Google Test framework. They are located in the `test/` directory and follow the naming convention `*_test.cpp`.

Example of a unit test:

```cpp
TEST(PassManagerTest, AddPassAddsToVector) {
  PassManager manager;
  auto pass = std::make_unique<MockPass>();
  manager.AddPass(std::move(pass));
  EXPECT_EQ(manager.NumPasses(), 1);
}
```

### End-to-End Tests

End-to-end tests for optimization passes are written as assembly language input/output pairs. These are located in `test/opt/` and use the `.spvasm` extension.

Example of an end-to-end test:

```
; RUN: spirv-opt %s -O --skip-validation | FileCheck %s
               OpCapability Shader
          %1 = OpExtInstImport "GLSL.std.450"
               OpMemoryModel Logical GLSL450
               OpEntryPoint Fragment %4 "main"
               OpExecutionMode %4 OriginUpperLeft
               OpSource ESSL 310
          %2 = OpTypeVoid
          %3 = OpTypeFunction %2
          %4 = OpFunction %2 None %3
          %5 = OpLabel
               OpReturn
               OpFunctionEnd

; CHECK: OpCapability Shader
; CHECK-NEXT: [[import:%\w+]] = OpExtInstImport "GLSL.std.450"
; CHECK-NEXT: OpMemoryModel Logical GLSL450
; CHECK-NEXT: OpEntryPoint Fragment [[main:%\w+]] "main"
; CHECK-NEXT: OpExecutionMode [[main]] OriginUpperLeft
; CHECK-NEXT: OpSource ESSL 310
; CHECK-NEXT: [[void:%\w+]] = OpTypeVoid
; CHECK-NEXT: [[functy:%\w+]] = OpTypeFunction [[void]]
; CHECK-NEXT: [[main]] = OpFunction [[void]] None [[functy]]
; CHECK-NEXT: [[entry:%\w+]] = OpLabel
; CHECK-NEXT: OpReturn
; CHECK-NEXT: OpFunctionEnd
```

### Fuzz Tests

Fuzz tests use the libFuzzer framework to generate random inputs. These are located in `test/fuzz/` and typically have a `*_fuzzer.cpp` naming convention.

## Running Tests

Tests can be run using CMake's ctest command:

```
cd <build-dir>
ctest
```

To run a specific test:

```
ctest -R <test-name>
```

For example, to run all optimizer tests:

```
ctest -R "Opt*"
```

## Adding Tests for New Passes

When adding a new optimization pass:

1. Create unit tests in `test/opt/pass_<name>_test.cpp`
2. Add end-to-end tests in `test/opt/pass_<name>.spvasm`
3. Update `test/opt/CMakeLists.txt` to include the new test files

## Adding Tests for New Validator Rules

When adding new validation rules:

1. Create unit tests in `test/val/<rule>_test.cpp`
2. Add end-to-end tests in `test/val/<rule>.spvasm`
3. Update `test/val/CMakeLists.txt` to include the new test files