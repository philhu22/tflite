# Visual Studio 2022 remote build of TensorFlow Lite

**Objective:** 

Build TensorFlow Lite applications on Raspberry PI 4 devices using Visual Studio remote Linux build.

**Main references:**
- [Linux development with C++ in Visual Studio 2022](https://docs.microsoft.com/en-us/cpp/linux/?view=msvc-170)


**Content:**

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Visual Studio 2022 remote build of TensorFlow Lite](#visual-studio-2022-remote-build-of-tensorflow-lite)
  - [Prerequisites](#prerequisites)

<!-- /code_chunk_output -->

## Prerequisites
* Install the Visual Studio [Linux development with C++ workload](https://docs.microsoft.com/en-us/cpp/linux/download-install-and-setup-the-linux-development-workload?view=msvc-170#visual-studio-setup) with the option `C++ CMake tools for Linux` checked:

![vs01](/assets/images/vs01.png)

* Install the Raspberry Pi GCC Compiler Toolchains version v3.1.0 or higher on the device. See [the explanations in my notes]('./04-TF%20Lite%20C++%20on%20Raspberry%20Pi.md#Update%20the%20CC%20Cross-Compiler%20Toolchains')