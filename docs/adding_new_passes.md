# Adding New Optimization Passes to SPIRV-Tools

This guide provides step-by-step instructions for adding new optimization passes to SPIRV-Tools.

## 1. Create a new pass class

1. Create a new header file for your pass in the `source/opt/` directory, e.g., `my_new_pass.h`.
2. Define a new class that inherits from the `Pass` base class:

```cpp
#include "source/opt/pass.h"

namespace spvtools {
namespace opt {

class MyNewPass : public Pass {
 public:
  const char* name() const override { return "my-new-pass"; }
  Status Process() override;

 private:
  // Add any private members or methods needed for your pass
};

}  // namespace opt
}  // namespace spvtools
```

3. Implement the `Process()` method in a corresponding `.cpp` file.

## 2. Integrate with the pass manager

1. Add your new pass header to `source/opt/passes.h`:

```cpp
#include "source/opt/my_new_pass.h"
```

2. In your optimization pipeline, add an instance of your pass to the `PassManager`:

```cpp
#include "source/opt/pass_manager.h"
#include "source/opt/passes.h"

PassManager pass_manager;
pass_manager.AddPass<MyNewPass>();
```

## 3. Implement the pass logic

1. In your `Process()` method, use the `IRContext` to analyze and transform the SPIR-V module:

```cpp
Pass::Status MyNewPass::Process() {
  // Access the module: context()->module()
  // Access the CFG: context()->cfg()
  // Perform your optimizations here
  
  return Status::SuccessWithChange;
}
```

2. Use helper methods from the `Pass` base class and `IRContext` to simplify common tasks.

## 4. Best practices for pass implementation

1. Use `IRContext::Analysis` flags to specify which analyses your pass preserves:

```cpp
IRContext::Analysis MyNewPass::GetPreservedAnalyses() {
  return IRContext::kAnalysisInstrToBlockMapping |
         IRContext::kAnalysisCFG |
         IRContext::kAnalysisDominatorAnalysis;
}
```

2. Invalidate appropriate analyses if your pass modifies the IR:

```cpp
context()->InvalidateAnalysesExceptFor(
    IRContext::Analysis(IRContext::kAnalysisNone));
```

3. Use RAII techniques to ensure analyses are invalidated even if exceptions occur.

4. Implement thorough unit tests for your pass in the `test/opt/` directory.

5. Document your pass, including its purpose, limitations, and any assumptions it makes about the input IR.

6. Consider performance implications and optimize your pass for large SPIR-V modules.

By following these steps and best practices, you can successfully add new optimization passes to SPIRV-Tools and contribute to its optimization capabilities.