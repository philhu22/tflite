# Visual Studio 2022 remote build and debug of TFLite applications

**Objective:** 

Build TensorFlow Lite applications on Raspberry PI 4 devices using Visual Studio 2022 Remote Linux build.

**Main references:**
- [Linux development with C++ in Visual Studio 2022](https://docs.microsoft.com/en-us/cpp/linux/?view=msvc-170)
- [CMake projects in Visual Studio](https://docs.microsoft.com/en-us/cpp/build/cmake-projects-in-visual-studio?view=msvc-170)
- [TensorFlow Lite C++ minimal example](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/examples/minimal)
- [TensorFlow Lite C++ label_image example](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/examples/label_image)


**Content:**

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Visual Studio 2022 remote build and debug of TFLite applications](#visual-studio-2022-remote-build-and-debug-of-tflite-applications)
  - [Prerequisites](#prerequisites)
    - [Visual Studio Linux development](#visual-studio-linux-development)
    - [GCC Toolchains](#gcc-toolchains)
    - [Clone TensorFlow](#clone-tensorflow)
  - [Build & debug the minimal example](#build-debug-the-minimal-example)
    - [Create CMake project](#create-cmake-project)
    - [Add a remote device](#add-a-remote-device)
    - [CMake Configuration](#cmake-configuration)
    - [Toolchains CMake file](#toolchains-cmake-file)
    - [Build, Run, Debug](#build-run-debug)
  - [Build & debug the label_image example](#build-debug-the-label_image-example)
  - [Enable XNNPACK](#enable-xnnpack)

<!-- /code_chunk_output -->

## Prerequisites
### Visual Studio Linux development
  Install the Visual Studio [Linux development with C++ workload](https://docs.microsoft.com/en-us/cpp/linux/download-install-and-setup-the-linux-development-workload?view=msvc-170#visual-studio-setup) with the option `C++ CMake tools for Linux` checked:

![vs01](/assets/images/vs01.png)

### GCC Toolchains
Install on the device the [Raspberry Pi GCC Native Compiler Toolchains version v3.1.0](https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/Raspberry%20Pi%20GCC%20Native-Compiler%20Toolchains/Bullseye/) or higher ([why](https://github.com/philhu22/tflite/blob/main/04-TF%20Lite%20C++%20on%20Raspberry%20Pi.md#update-the-cc-native-compiler-toolchains))). Here are the instructions to [install permanently](https://github.com/abhiTronix/raspberry-pi-cross-compilers/wiki/Native-Compiler:-Installation-Instructions#c2--permanent-installation) (recommended) the toolchain v 3.1.0:
  ```bash
  mkdir gcctoolchains
  cd gcctoolchains
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

### Clone TensorFlow
Clone the TensorFlow repository on the Raspberry device (we do not want to copy theses files from the Windows host to the Raspberry device):
  ```bash
  cd ~
  mkdir tensorflow
  cd tensorflow
  git clone https://github.com/tensorflow/tensorflow.git tensorflow_src
  ```
 
## Build & debug the minimal example
### Create CMake project
On your Visual Studio development host, prepare a new folder `minimal` with 2 files copied from the TensorFlow Lite [example repository](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/examples/minimal):
- [minimal.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/examples/minimal/minimal.cc)
- [CMakeLists.txt](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/examples/minimal/CMakeLists.txt)

Open this folder in Visual Studio. When you open a folder that contains a file `CMakeLists.txt`, Visual Studio understands it is a CMake project and it will create files to let you configure the CMake build and debugging (more information on [Microsoft Docs](https://docs.microsoft.com/en-us/cpp/linux/cmake-linux-configure?view=msvc-170))

Edit CMakeLists.txt to make the following changes:

1. Specify the path to the TensorFlow source on the Raspberry device:
    ```cmake
    set(TENSORFLOW_SOURCE_DIR "/home/toto/tensorflow/tensorflow_src" CACHE PATH
    "Directory that contains the TensorFlow project")
    ```

2. Add the following compile definitions to avoid the gcc [compilation errors](https://github.com/philhu22/tflite/blob/main/04-TF%20Lite%20C++%20on%20Raspberry%20Pi.md#step-3-fix-includes)
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
In Visual Studio, add your Raspberry with its ssh credentials using `Tools\Options\Cross Platform` ([more](https://docs.microsoft.com/en-us/cpp/linux/connect-to-your-remote-linux-computer?view=msvc-170))


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

### Build, Run, Debug
Now, you are ready to generate the CMake build files with the command `Project\Delete Cache and Reconfigure`:

```bash
1> Copying files to the remote machine.
1> Starting copying files to remote machine.
1> Finished copying files (elapsed time 00h:00m:00s:369ms).
1> CMake generation started for configuration: 'Raspberry-GCC-Debug'.
1> Found cmake executable at /usr/bin/cmake.
1> /usr/bin/cmake -G "Ninja"   -DCMAKE_BUILD_TYPE:STRING="Debug" -DCMAKE_INSTALL_PREFIX:PATH="$HOME/tensorflow/minimal/install" -DCMAKE_TOOLCHAIN_FILE:FILEPATH="$HOME/tensorflow/toolchains.cmake" -DTFLITE_ENABLE_XNNPACK:STRING="OFF"  /home/toto/tensorflow/minimal/src/CMakeLists.txt;
...
1> Extracted CMake variables.
1> Extracted source files and headers.
1> Extracted code model.
1> Extracted includes paths.
1> CMake generation finished.
```

Next, build with the command `Build\Build All`.

When the build is done, select the target `minimal` and press F5 to start debugging in Visual Studio:

![vs04](/assets/images/vs04.png)


## Build & debug the label_image example

Let's now repeat the steps explained above to build and debug the [TF Lite label_image example](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/examples/label_image). 

1. Prepare a new CMake project `label_image` with the files copied from the TF Lite example repository.
2. Edit CMakeLists.txt and copy these instructions:
   
    ```cmake
    cmake_minimum_required(VERSION 3.16)
    project(label_image C CXX)

    set(TF_SOURCE_DIR "/home/toto/tensorflow/tensorflow_src/tensorflow" CACHE PATH
      "Directory that contains the TensorFlow project"
    )
    set(TFLITE_SOURCE_DIR "${TF_SOURCE_DIR}/lite" CACHE PATH
      "Directory that contains the TensorFlow project"
    )

    # $ grep PATH_MAX /usr/include/linux/limits.h
    # define PATH_MAX        4096    /* # chars in a path name including nul */
    set_directory_properties(PROPERTIES  
        COMPILE_DEFINITIONS "SSIZE_MAX=LONG_MAX;PATH_MAX=4096" 
    )

    add_subdirectory( 
      "${TFLITE_SOURCE_DIR}"
      "${CMAKE_CURRENT_BINARY_DIR}/tensorflow-lite"
      EXCLUDE_FROM_ALL
    )

    set(CMAKE_CXX_STANDARD 11)

    set(TFLITE_LABEL_IMAGE_SRCS
      bitmap_helpers.cc
      label_image.cc
    )
    list(APPEND TFLITE_LABEL_IMAGE_SRCS
      ${TF_SOURCE_DIR}/core/util/stats_calculator.cc
      ${TFLITE_SOURCE_DIR}/profiling/memory_info.cc
      ${TFLITE_SOURCE_DIR}/profiling/profile_summarizer.cc
      ${TFLITE_SOURCE_DIR}/profiling/profile_summary_formatter.cc
      ${TFLITE_SOURCE_DIR}/profiling/time.cc
      ${TFLITE_SOURCE_DIR}/tools/command_line_flags.cc
      ${TFLITE_SOURCE_DIR}/tools/delegates/default_execution_provider.cc
      ${TFLITE_SOURCE_DIR}/tools/delegates/delegate_provider.cc
      ${TFLITE_SOURCE_DIR}/tools/evaluation/utils.cc
      ${TFLITE_SOURCE_DIR}/tools/tool_params.cc
    )

    if(TFLITE_ENABLE_XNNPACK)
      list(APPEND TFLITE_LABEL_IMAGE_SRCS
        ${TFLITE_SOURCE_DIR}/tools/delegates/xnnpack_delegate_provider.cc
      )
    else()
      set(TFLITE_LABEL_IMAGE_CC_OPTIONS "-DTFLITE_WITHOUT_XNNPACK")
    endif()  # TFLITE_ENABLE_XNNPACK


    add_executable(labelimage
      ${TFLITE_LABEL_IMAGE_SRCS}
    )

    target_compile_options(labelimage
      PRIVATE
        ${TFLITE_LABEL_IMAGE_CC_OPTIONS}
    )

    target_link_libraries(labelimage
      tensorflow-lite
    )
    ```

3. Copy the [Cmake configuration](#cmake-configuration) used for the minimal example.
4. On the Raspberry device, get a copy of the model and the labels:  
  
    ```bash
    mkdir -p ~/tensorflow/data/label_image/models/mobilenet
    cd ~/tensorflow/data/label_image/models/mobilenet

    wget https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_2018_02_22/mobilenet_v1_1.0_224.tgz
    tar xf mobilenet_v1_1.0_224.tgz
    rm mobilenet_v1_1.0_224.tgz

    mkdir labels & cd labels
    wget https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_1.0_224_frozen.tgz
    tar xf mobilenet_v1_1.0_224_frozen.tgz
    rm mobilenet_v1_1.0_224_frozen.tgz
    ```
  
    and copy the sample image `testdata\grace_hopper.bmp` to `~/tensorflow/data/label_image/images`

5. In Visual Studio, invoke `Debug\Debug and Launch Settings`  to include the command-line arguments passed on startup:

   ```json

    "args": [
      "--tflite_model",
      "/home/toto/tensorflow/data/label_image/models/mobilenet/mobilenet_v1_1.0_224.tflite",
      "--labels",
      "/home/toto/tensorflow/data/label_image/models/mobilenet/labels/mobilenet_v1_1.0_224/labels.txt",
      "--image",
      "/home/toto/tensorflow/data/label_image/images/grace_hopper.bmp"
      ],
      "env": {}
   ```

6. It's now time to generate the CMake build files with the command 'Project\Delete Cache and Reconfigure', and next build the application with `Build All`. 
7. When the build is done, select the target `labelimage`, add a breakpoint at the return statement in `label_image.cc` and press F5 to start debugging in Visual Studio. You should see results like this:

    ![vs05](/assets/images/vs05.png)

## Enable XNNPACK

 The [XNNPACK](https://github.com/google/XNNPACK) library is now [integrated](https://blog.tensorflow.org/2020/07/accelerating-tensorflow-lite-xnnpack-integration.html) into TensorFlow Lite and important performance improvement can be achieved by enabling the XNNPACK delegate. To enable XNNPACK, set `TFLITE_ENABLE_XNNPACK` to `ON` in CMakeSettings.json and regenerate the build files. Then `Rebuild All`. 
 
 Launch the target `labelimage`. You should see that XNNPACK is used and that it improves significantly the inference performance:
 
![vs06](/assets/images/vs06.png)
 
