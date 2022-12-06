# SYCL Support for PIM  

## Introduction

The purpose of this repostiry is to support PIM with SYCL (DPC++)_. 

## How to use PIM Patches 

### HIP 

```bash
# Follow the ROCM 4.0.0 platform libraries to support PIM operations 
# You can see the PIMPatches README.md in detail
git clone https://github.com/ROCm-Developer-Tools/HIP.git
export HIP_DIR="$(readlink -f HIP)"
cd $HIP_DIR
git checkout -b rocm-4.0.0
cp <PIMPatches>/rocm/0001-Register-FIM-memory-in-memory-Managemenr-of-HIP.patch .
git am 0001-Register-FIM-memory-in-memory-Managemenr-of-HIP.patch

# Also you need to apply patch for DPCPP compile
cp <PIMPatches>/SYCL_dpcpp/0001-Accept-code-triple-generated-by-DPC-for-SYCL.patch .
git am 0001-Accept-code-triple-generated-by-DPC-for-SYCL.patch

# Build and Install
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DHIP_COMPILER=clang -DHIP_PLATFORM=rocclr -DCMAKE_PREFIX_PATH="$ROCclr_DIR/build;/opt/rocm-4.0.0/" -DCMAKE_INSTALL_PREFIX=/opt/rocm-4.0.0 -DHSA_PATH=/opt/rocm-4.0.0 ..
make -j
sudo make install
```


### DPCPP
After HIP installaiton, yu need to setup ROCT and PIMLibrary again. 

####PIMMock
PIMMock is required as below;

```bash
#Prerequisite: cmake>=3.14.1, ninja, C++17 Compiler (i.e) gcc-8, g++-8)
sudo apt install ninja-build
sudo apt-get install gcc-8
sudo apt-get install g++-8

# Build and Install 
export BASE_DIR=`pwd`
git clone https://github.com/SAITPublic/PIMMock.git pimmock
cd pimmock
git submodule init
git submodule update

mkdir build
cd build
cmake -G Ninja -DCMAKE_CXX_COMPILER=/usr/bin/g++-8 -DCMAKE_C_COMPILER=/usr/bin/gcc-8 -DCMAKE_INSTALL_PREFIX=$BASE_DIR/pimmock/install ..
ninja
ctest
ninja install
```

#### DPCPP

```bash
cd $BASE_DIR
# Please check the prerequisites from https://github.com/intel/llvm/blob/sycl/sycl/doc/GetStartedGuide.md
git clone -b sycl https://github.com/intel/llvm.git 
export DPCPP_DIR="$(readlink -f llvm)"
cd $DPCPP_DIR
cp <PIMPatches>/SYCL_dpcpp/00##.patch . 

# Apply patch 
# Please check the permission of 'applyPIMPatch.sh' 
# chmod +x applyPIMPatch.sh
./applyPIMPatch.sh

mkdir build
cd build

# Please check the PATH for pimmock, hip, pim library 
# When you use the pim library docker, you can use the below example. 
python ../buildbot/configure.py \
-t RelWithDebInfo \
--llvm-external-projects=lld,openmp \
--cmake-opt="-DLLVM_ENABLE_NEW_PASS_MANAGER=ON" \
--cmake-opt="-DSYCL_BUILD_PI_HIP_HSA_INCLUDE_DIR=/opt/rocm-4.0.0/include" \
--cmake-opt="-DSYCL_BUILD_PI_HIP_INCLUDE_DIR=/opt/rocm-4.0.0/include/" \
--cmake-opt="-DSYCL_BUILD_PIM_MOCK_DIR=$BASE_DIR/pimmock/install" \
--cmake-opt="-DPIM_HEADERS=$BASE_DIR/PIMLibrary/runtime/include" \
--cmake-opt="-DSYCL_BUILD_PI_PIM_SDK_LIBRARY=$BASE_DIR/PIMLibrary/build/runtime/source/libPimRuntime.so" \
--hip --hip-platform=AMD -o ./

ninja sycl-toolchain

# Make sure the following command lists a Samsung PIM device
# We assume that PIM has only one 
HIP_VISIBLE_DEVICES=1 ./bin/sycl-ls 
HIP_VISIBLE_DEVICES=1 ninja check-sycl-PiPimTest
```



