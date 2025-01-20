# LLVM-FunctionPass-for-Instrumentation
To write your first LLVM FunctionPass for instrumentation, you'll need to interact with the LLVM Pass infrastructure, particularly to modify the Intermediate Representation (IR) of a program during compilation.
What is a FunctionPass in LLVM?

A FunctionPass operates on individual functions within the LLVM intermediate representation (IR). It is used to analyze or transform functions in some way, such as adding instrumentation for profiling, debugging, or performance analysis.
Basic Steps:

    Create a Pass: Create a custom pass that implements the FunctionPass interface.
    Insert Instrumentation Code: Add custom instructions or transformations to the functions.
    Register the Pass: Register the pass with LLVM.
    Use the Pass: Compile the code and apply the pass using opt or through your toolchain.

Example Code: A Simple Instrumentation Pass

This pass will insert an LLVM instruction at the entry of each function to print the name of the function (a simple form of instrumentation).
Step 1: Setting Up Your LLVM Pass Environment

Before writing the pass, make sure you have LLVM set up properly. You need to install LLVM and set up the build environment.
Step 2: Code for the FunctionPass

#include "llvm/IR/Function.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/Pass.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/Module.h"

using namespace llvm;

namespace {
  // Define your custom pass
  struct FunctionInstrumentation : public FunctionPass {
    static char ID; // Pass identification, replacement for typeid
    FunctionInstrumentation() : FunctionPass(ID) {}

    bool runOnFunction(Function &F) override {
      // We need to instrument the function entry point
      // Get the first basic block of the function
      BasicBlock &entry = F.getEntryBlock();

      // Create an IRBuilder to add instructions at the beginning of the function
      IRBuilder<> builder(&entry, entry.begin());

      // Create a string constant to represent the function's name
      Value *funcName = builder.CreateGlobalStringPtr(F.getName());

      // Assuming we want to call a simple function that prints the name
      Function *printFunc = Intrinsic::getDeclaration(F.getParent(), Intrinsic::donothing);
      
      // Create the print call (or you can create a custom function call)
      builder.CreateCall(printFunc, {funcName});

      // Return false since we haven't modified the function body
      return false;
    }
  };
}

// Register the pass with LLVM
char FunctionInstrumentation::ID = 0;
static RegisterPass<FunctionInstrumentation> X("function-instrumentation", "Function Instrumentation Pass", false, false);

Explanation:

    FunctionPass: This is the base class for a function-level pass. Your pass will extend this class.
    runOnFunction: This is the main function of the pass. It is called for each function in the LLVM IR.
    IRBuilder: We use IRBuilder<> to insert instructions at the beginning of the function.
    Global String: We create a global string containing the function name (F.getName()).
    Create Call: We insert a call to a function that prints the function's name (in this case, we simulate it with donothing).

Step 3: Build and Use the Pass

    Build the Pass: You'll need to compile this into a shared library (DLL on Windows or .so on Linux). Assuming you have the necessary LLVM development environment:

clang++ -shared -fPIC -o libFunctionInstrumentation.so FunctionInstrumentation.cpp `llvm-config --cxxflags --ldflags --libs`

Use the Pass with opt: Once the pass is compiled, you can apply it to an LLVM IR file using the opt tool:

    opt -load ./libFunctionInstrumentation.so -function-instrumentation input.bc -o output.bc

    This command loads your pass (libFunctionInstrumentation.so) and applies it to the input LLVM bytecode (input.bc), generating a new LLVM bytecode file (output.bc).

Step 4: Debugging and Testing the Pass

To test the pass:

    First, generate the LLVM IR code from a C/C++ program using clang:

clang -S -emit-llvm -o input.ll input.c

Then, apply your instrumentation pass to the LLVM IR file:

    opt -load ./libFunctionInstrumentation.so -function-instrumentation input.ll -o output.ll

    Finally, you can check the modified LLVM IR in output.ll or compile it back to executable using clang.

Step 5: Customize Instrumentation

The above example just inserts a string representing the function name. However, you can extend this pass to:

    Add profiling counters.
    Log more sophisticated runtime information (e.g., memory accesses, function calls).
    Insert conditional instrumentation based on function characteristics.

Conclusion:

This simple LLVM FunctionPass demonstrates how you can use LLVM's infrastructure to perform basic instrumentation, such as printing the name of each function. This concept can be extended to more complex tasks like profiling, debugging, or analyzing performance. The key steps include working with BasicBlock, IRBuilder, and Function classes in LLVM.
