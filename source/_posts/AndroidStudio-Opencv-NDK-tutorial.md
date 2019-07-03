---
title: AndroidStudio Opencv NDK tutorial
date: 2019-07-02 21:47:09
tags: [AndroidStudio,Gradle,Tutorial]
categories:
- Android
---

## 简介
* Opencv是目前计算机视觉里面非常常用的一个开源库，支持各大平台

## 准备条件
* AndroidStudio 最新版本
* SDK Manager里面安装SDK, NDK
* Opencv 3.4/4.0


## 准备知识-CMake
AndroidStudio 支持cmake编译C/C++文件，通过编写CMakeList.txt来实现

### 常用变量
```makefile
${CMAKE_SOURCE_DIR} # CMakeLists.txt文件所在位置
${ANDROID_ABI} # Android ABI架构 ("armeabi-v7a","arm64-v8a")
```

### 常用函数

* 设置变量值
```makefile
set(VARIABLE, VALUE)
```

* 添加共享库
```makefile
add_library( 
# Sets the name of the library.
native-lib
# Sets the library as a shared library.
SHARED
# Provides a relative path to your source file(s).
src/main/cpp/native-lib.cpp)
```

* 添加可执行文件
```makefile
add_executable( 
# Sets the name of the executable file.
test
# Provides a relative path to your source file(s).
src/main/cpp/main.cpp)
```

* 查找NDK library
```makefile
# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)
```

* 指定链接库
```makefile
# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
        test
        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
```

* 指定包含头文件
```makefile
include_directories(${CMAKE_SOURCE_DIR}/src/main/cpp/include/)
include_directories(${CMAKE_SOURCE_DIR}/src/main/cpp/opencv)
```

* 手动指定共享库
```makefile
add_library(libopencv_java3 SHARED IMPORTED)
set_target_properties(libopencv_java3 PROPERTIES IMPORTED_LOCATION
        ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libopencv_java3.so)
```


### 编译器设置
我们可以在CMakeLists.txt里面设置编译器参数，也可以在build.gradle脚本里面设置

* CMakeLists.txt 设置
```

```

* build.gradle 脚本设置
```gradle
apply plugin: 'com.android.application'

android {
    defaultConfig {
        externalNativeBuild {
            cmake {
                //设置cpp Flags
                cppFlags "-std=c++11 -frtti -fexceptions"
                //设置STL共享库，这里要注意，部分Android系统里面可能没有libc++_shared.so，这个时候需要在app里面打包进去
                //文件路径在NDK安装目录下 sources/cxx-stl/llvm-libc++/libs/${ANDROID_ABI}/libc++_shared.so
                arguments "-DANDROID_STL=c++_shared"
            }
        }
    }
}
```

### 输出设置
* 动态共享库 `.so`
```makefile
include_directories(${CMAKE_SOURCE_DIR}/src/main/cpp/opencv)
include_directories(${CMAKE_SOURCE_DIR}/src/main/cpp/include/)
add_library(libopencv_java3 SHARED IMPORTED)
set_target_properties(libopencv_java3 PROPERTIES IMPORTED_LOCATION
        ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libopencv_java3.so)
find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)
add_library( # Sets the name of the library.
        native-lib

        # Sets the library as a shared library.
        SHARED

        # Provides a relative path to your source file(s).

        src/main/cpp/native-lib.cpp)
target_link_libraries( # Specifies the target library.
        native-lib
        libopencv_java3
        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
```

* 静态共享库 `.a`
```makefile
include_directories(${CMAKE_SOURCE_DIR}/src/main/cpp/opencv)
include_directories(${CMAKE_SOURCE_DIR}/src/main/cpp/include/)
add_library(libopencv_java3 SHARED IMPORTED)
set_target_properties(libopencv_java3 PROPERTIES IMPORTED_LOCATION
        ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libopencv_java3.so)
find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)
add_library( # Sets the name of the library.
        native-lib

        # Sets the library as a static library.
        STATIC

        # Provides a relative path to your source file(s).

        src/main/cpp/native-lib.cpp)
target_link_libraries( # Specifies the target library.
        native-lib
        libopencv_java3
        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
```

* Executable
```makefile
include_directories(${CMAKE_SOURCE_DIR}/src/main/cpp/opencv)
include_directories(${CMAKE_SOURCE_DIR}/src/main/cpp/include/)
add_library(libopencv_java3 SHARED IMPORTED)
set_target_properties(libopencv_java3 PROPERTIES IMPORTED_LOCATION
        ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libopencv_java3.so)
find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)
# 设置可执行文件的输出目录
set(EXECUTABLE_OUTPUT_PATH      "${CMAKE_SOURCE_DIR}/src/main/assets/${ANDROID_ABI}")
# 添加可执行文件的源文件
add_executable(
        test

        src/main/cpp/main.cpp
)
target_link_libraries( # Specifies the target library.
        test
        libopencv_java3
        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
```

### 依赖设置

### 与Gradle结合


## 新建Project
用AndroidStudio创建项目，并选择Native C++支持

## 导入opencv头文件

## 导入opencv so库

## 编译验证


