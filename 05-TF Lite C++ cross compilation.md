# Cross Compilation of TensorFlow Lite on WSL2 for Raspberry PI 4

**Objective:** 

Build TensorFlow Lite applications on WSL2 for ARMv7 architecture and particularly for Raspberry PI 4 devices.

**Main references:**
- [Cross compilation TensorFlow Lite with CMake](https://www.tensorflow.org/lite/guide/build_cmake_arm#build_for_aarch64_arm64)


**Content:**

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Cross Compilation of TensorFlow Lite on WSL2 for Raspberry PI 4](#cross-compilation-of-tensorflow-lite-on-wsl2-for-raspberry-pi-4)
- [Build for ARMv7 NEON enabled](#build-for-armv7-neon-enabled)
- [TensorFlow Lite C++ minimal example](#tensorflow-lite-c-minimal-example)
  - [Build minimal on WSL2](#build-minimal-on-wsl2)
  - [Test minimal on Raspberry PI 4](#test-minimal-on-raspberry-pi-4)

<!-- /code_chunk_output -->


# Build for ARMv7 NEON enabled
First, make sure you have an ARMv7 architecture with the following command on your Raspberry PI:
```bash
uname -a
Linux rpi1 5.10.103-v7l+ #1530 SMP Tue Mar 8 13:05:01 GMT 2022 armv7l GNU/Linux
```
Then on your Windows host, open a WSL2 terminal and follow the [TensorFlow Lite instructions](https://www.tensorflow.org/lite/guide/build_cmake_arm#build_for_armv7_neon_enabled). Here are the commands I have executed:

- Download and install the GCC ARM toolchain under ${HOME}/toolchains
    <br/>
    ```bash
    curl -LO https://storage.googleapis.com/mirror.tensorflow.org/developer.arm.com/media/Files/downloads/gnu-a/8.3-2019.03/binrel/gcc-arm-8.3-2019.03-x86_64-arm-linux-gnueabihf.tar.xz
    mkdir -p ${HOME}/toolchains
    tar xvf gcc-arm-8.3-2019.03-x86_64-arm-linux-gnueabihf.tar.xz -C ${HOME}/toolchains

    ARMCC_FLAGS="-march=armv7-a -mfpu=neon-vfpv4 -funsafe-math-optimizations -mfp16-format=ieee"
    ARMCC_PREFIX=${HOME}/toolchains/gcc-arm-8.3-2019.03-x86_64-arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
    ```
- Clone the TensorFlow repository
   <br/>
    ```bash   
    cd ~
    git clone https://github.com/tensorflow/tensorflow.git tensorflow_src
    ```
- Run CMake tool in the build directory
  <br/>
  ```bash
  cd tensorflow
  mkdir tflite_build_arm
  cd tflite_build_arm

  cmake -DCMAKE_C_COMPILER=${ARMCC_PREFIX}gcc \
  -DCMAKE_CXX_COMPILER=${ARMCC_PREFIX}g++ \
  -DCMAKE_C_FLAGS="${ARMCC_FLAGS}" \
  -DCMAKE_CXX_FLAGS="${ARMCC_FLAGS}" \
  -DCMAKE_VERBOSE_MAKEFILE:BOOL=ON \
  -DCMAKE_SYSTEM_NAME=Linux \
  -DCMAKE_SYSTEM_PROCESSOR=armv7 \
  ../tensorflow_src/tensorflow/lite
  ```

- Build 
  <br/>
    ```bash   
    cmake --build . -j
    ```
    This creates `libtensorflow-lite.a` in the current directory `tflite_build`:
    <br/>
    ```bash
    ...
    [100%] Built target tensorflow-lite
    make[1]: Leaving directory '/home/toto/tensorflow/tflite_build_arm'
    /usr/bin/cmake -E cmake_progress_start /home/toto/tensorflow/tflite_build_arm/CMakeFiles 0
  
    toto:tflite_build_arm$ ls -l libtensor*
    -rw-r--r-- 1 toto toto 8060544 Apr  1 04:03 libtensorflow-lite.a
    ```

# TensorFlow Lite C++ minimal example
Let's try now to build and run the [TensorFlow Lite C++ minimal example](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/examples/minimal). 
## Build minimal on WSL2
First, we build  without XNNPACK which is enabled by default.

```bash
cd ~/tensorflow

# create CMake build directory
mkdir minimal_build_arm_no_xnnpack
cd minimal_build_arm_no_xnnpack

# run CMake tool
 cmake -DCMAKE_C_COMPILER=${ARMCC_PREFIX}gcc \
  -DCMAKE_CXX_COMPILER=${ARMCC_PREFIX}g++ \
  -DCMAKE_C_FLAGS="${ARMCC_FLAGS}" \
  -DCMAKE_CXX_FLAGS="${ARMCC_FLAGS}" \
  -DCMAKE_VERBOSE_MAKEFILE:BOOL=ON \
  -DCMAKE_SYSTEM_NAME=Linux \
  -DCMAKE_SYSTEM_PROCESSOR=armv7 \
  -DTFLITE_ENABLE_XNNPACK=OFF \
  ../tensorflow_src/tensorflow/lite/examples/minimal

# build
cmake --build . -j
```

After a successful build, copy the application `minimal` to the target Raspberry PI device. 

## Test minimal on Raspberry PI 4
Try the minimal application with
```bash
./minimal mymodel.tflite
```
You can download a TF Lite model from the [TensorFlow Hub](https://tfhub.dev/s?deployment-format=lite). For this example, I used the [EfficientDet-Lite0 Object detection model](https://tfhub.dev/tensorflow/lite-model/efficientdet/lite0/detection/default/1):

```bash
./minimal efficientdet_lite0.tflite

toto@rpi1:~/tflite/sandbox $ ./minimal efficientdet_lite0.tflite                                                     
=== Pre-invoke Interpreter State ===                                                                                 
Interpreter has 1 subgraphs.                                                                                         
                                                                                                                     
-----------Subgraph-0 has 605 tensors and 267 nodes------------                                                      
1 Inputs: [0] -> 307200B (0.29MB)                                                                                    
4 Outputs: [598-601] -> 604B (0.00MB)                                                                                
                                                                                                                     
Tensor  ID Name                      Type            AllocType          Size (Bytes/MB)    Shape      MemAddr-Offset 
Tensor   0 serving_default_images:0  kTfLiteUInt8    kTfLiteArenaRw     307200   / 0.29 [1,320,320,3] [0, 307200)    
Tensor   1 fpn_cells/cell_2/fnode... kTfLiteInt32    kTfLiteMmapRo      8        / 0.00 [2] [3730368, 3730376)       
Tensor   2 fpn_cells/cell_2/fnode... kTfLiteInt32    kTfLiteMmapRo      8        / 0.00 [2] [3730348, 3730356)       
Tensor   3 fpn_cells/cell_2/fnode... kTfLiteInt32    kTfLiteMmapRo      8        / 0.00 [2] [3730328, 3730336)       

```

It works. We are done with this first cross-compilation on WSL2 for a Raspberry PI device.