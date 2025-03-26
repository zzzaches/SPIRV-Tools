# Adding New Optimization Passes to SPIRV-Tools

This guide provides step-by-step instructions on how to add new optimization passes to SPIRV-Tools.

## Steps

1. Create a new header file for your pass in the `source/opt/` directory. Name it `your_pass_name.h`.

2. In the header file, define a new class that inherits from the `Pass` base class:

```cpp
#include "source/opt/pass.h"

namespace spvtools {
namespace opt {

class YourPassName : public Pass {
 public:
  const char* name() const override { return "your-pass-name"; }
  Status Process() override;

 private:
  // Add any private members or helper methods
};

}  // namespace opt
}  // namespace spvtools
```

3. Create a corresponding `.cpp` file in the same directory to implement your pass.

4. Implement the `Process()` method in your `.cpp` file. This is where the main logic of your pass goes:

```cpp
Pass::Status YourPassName::Process() {
  // Implement your pass logic here
  // Return Status::SuccessWithChange if the module was modified
  // Return Status::SuccessWithoutChange if no changes were made
  // Return Status::Failure if an error occurred
}
```

5. Add your new pass header to `source/opt/passes.h`:

```cpp
#include "source/opt/your_pass_name.h"
```

6. Integrate your pass with the PassManager by adding it to the appropriate optimization level in `source/opt/optimizer.cpp`:

```cpp
Optimizer& Optimizer::RegisterPerformancePasses() {
  // Add your pass to the desired optimization level
  RegisterPass(CreateYourPassName());
  return *this;
}
```

7. Implement the pass creation function in your `.cpp` file:

```cpp
Pass* CreateYourPassName() { return new YourPassName(); }
```

8. Add a command-line option for your pass in `tools/opt/opt.cpp`:

```cpp
if (0 == strcmp(cur_arg, "--your-pass-name")) {
  optimizer.RegisterPass(CreateYourPassName());
}
```

## Best Practices

1. Use the IRContext to access and modify the SPIR-V module.

2. Implement the `GetPreservedAnalyses()` method to indicate which analyses your pass preserves.

3. Use SPIRV-Tools utility functions and existing passes when possible to avoid duplicating code.

4. Write unit tests for your pass in the `test/opt/` directory.

5. Document your pass thoroughly, including its purpose, limitations, and any assumptions it makes.

6. Ensure your pass handles all possible SPIR-V instructions and edge cases.

7. Optimize for both correctness and performance.

8. Follow the existing code style and naming conventions in SPIRV-Tools.

By following these steps and best practices, you can successfully add new optimization passes to SPIRV-Tools and contribute to its ongoing development.