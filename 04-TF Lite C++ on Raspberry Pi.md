# Native Tensorflow Lite build on Raspberry Pi
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
* Install new version of installed software and make sure you have all important packages:
   ```bash
   sudo apt update && sudo apt dist-upgrade
   sudo apt-get install build-essential gawk gcc g++ gfortran git texinfo bison  wget bzip2 libncurses-dev libssl-dev openssl zlib1g-dev
   ```
### Update the CC Cross-Compiler Toolchains
There is an [issue](https://github.com/abhiTronix/raspberry-pi-cross-compilers/issues/90) with the Raspberry Pi GCC Cross-Compiler Toolchains for Bullseye that prevent to build Tensorflow Lite with gcc. This issue has been resolved in the latest Toolchains version v3.1.0. We must install this new version and enable it temporarily for building Tensorflow Lite. Here is a summary of the [instructions](https://github.com/abhiTronix/raspberry-pi-cross-compilers/wiki/Native-Compiler:-Installation-Instructions):

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

You can find more on the Raspberry Pi GCC Toolchains Files on [SourceForge](https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/Raspberry%20Pi%20GCC%20Cross-Compiler%20Toolchains/Bullseye/GCC%2010.3.0/) and on [Abhishek Thakur's github](https://github.com/abhiTronix/raspberry-pi-cross-compilers). Abhishek has also instructions for [permanent installation](https://github.com/abhiTronix/raspberry-pi-cross-compilers/wiki/Native-Compiler:-Installation-Instructions#c2--permanent-installation) of the Toolchains.


## Build TensorFlow Lite with CMake
I have follow the [instructions](https://www.tensorflow.org/lite/guide/build_cmake) from the TF Google team. As usual, not everything worked immediately and I had to make some modifications to obtain a successful build. Here are the steps to follow:

### Step 1. Clone TensorFlow repository
```bash
cd ~
mkdir tensorflow
cd tensorflow
git clone https://github.com/tensorflow/tensorflow.git tensorflow_src
```
### Step 2. Run CMake tool with configurations
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
  3.   
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
This creates libtensorflow-lite.a in the current directory 











- [Install OpenCV 4.5](https://qengineering.eu/install-opencv-4.5-on-raspberry-pi-4.html)
