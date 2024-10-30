# clang-llvm-toolchain-x86_64-linux
Github Actions 用于为 x86-64 Linux 架构构建独立的 llvm 工具链，包含clang/clang++、lld、compiler-rt、libunwind、libc++和libc++abi，不依赖于 GNU 工具

<br>
##过程
构建完整工具链的流程如下：

1.环境设置
安装构建 LLVM 项目组件所需的工具（GNU 编译器集合、CMake、ninja、python）
从GitHub 版本下载并提取完整的 LLVM 项目
修补编译器 rt 以支持 x86_64 平台上的 float128 内置函数（有关更多详细信息，请参阅此问题）

2.使用 GNU 编译器集合 (gcc/g++) 配置和构建第一阶段工具链
构建分为两个阶段，以产生最终结果。第 1 阶段（依赖于 GNU）旨在提供一个可用的 clang+llvm 工具链，然后用于构建第 2 阶段（不依赖于 GNU）
第一阶段的编译器-rt、libunwind、libc++ 和 libc++abi 都是通过LLVM_ENABLE_RUNTIMES而不是LLVM_ENABLE_PROJECTS使用刚构建的第一阶段 clang 而不是 gcc/g++ 构建的。这是因为额外的配置选项（例如LIBUNWIND_USE_COMPILER_RT）将-rtlib=compiler-rt在构建运行时组件时向编译器调用添加诸如这样的标志。只有 clang 知道这些标志，而 gcc/g++ 不知道，如果使用 gcc/g++ 构建，这将导致构建失败

3.使用第一阶段工具链配置并构建第二阶段（最终）工具链
CMAKE_C_COMPILER第 1 阶段工具链通过和选项使用CMAKE_CXX_COMPILER。第 1 阶段的 clang/clang++ 已配置为通过、、和选项使用所有 llvm 工具（无 GNU 工具）CLANG_DEFAULT_UNWINDLIB=libunwind。CLANG_DEFAULT_RTLIB=compiler-rt这CLANG_DEFAULT_CXX_STDLIB=libc++将CLANG_DEFAULT_LINKER=lld导致第 2 阶段的构建不依赖于 GNU
第二阶段的编译器-rt、libunwind、libc++ 和 libc++abi 都是通过LLVM_ENABLE_PROJECTS而不是构建的，LLVM_ENABLE_RUNTIMES以便构建和包含所有必要的组件（例如crtbeginS.o编译器-rt）
