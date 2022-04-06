# Visual Studio 2022 remote build of TensorFlow Lite

**Objective:** 

Build TensorFlow Lite applications on Raspberry PI 4 devices using Visual Studio remote Linux build.

**Main references:**
- [Linux development with C++ in Visual Studio 2022](https://docs.microsoft.com/en-us/cpp/linux/?view=msvc-170)
- [CMake projects in Visual Studio](https://docs.microsoft.com/en-us/cpp/build/cmake-projects-in-visual-studio?view=msvc-170)


**Content:**

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Visual Studio 2022 remote build of TensorFlow Lite](#visual-studio-2022-remote-build-of-tensorflow-lite)
  - [Prerequisites](#prerequisites)
  - [Visual Studio remote build](#visual-studio-remote-build)
    - [Create CMake project](#create-cmake-project)
    - [Add a remote device](#add-a-remote-device)
    - [CMake Configuration](#cmake-configuration)
    - [Toolchains CMake file](#toolchains-cmake-file)
    - [Build](#build)

<!-- /code_chunk_output -->

## Prerequisites
* Install the Visual Studio [Linux development with C++ workload](https://docs.microsoft.com/en-us/cpp/linux/download-install-and-setup-the-linux-development-workload?view=msvc-170#visual-studio-setup) with the option `C++ CMake tools for Linux` checked:

![vs01](/assets/images/vs01.png)

* Install on the device the [Raspberry Pi GCC Native Compiler Toolchains version v3.1.0](https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/Raspberry%20Pi%20GCC%20Native-Compiler%20Toolchains/Bullseye/) or higher ([why](./04-TF%20Lite%20C++%20on%20Raspberry%20Pi.md#Update%20the%20CC%20Cross-Compiler%20Toolchains)). Here are the instructions to [install permanently](https://github.com/abhiTronix/raspberry-pi-cross-compilers/wiki/Native-Compiler:-Installation-Instructions#c2--permanent-installation) (recommended) the toolchain v 3.1.0:
<br/>
    ```bash
    mkdir gcctoolchains
    cd gcctoolchains
    ```
    
    ```bash
    wget "https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/Raspberry Pi GCC Native-Compiler Toolchains/Stretch/GCC 10.3.0/Raspberry Pi 3A+, 3B+, 4/native-gcc-10.3.0-pi_3+.tar.gz"

    tar xf native-gcc-10.3.0-pi_3+.tar.gz
    rm native-gcc-10.3.0-pi_3+.tar.gz

    sudo mv native-pi-gcc-10.3.0-2 /opt

    echo 'export PATH=/opt/native-pi-gcc-10.3.0-2/bin:$PATH' >> .profile  
    echo 'export LD_LIBRARY_PATH=/opt/native-pi-gcc-10.3.0-2/lib:$LD_LIBRARY_PATH' >> .profile
    source .profile

    sudo ln -sf /usr/include/arm-linux-gnueabihf/asm/* /usr/include/asm
    sudo ln -sf /usr/include/arm-linux-gnueabihf/gnu/* /usr/include/gnu
    sudo ln -sf /usr/include/arm-linux-gnueabihf/bits/* /usr/include/bits
    sudo ln -sf /usr/include/arm-linux-gnueabihf/sys/* /usr/include/sys
    sudo ln -sf /usr/include/arm-linux-gnueabihf/openssl/* /usr/include/openssl
    sudo ln -sf /usr/lib/arm-linux-gnueabihf/crtn.o /usr/lib/crtn.o
    sudo ln -sf /usr/lib/arm-linux-gnueabihf/crt1.o /usr/lib/crt1.o
    sudo ln -sf /usr/lib/arm-linux-gnueabihf/crti.o /usr/lib/crti.o
    ```
  
* Clone the TensorFlow repository on the Raspberry device (we do not want to copy theses files from the Windows host to the Raspberry device):
  <br/>
    ```bash
    cd ~
    mkdir tensorflow
    cd tensorflow
    git clone https://github.com/tensorflow/tensorflow.git tensorflow_src
    ```
 
## Visual Studio remote build
### Create CMake project
On your Visual Studio development host, prepare a new folder `minimal` with 2 files copied from the TensorFlow Lite [example repository](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/examples/minimal):
- [minimal.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/examples/minimal/minimal.cc)
- [CMakeLists.txt](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/examples/minimal/CMakeLists.txt)

Open this folder in Visual Studio. When you open a folder that contains a file `CMakeLists.txt`, Visual Studio understands it is a CMake project and it will create files to let you configure the CMake build and debugging (more information on [Microsoft Docs](https://docs.microsoft.com/en-us/cpp/linux/cmake-linux-configure?view=msvc-170))


Edit CMakeLists.txt to make the following changes:

1. Specify the path to the TensorFlow source on the Raspberry device:
<br/>
    ```cmake
    set(TENSORFLOW_SOURCE_DIR "/home/toto/tensorflow/tensorflow_src" CACHE PATH
    "Directory that contains the TensorFlow project")
    ```

2. Add the following compile definitions to avoid the gcc [compilation errors](./04-TF%20Lite%20C++%20on%20Raspberry%20Pi.md#Step%203.%20Fix%20includes)
<br/>
    ```cmake
    set_directory_properties(PROPERTIES  
        COMPILE_DEFINITIONS "SSIZE_MAX=LONG_MAX;PATH_MAX=4096" )
    ```

Here is the complete list of instructions:
```cmake
# Builds the minimal Tensorflow Lite example.

cmake_minimum_required(VERSION 3.16)
project(minimal C CXX)

set(TENSORFLOW_SOURCE_DIR "/home/toto/tensorflow/tensorflow_src" CACHE PATH
  "Directory that contains the TensorFlow project"
)

# $ grep PATH_MAX /usr/include/linux/limits.h
# define PATH_MAX        4096    /* # chars in a path name including nul */
set_directory_properties(PROPERTIES  
    COMPILE_DEFINITIONS "SSIZE_MAX=LONG_MAX;PATH_MAX=4096" 
)

add_subdirectory( 
  "${TENSORFLOW_SOURCE_DIR}/tensorflow/lite"
  "${CMAKE_CURRENT_BINARY_DIR}/tensorflow-lite"
  EXCLUDE_FROM_ALL
)

set(CMAKE_CXX_STANDARD 11)

add_executable(minimal
  minimal.cc
)
target_link_libraries(minimal
  tensorflow-lite
)
```
### Add a remote device
In Visual Studio, add your Raspberry with its ssh credentials using `Tools\Options\Cross Platform` ([more]())


### CMake Configuration
Create a new CMake configuration `Raspberry-GCC-Debug` of type `Linux-GCC-Debug`:
![vs02](/assets/images/vs02.png)

Edit the settings with the following changes:

1. **Toolset** is `linux-arm`
2. Specify your **Remote Machine Name**
3. Set two new variables:
   1.  `CMAKE_TOOLCHAIN_FILE` to specify the location of the toolchains cmakefile (see below).
   2. `TFLITE_ENABLE_XNNPACK=OFF` to disable XNNPACK that is enabled by default

    ![vs03](/assets/images/vs03.png)
4. Optionally, modify the remote src/build/install folders

Here is my configuration:

```json
    {
      "name": "Raspberry-GCC-Debug",
      "generator": "Ninja",
      "configurationType": "Debug",
      "cmakeExecutable": "cmake",
      "remoteCopySourcesExclusionList": [ ".vs", ".git", "out" ],
      "cmakeCommandArgs": "",
      "buildCommandArgs": "",
      "ctestCommandArgs": "",
      "inheritEnvironments": [ "linux_arm" ],
      "remoteMachineName": "-1225463274;xxx.xxx.xxx.xxx (username=yyyy, port=22, authentication=Password)",
      "remoteCMakeListsRoot": "$HOME/tensorflow/${projectDirName}/src",
      "remoteBuildRoot": "$HOME/tensorflow/${projectDirName}/build",
      "remoteInstallRoot": "$HOME/tensorflow/${projectDirName}/install",
      "remoteCopySources": true,
      "rsyncCommandArgs": "-t --delete",
      "remoteCopyBuildOutput": false,
      "remoteCopySourcesMethod": "rsync",
      "variables": [
        {
          "name": "CMAKE_TOOLCHAIN_FILE",
          "value": "$HOME/tensorflow/toolchains.cmake",
          "type": "FILEPATH"
        },
        {
          "name": "TFLITE_ENABLE_XNNPACK",
          "value": "OFF",
          "type": "STRING"
        }
      ]
    }
```

### Toolchains CMake file
A Toolchains CMake file is needed to direct the gcc compiler to the proper GCC Native Compiler Toolchains (see [Prerequisites](#prerequisites)). The file is specified in the CMake configuration (variable `CMAKE_TOOLCHAIN_FILE`) and is located on the remote device. Here is its content:
```cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR armv7)

set(tools /opt/native-pi-gcc-10.3.0-2)
set(CMAKE_C_COMPILER ${tools}/bin/gcc-10.3.0)
set(CMAKE_CXX_COMPILER ${tools}/bin/g++-10.3.0)
set(CMAKE_CPP_COMPILER ${tools}/bin/cpp-10.3.0)
set(CMAKE_FC_COMPILER ${tools}/bin/gfortran-10.3.0)
set(CMAKE_AR_COMPILER ${tools}/bin/gcc-ar-10.3.0)
set(CMAKE_RANLIB ${tools}/bin/gcc-ranlib-10.3.0)
set(CMAKE_CPP_COMPILER ${tools}/bin/cpp-10.3.0)

set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```

### Build
Now, you are ready to generate the CMake build files and build them:



 
 
