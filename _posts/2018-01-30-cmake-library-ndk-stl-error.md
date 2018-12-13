---
title: "Android NDK CMake 빌드 중 라이브러리 메소드 사용시 에러 발생"
date: 2018-01-30 23:39:09 +0900
categories: android
layout: post
comments: true
pinned: true
---
# 1. 문제점
C++로 작성된 정적 라이브러리 파일을 안드로이드 스튜디오에서 NDK를 이용하여 추가해 사용하려 했으나 다음과 같은 오류가 발생하였다.
```bash
FAILURE: Build failed with an exception.
* What went wrong:
Execution failed for task ':libProject:externalNativeBuildDebug'.
> Build command failed.
  Error while executing process /Users/username/Library/Android/sdk/cmake/3.6.4111459/bin/cmake with arguments {--build /Users/username/Documents/androidProject/libProject/.externalNativeBuild/cmake/debug/arm64-v8a --target native-lib}
  [1/3] Building CXX object CMakeFiles/native-lib.dir/src/main/cpp/SomeFile.cpp.o
  [2/3] Building CXX object CMakeFiles/native-lib.dir/src/main/cpp/SomeFile2.cpp.o
  [3/3] Linking CXX shared library ../../../../build/intermediates/cmake/debug/obj/arm64-v8a/libnative-lib.so
  FAILED: : && /Users/username/Documents/android-ndk-r16b/toolchains/llvm/prebuilt/darwin-x86_64/bin/clang++  --target=aarch64-none-linux-android --gcc-toolchain=/Users/username/Documents/android-ndk-r16b/toolchains/aarch64-linux-android-4.9/prebuilt/darwin-x86_64 --sysroot=/Users/username/Documents/android-ndk-r16b/sysroot -fPIC -isystem /Users/username/Documents/android-ndk-r16b/sysroot/usr/include/aarch64-linux-android -D__ANDROID_API__=22 -g -DANDROID -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -Wa,--noexecstack -Wformat -Werror=format-security  -std=c++11 -fexceptions -O0 -fno-limit-debug-info  -Wl,--exclude-libs,libgcc.a -Wl,--exclude-libs,libatomic.a --sysroot /Users/username/Documents/android-ndk-r16b/platforms/android-22/arch-arm64 -Wl,--build-id -Wl,--warn-shared-textrel -Wl,--fatal-warnings -Wl,--no-undefined -Wl,-z,noexecstack -Qunused-arguments -Wl,-z,relro -Wl,-z,now -shared -Wl,-soname,libnative-lib.so -o ../../../../build/intermediates/cmake/debug/obj/arm64-v8a/libnative-lib.so CMakeFiles/native-lib.dir/src/main/cpp/SomeFile.cpp.o CMakeFiles/native-lib.dir/src/main/cpp/SomeFile2.cpp.o  ../../../../libs/CoreLib/libs/arm64-v8a/libAnyfiMesh_CoreLib.a -llog -latomic -lm "/Users/username/Documents/android-ndk-r16b/sources/cxx-stl/gnu-libstdc++/4.9/libs/arm64-v8a/libgnustl_static.a" && :
  ../../../../libs/CoreLib/libs/arm64-v8a/libAnyfiMesh_CoreLib.a(some_file3.cpp.o): In function `SomeClass::someMethod2()':
  /Users/username/Documents/libraryProject/src/Common/some_file3.cpp:(.text+0x384): undefined reference to `std::__ndk1::__shared_weak_count::__release_weak()'
  /Users/username/Documents/libraryProject/src/Common/some_file3.cpp:(.text+0x3bc): undefined reference to `std::__ndk1::__shared_weak_count::__release_weak()'
  /Users/username/Documents/libraryProject/src/Common/some_file3.cpp:(.text+0x3f4): undefined reference to `std::__ndk1::__shared_weak_count::__release_weak()'
  ../../../../libs/CoreLib/libs/arm64-v8a/libAnyfiMesh_CoreLib.a(some_file3.cpp.o): In function `SomeClass::someMethod3() const':
  /Users/username/Documents/libraryProject/src/Common/some_file3.cpp:(.text+0x5d4): undefined reference to `std::__ndk1::some_method::init(void*)'
  /Users/username/Documents/libraryProject/src/Common/some_file3.cpp:(.text+0x614): undefined reference to `std::__ndk1::locale::locale()'
  /Users/username/Documents/libraryProject/src/Common/some_file3.cpp:(.text+0x798): undefined reference to `std::__ndk1::locale::~locale()'
  /Users/username/Documents/libraryProject/src/Common/some_file3.cpp:(.text+0x7a0): undefined reference to `std::__ndk1::some_method::~some_method()'
  /Users/username/Documents/libraryProject/src/Common/some_file3.cpp:(.text+0x82c): undefined reference to `std::__ndk1::locale::~locale()'
  /Users/username/Documents/libraryProject/src/Common/some_file3.cpp:(.text+0x834): undefined reference to `std::__ndk1::some_method::~some_method()'
  ../../../../libs/CoreLib/libs/arm64-v8a/libAnyfiMesh_CoreLib.a(some_file3.cpp.o): In function `ByteBufferPool::acquire()':
  /Users/username/Documents/libraryProject/src/Common/some_file3.cpp:(.text._ZN14ByteBufferPool7acquireEv[_ZN14ByteBufferPool7acquireEv]+0xa0): undefined reference to `std::__ndk1::__shared_weak_count::__release_weak()'
  .... 이하 생략 ...
```
라이브러리 빌드도 완료되었으며 사용한 NDK Toolchain 버전도 일치해서 문제없을거라 생각했는데 오류가 발생하여 구글링을 진행하였다.

# 2. 원인
구글링 결과 원인은 생각보다 간단했다. C++ 런타임 라이브러리가 달라 발생한 것이다. [Polly](https://github.com/ruslo/polly)를 사용하면서 C++ 라이브러리를 만들 때는 libc++ (C++_static)를 썼지만,
Android Studio에서는 기본값인 gnustl_static을 사용하여 에러가 발생했다.

# 3. 해결방안
라이브러리의 STL과 Android Studio의 STL을 맞춰주기만 하면 된다.
build.gradle에서 cmake argument에서 라이브러리와 똑같은 런타임 라이브러리로 맞춰주면 된다.
```bash
defaultConfig {
        minSdkVersion 22
        ...

        externalNativeBuild {
            cmake {
                arguments "-DANDROID_STL=c++_static" //, "-DANDROID_TOOLCHAIN=clang"
                cppFlags "-std=c++11 -fexceptions"
            }
        }
    }
```
`-DANDROID_TOOLCHAIN=clang`은 써도 안써도 그만이지만, C++ 라이브러리 빌드와 맞춰 사용하기 위해 추가하였다.