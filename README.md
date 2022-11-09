# clang-llvm-toolchain-x86_64-linux
Github Actions to build standalone llvm toolchain containing [clang/clang++](https://clang.llvm.org/), [lld](https://lld.llvm.org/), [compiler-rt](https://compiler-rt.llvm.org/), [libunwind](https://bcain-llvm.readthedocs.io/projects/libunwind/en/latest/), [libc++](https://libcxx.llvm.org/), and [libc++abi](https://libcxxabi.llvm.org/) for x86-64 Linux architechture which does not depend on GNU tooling

<br>

## Process
The process to build a complete toolchain is as follows:
1. Environment setup
    - Install tools needed for building LLVM project components ([GNU compiler collection](https://gcc.gnu.org/), [CMake](https://cmake.org/), [ninja](https://ninja-build.org/), [python](https://www.python.org/))
    - Download and extract complete LLVM project from [GitHub releases](https://github.com/llvm/llvm-project/releases)
    - Patch compiler-rt to support float128 builtins on x86_64 platforms (see [this issue](https://reviews.llvm.org/D53608) for more details)<br><br>
2. Configure and build stage 1 toolchain with GNU compiler collection (gcc/g++)
    - Building is done in 2 stages to produce the final result. Stage 1 (which depends on GNU) is meant to provide a working clang+llvm toolchain that will then be used to build stage 2 (which will NOT depend on GNU)
    - compiler-rt, libunwind, libc++, and libc++abi of stage 1 are all built via `LLVM_ENABLE_RUNTIMES` instead of `LLVM_ENABLE_PROJECTS` to have them be built with the just-built stage 1 clang instead of gcc/g++. This is because of additional configuration options (e.g. `LIBUNWIND_USE_COMPILER_RT`) which will add flags such as `-rtlib=compiler-rt` to the compiler invocation when building the runtime components. Only clang knows about such flags and gcc/g++ do not, which would cause the build to fail if they were built with gcc/g++<br><br>
3. Configure and build stage 2 (final) toolchain with stage 1 toolchain
    - The stage 1 toolchain is utilized via the `CMAKE_C_COMPILER` and `CMAKE_CXX_COMPILER` options. The clang/clang++ of stage 1 are already configured to utilize all llvm tools (and no GNU tools) via `CLANG_DEFAULT_UNWINDLIB=libunwind`, `CLANG_DEFAULT_RTLIB=compiler-rt`, `CLANG_DEFAULT_CXX_STDLIB=libc++`, and `CLANG_DEFAULT_LINKER=lld` options. This will cause stage 2 to be built without dependence on GNU
    - compiler-rt, libunwind, libc++, and libc++abi of stage 2 are all built via `LLVM_ENABLE_PROJECTS` instead of `LLVM_ENABLE_RUNTIMES` so that all necessary components are built and included (e.g. `crtbeginS.o` of compiler-rt)