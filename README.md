# zig-and-llvm-build-instructions

[Zig](https://ziglang.org) and [LLVM](https://llvm.org) build instructions. These are instructions
I used for Arch Linux and may work on other linuxes.

## Prerequisets
* clang 6.0.1
* cmake 3.12

## Build LLVM 
I'm using [mkllvm-tool-chain](https://github.com/winksaville/mkllvm-tool-chain) to build llvm:
```
git clone git@github.com:winksaville/mkllvm-tool-chain
cd mkllvm-tool-chain
CC=clang CXX=clang++ make llvm-7.0.0 LLVM_INSTALL_DIR=$HOME/llvm-clang
```
## Build Zig
```
git clone git@github.com:ziglang/zig
cd zig
mkdir build
cd build
CC=clang CXX=clang++ cmake .. -G Ninja -DCMAKE_INSTALL_PREFIX=$HOME/opt -DCMAKE_PREFIX_PATH=$HOME/llvm-clang
ninja install
export LD_LIBRARY_PATH=$HOME/llvm-clang/lib
```
## Verify Zig runs
```
~/opt/bin/zig version
```
## Explanation
To get Zig working both llvm and zig need to use the same compiler. I needed to use clang as I was getting errors compiling zig with gcc (compiled llvm fine):
```
FAILED: CMakeFiles/zig_cpp.dir/src/zig_llvm.cpp.o 
/usr/bin/c++   -I../deps/lld/include -I/home/wink/llvm-gcc/include -I../deps/SoftFloat-3e/source/include -I../ -I. -I../src -g   -std=c++11 -Werror -Wall -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS -D_GNU_SOURCE -fno-exceptions -fno-rtti  -Werror=strict-prototypes -Werror=old-style-definition -Werror=type-limits -Wno-missing-braces -MD -MT CMakeFiles/zig_cpp.dir/src/zig_llvm.cpp.o -MF CMakeFiles/zig_cpp.dir/src/zig_llvm.cpp.o.d -o CMakeFiles/zig_cpp.dir/src/zig_llvm.cpp.o -c ../src/zig_llvm.cpp
In file included from /home/wink/llvm-gcc/include/llvm/ADT/STLExtras.h:21,
                 from /home/wink/llvm-gcc/include/llvm/ADT/StringRef.h:13,
                 from /home/wink/llvm-gcc/include/llvm/ADT/StringMap.h:17,
                 from /home/wink/llvm-gcc/include/llvm/Support/Host.h:17,
                 from /home/wink/llvm-gcc/include/llvm/ADT/Hashing.h:49,
                 from /home/wink/llvm-gcc/include/llvm/ADT/ArrayRef.h:13,
                 from /home/wink/llvm-gcc/include/llvm/ADT/DenseMapInfo.h:17,
                 from /home/wink/llvm-gcc/include/llvm/ADT/DenseMap.h:17,
                 from /home/wink/llvm-gcc/include/llvm/Analysis/TargetLibraryInfo.h:13,
                 from ../src/zig_llvm.cpp:18:
/home/wink/llvm-gcc/include/llvm/ADT/SmallVector.h: In instantiation of ‘void llvm::SmallVectorTemplateBase<T, true>::push_back(const T&) [with T = std::pair<void*, long unsigned int>]’:
/home/wink/llvm-gcc/include/llvm/Support/Allocator.h:249:33:   required from ‘void* llvm::BumpPtrAllocatorImpl<AllocatorT, SlabSize, SizeThreshold>::Allocate(size_t, size_t) [with AllocatorT = llvm::MallocAllocator; long unsigned int SlabSize = 4096; long unsigned int SizeThreshold = 4096; size_t = long unsigned int]’
/home/wink/llvm-gcc/include/llvm/Support/YAMLParser.h:138:42:   required from here
/home/wink/llvm-gcc/include/llvm/ADT/SmallVector.h:313:11: error: ‘void* memcpy(void*, const void*, size_t)’ writing to an object of type ‘struct std::pair<void*, long unsigned int>’ with no trivial copy-assignment; use copy-assignment or copy-initialization instead [-Werror=class-memaccess]
     memcpy(this->end(), &Elt, sizeof(T));
     ~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```
* Build LLVM toolchain, I used [mkllvm-tool-chain](https://github.com/winksaville/mkllvm-tool-chain).
  * C=clang CXX=clang++ make llvm-7.0.0 LLVM_INSTALL_DIR=$HOME/llvm-clang
    * Currently LLVM_LINK_LLVM_DYLIB=ON is the default
    * Zig required that I build the tool chain with LLVM_LINK_LLVM_DYLIB := ON see LLVM Cmake docs for more info. When LLVM_LINK_LLVM_DYLIB is OFF (the default) then the default  will default to linking statically and zig cmake build doesn’t seem to handle statically linking zig to the llvm build tools.

* Zig build makefile using cmake:
  * CC=clang CXX=clang++ cmake .. -G Ninja -DCMAKE_INSTALL_PREFIX=$HOME/opt -DCMAKE_PREFIX_PATH=$HOME/llvm-clang
    * CC=clang CXX=clang++				c and c++ compilers
    * cmake 							Run cmake
    * .. 							Directory with CMakeLists.txt
    * -DCMAKE_INSTALL_PREFIX=$HOME/opt		Install directory for zig
    * -DCMAKE_PREFIX_PATH=$HOME/llvm-clang	Directory where llvm was installed (LLVM_INSTALL_DIR)
* Build Zig and install it
  * ninja install
* Optional - LD_LIBRARY_PATH if zig reports it can't find libLLVM-7.so
  export LD_LIBRARY_PATH=$HOME/llvm-clang/lib
* Run zig likely you'll want to add $HOME/opt/bin to your PATH but you can execute it directly too:
  * ~/opt/bin/zig version
