# Native C++ Tensorflow Lite build on Raspberry Pi 4

**Objective:** 

Make a native C++ build of TensorFlow Lite on Raspberry Pi 4 and run the TensorFlow Lite minimal application (without XNNPACK).

**Main references:**
- [Build TensorFlow Lite with CMake](https://www.tensorflow.org/lite/guide/build_cmake#create_a_cmake_project_which_uses_tensorflow_lite)
- [Raspberry Pi GCC Toolchains Files](https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/)

**Content:**

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Native C++ Tensorflow Lite build on Raspberry Pi 4](#native-c-tensorflow-lite-build-on-raspberry-pi-4)
  - [Pre-installations steps](#pre-installations-steps)
    - [Update the environment](#update-the-environment)
    - [Update the CC Cross-Compiler Toolchains](#update-the-cc-cross-compiler-toolchains)
  - [Build TensorFlow Lite with CMake](#build-tensorflow-lite-with-cmake)
    - [Step 1. Clone TensorFlow repository](#step-1-clone-tensorflow-repository)
    - [Step 2. Run CMake tool](#step-2-run-cmake-tool)
    - [Step 3. Fix includes](#step-3-fix-includes)
    - [Step 4. Build](#step-4-build)
  - [Build Flatbuffers](#build-flatbuffers)
- [TensorFlow Lite C++ minimal example](#tensorflow-lite-c-minimal-example)

<!-- /code_chunk_output -->

## Pre-installations steps

### Update the environment
* You'll need to know if you have the latests Bullseye OS running on your Raspberry Pi or the Buster OS:
   ```bash
   cat /etc/os-release
   PRETTY_NAME="Raspbian GNU/Linux 11 (bullseye)"
   NAME="Raspbian GNU/Linux"
   VERSION_ID="11"
   VERSION="11 (bullseye)"
   VERSION_CODENAME=bullseye
   ```

* Update the EEPROM version if needed. See [Q-engineering's note](https://qengineering.eu/install-opencv-4.5-on-raspberry-pi-4.html) for more details.

   ```bash
   # to get the current status
   $ sudo rpi-eeprom-update
   # if needed, to update the firmware
   $ sudo rpi-eeprom-update -a
   $ sudo reboot
   ```

* Update installed software and make sure you have all important packages:
   ```bash
   sudo apt update && sudo apt dist-upgrade
   sudo apt-get install build-essential gawk gcc g++ gfortran git texinfo bison  wget bzip2 libncurses-dev libssl-dev openssl zlib1g-dev
   ```
### Update the CC Cross-Compiler Toolchains
There is an [issue](https://github.com/abhiTronix/raspberry-pi-cross-compilers/issues/90) with the installed Raspberry Pi GCC Compiler Toolchains for Bullseye that prevents to build Tensorflow Lite with gcc. This issue has been resolved in the Toolchains version v3.1.0 or higher. We must install this new version and enable it temporarily for building Tensorflow Lite. 

The Raspberry Pi GCC Toolchains Files are available on [SourceForge](https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/Raspberry%20Pi%20GCC%20Cross-Compiler%20Toolchains/Bullseye/GCC%2010.3.0/) and on [Abhishek Thakur's github](https://github.com/abhiTronix/raspberry-pi-cross-compilers). We need the latest version of the [Native Compiler Toolchains](https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/Raspberry%20Pi%20GCC%20Native-Compiler%20Toolchains/Bullseye/) which is [GCC10.3.0](https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/Raspberry%20Pi%20GCC%20Native-Compiler%20Toolchains/Bullseye/GCC%2010.3.0/) at the time of writing this note.

Here is a summary of the [instructions](https://github.com/abhiTronix/raspberry-pi-cross-compilers/wiki/Native-Compiler:-Installation-Instructions) for a Raspberry Pi 4:

* Download [native-gcc-10.3.0-pi_3+.tar.gz](https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/Raspberry%20Pi%20GCC%20Native-Compiler%20Toolchains/Bullseye/GCC%2010.3.0/Raspberry%20Pi%203A%2B%2C%203B%2B%2C%204/) or:
   <br/>
   ```bash
   mkdir gcctoolchains
   cd gcctoolchains
   wget https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/Raspberry%20Pi%20GCC%20Native-Compiler%20Toolchains/Bullseye/GCC%2010.3.0/Raspberry%20Pi%203A%2B%2C%203B%2B%2C%204/native-gcc-10.3.0-pi_3%2B.tar.gz/download
   ```
* Extract:
  <br/>
   ```bash
   tar xf native-gcc-10.3.0-pi_3+.tar.gz
   rm native-gcc-10.3.0-pi_3+.tar.gz
   ```

* Setup paths as follows:
   <br/>
   ```bash
   # use your extracted folder-name
   cd native-pi-gcc-10.3.0-2/
   pwd
   /home/toto/gcctoolchains/native-pi-gcc-10.3.0-2

   PATH=/home/toto/gcctoolchains/native-pi-gcc-10.3.0-2/bin:$PATH
   LD_LIBRARY_PATH=/home/toto/gcctoolchains/native-pi-gcc-10.3.0-2/lib:$LD_LIBRARY_PATH
   ```

* Setup important Symlinks as follows:
   <br/>
   ```bash
   cd ..
   wget https://raw.githubusercontent.com/abhiTronix/raspberry-pi-cross-compilers/master/utils/SSymlinker

   sudo chmod +x SSymlinker
   ./SSymlinker -s /usr/include/arm-linux-gnueabihf/asm -d /usr/include
   ./SSymlinker -s /usr/include/arm-linux-gnueabihf/gnu -d /usr/include
   ./SSymlinker -s /usr/include/arm-linux-gnueabihf/bits -d /usr/include
   ./SSymlinker -s /usr/include/arm-linux-gnueabihf/sys -d /usr/include
   ./SSymlinker -s /usr/include/arm-linux-gnueabihf/openssl -d /usr/include
   ./SSymlinker -s /usr/lib/arm-linux-gnueabihf/crtn.o -d /usr/lib/crtn.o
   ./SSymlinker -s /usr/lib/arm-linux-gnueabihf/crt1.o -d /usr/lib/crt1.o
   ./SSymlinker -s /usr/lib/arm-linux-gnueabihf/crti.o -d /usr/lib/crti.o
   ```

* Extra Step to use these binaries(temporarily) as your default native GCC Compiler (instead of default GCC ) at the time of compilation:
   <br/>
   ```bash
   export AR="gcc-ar-10.3.0"
   export CC="gcc-10.3.0"
   export CXX="g++-10.3.0"
   export CPP="cpp-10.3.0"
   export FC="gfortran-10.3.0"
   export RANLIB="gcc-ranlib-10.3.0"
   export LD="$CXX"
   ```

 Abhishek has also instructions for a [permanent installation](https://github.com/abhiTronix/raspberry-pi-cross-compilers/wiki/Native-Compiler:-Installation-Instructions#c2--permanent-installation) of the Toolchains. 


## Build TensorFlow Lite with CMake
I have follow the [instructions](https://www.tensorflow.org/lite/guide/build_cmake) from the TF Google team. As usual, not everything worked immediately and I had to make some modifications to obtain a successful build. Here are the steps to follow:

### Step 1. Clone TensorFlow repository
```bash
cd ~
mkdir tensorflow
cd tensorflow
git clone https://github.com/tensorflow/tensorflow.git tensorflow_src
```
### Step 2. Run CMake tool
```bash
mkdir tflite_build
cd tflite_build
# to generates an optimized release binary:
cmake ../tensorflow_src/tensorflow/lite
# if you need to produce a debug build which has symbol information:
cmake ../tensorflow_src/tensorflow/lite -DCMAKE_BUILD_TYPE=Debug
```
### Step 3. Fix includes
There were several compilation errors the first time I launched the build. They were caused by constants being not found in 2 files (e.g. SSMAX_SIZE, PATH_MAX). 

   1. 
      ```bash
      sudo nano /home/toto/tensorflow/tflite_build/flatbuffers/src/util.cpp
      ```
       and include <linux/limits.h>:
      ```cpp
      #ifdef_WIN32 
      ...       
      #else      
      #  define _XOPEN_SOURCE 600 // For PATH_MAX from limits.h (SUSv2 extension)     
      #  include <linux/limits.h>                                                     
      #endif
      ```  
   2.   
      ```bash
      sudo nano /home/toto/tensorflow/tflite_build/abseil-cpp/absl/debugging/symbolize.cc
      ```
      and make sure that `SSMAX_SIZE` is defined for Linux build:
      ```cpp
      #if defined(ABSL_INTERNAL_HAVE_ELF_SYMBOLIZE)
      #ifndef SSIZE_MAX
      #define SSIZE_MAX       LONG_MAX
      #endif
      #include "absl/debugging/symbolize_elf.inc"
      ```
   There is also a problem with the rounding method `RoundToNearest` that could not find the ARM function `vcvtaq_s32_f32`. Enable `vcvtq_s32_f32` on ARM:
  
   ```bash
   sudo nano /home/toto/tensorflow/tensorflow_src/tensorflow/lite/kernels/internal/optimized/neon_tensor_utils.cc
   ```
   In `RoundToNearest`, comment the line that calls `vcvtaq_s32_f32` that is not found on the system:
   ```cpp
   inline int32x4_t RoundToNearest(const float32x4_t input) {                       
   //#if __ARM_ARCH >= 8                                                            
   //  return vcvtaq_s32_f32(input);                                                
   //#else                                                                          
   static const float32x4_t zero_val_dup = vdupq_n_f32(0.0f);                     
   static const float32x4_t point5_val_dup = vdupq_n_f32(0.5f);                   
                                                                                    
   const int32x4_t mask = vreinterpretq_s32_u32(vcltq_f32(input, zero_val_dup));  
   const float32x4_t casted_mask = vcvtq_f32_s32(mask);                           
   const float32x4_t round = vaddq_f32(casted_mask, point5_val_dup);              
   return vcvtq_s32_f32(vaddq_f32(input, round));                                 
   //#endif                                                                         
   ```

### Step 4. Build

```bash
cmake --build . -j
```

Check that the proper gcc Toolchains is used (in my case 10.3.1):

![tfbuild1](/assets/images/tfbuild1.png)

This creates `libtensorflow-lite.a` in the current directory `tflite_build`.

## Build Flatbuffers
The TensorFlow Lite flat buffers are also needed. Please use the following commands copied from the TensorFlow Lite [Build with CMake](https://www.tensorflow.org/lite/guide/build_cmake#specifics_of_kernel_unit_tests_cross-compilation) guide:
```bash
cd ~/tensorflow
mkdir flatc-native-build && cd flatc-native-build
cmake ../tensorflow_src/tensorflow/lite/tools/cmake/native_tools/flatbuffers
cmake --build .
```


# TensorFlow Lite C++ minimal example

Let's try now to build and run the [TensorFlow Lite C++ minimal example](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/examples/minimal). We build first without XNNPACK which is enabled by default.

```bash
cd ~/tensorflow

# Create CMake build directory and run CMake tool
mkdir minimal_build_no_xnnpack
cd minimal_build_no_xnnpack
# -DTFLITE_ENABLE_XNNPACK=OFF to disable XNNPACK which is enabled by default
cmake ../tensorflow_src/tensorflow/lite/examples/minimal -DTFLITE_ENABLE_XNNPACK=OFF 
```
This creates a new build directory in which TensorFlow Lite will be built again. So, we have to make the same changes we made above to have no error during the build. See Step 3 above to remember the fixes. Here is the files to edit:
```bash
sudo nano /home/toto/tensorflow/minimal_build_no_xnnpack/abseil-cpp/absl/debugging/symbolize.cc
sudo nano /home/toto/tensorflow/minimal_build_no_xnnpack/flatbuffers/src/util.cpp
```
Finally, we can build:
```bash
cmake --build . 
```

After a successful build, try the minimal application with
```bash
./minimal mymodel.tflite
```
You can download a TF Lite model from the [TensorFlow Hub](https://tfhub.dev/s?deployment-format=lite). For this example, I used the [EfficientDet-Lite0 Object detection model](https://tfhub.dev/tensorflow/lite-model/efficientdet/lite0/detection/default/1):

```bash
./minimal efficientdet_lite0.tflite

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
Tensor   4 fpn_cells/cell_2/fnode... kTfLiteInt32    kTfLiteMmapRo      8        / 0.00 [2] [3730308, 3730316)
...
```

We are done. We can now build C++ TensorFlow Lite application natively on a Raspberry Pi 4.




