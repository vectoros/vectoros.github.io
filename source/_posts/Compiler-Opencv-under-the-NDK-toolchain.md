---
title: Compiler OpenCV under the NDK toolchain
date: 2019-10-08 10:56:00
tags: [Android,NDK,OpenCV,CMake]
categories: 
- Android
- NDK
- OpenCV
---

## 为什么要手动编译OpenCV
正常情况下，我们利用OpenCV来开发Android应用，使用预编译的so和jar包已经足够了，但是在某些特殊情形下，我们希望自己编译OpenCV，借此来提高OpenCV的性能。

## 准备条件
1. Ubuntu 18.04 64Bit
2. cmake
3. [NDK Reversion 17.2.4988734](https://dl.google.com/android/repository/android-ndk-r17c-linux-x86_64.zip)
4. [OpenCV 4.0.1](https://opencv.org/releases/)
5. Android API
6. Target armeabi-v7a


## 下载和配置NDK
1. 从Google官网下载
```
wget https://dl.google.com/android/repository/android-ndk-r17c-linux-x86_64.zip
```

2. 下载后解压到特定目录
```bash
unzip android-ndk-r17c-linux-x86_64.zip
mv android-ndk-r17c /opt/ndk
```

## 下载OpenCV源码
1. 从官网下载最新版本或者特定版本
```bash
unzip opencv.zip
```

2. 从Github上下载
```bash
git clone https://github.com/opencv/opencv.git opencv
```

## 编译OpenCV
1. 创建输出目录
```bash
$ cd opencv
$ make build
$ cd build
```

2. 配置编译规则
```cmake
cmake -DCMAKE_TOOLCHAIN_FILE=/opt/ndk/build/cmake/android.toolchain.cmake \
-DANDROID_NDK=/opt/ndk \
-DANDROID_NATIVE_API_LEVEL=android-21 \
-DCMAKE_BUILD_TYPE=Release \
-DENABLE_VFPV3=ON \
-DANDROID_ARM_NEON=TRUE \
-DENABLE_NEON=ON \
-DBUILD_PNG=ON \
-DBUILD_JASPER=ON \
-DBUILD_JPEG=ON \
-DBUILD_TIFF=ON \
-DBUILD_ZLIB=ON \
-DWITH_JPEG=ON \
-DWITH_PNG=ON \
-DWITH_JASPER=ON \
-DWITH_TIFF=ON \
-DSOFTFP=ON \
-DBUILD_JAVA=OFF \
-DBUILD_ANDROID_EXAMPLES=OFF \
-DBUILD_ANDROID_PROJECTS=OFF \
-DANDROID_STL=c++_shared \
-DANDROID_ABI=armeabi-v7a \
-DBUILD_SHARED_LIBS=ON \
-DCMAKE_INSTALL_PREFIX=opencv/install \
..
```

3. 编译
```bash
$ make -j24 && make install
```

4. 检查生成文件
```bash
cd opencv/install/sdk
# etc 目录，存放了人脸识别模块部分资源文件，licenses等
vectoros@vectoros:~/opencv/install/sdk$ ll etc/
total 28
drwxrwxr-x 5 vectoros vectoros 4096 Oct  8 10:50 ./
drwxrwxr-x 4 vectoros vectoros 4096 Oct  8 10:50 ../
drwxrwxr-x 2 vectoros vectoros 4096 Oct  8 10:50 haarcascades/
drwxrwxr-x 2 vectoros vectoros 4096 Oct  8 10:50 lbpcascades/
drwxrwxr-x 2 vectoros vectoros 4096 Oct  8 10:50 licenses/
-rw-r--r-- 1 vectoros vectoros 2593 Aug 24 23:19 valgrind_3rdparty.supp
-rw-r--r-- 1 vectoros vectoros 4088 Aug 24 23:19 valgrind.supp

# native/jni/include 目录，存放了对应的.h .hpp头文件
vectoros@vectoros:~/opencv/install/sdk/native$ ll jni/include/
drwxrwxr-x 16 vectoros vectoros 4096 Oct  8 10:50 opencv2/

# native/libs/armeabi-v7a/ 目录，存放了jni的so库或者.a文件
vectoros@vectoros:~/opencv/install/sdk/native$ ll libs/armeabi-v7a/*.so
-rw-r--r-- 1 vectoros vectoros 13462060 Oct  8 10:48 libs/armeabi-v7a/libopencv_calib3d.so
-rw-r--r-- 1 vectoros vectoros 25141056 Oct  8 10:46 libs/armeabi-v7a/libopencv_core.so
-rw-r--r-- 1 vectoros vectoros 50133564 Oct  8 10:47 libs/armeabi-v7a/libopencv_dnn.so
-rw-r--r-- 1 vectoros vectoros  6983552 Oct  8 10:47 libs/armeabi-v7a/libopencv_features2d.so
-rw-r--r-- 1 vectoros vectoros  3502996 Oct  8 10:46 libs/armeabi-v7a/libopencv_flann.so
-rw-r--r-- 1 vectoros vectoros   525868 Oct  8 10:47 libs/armeabi-v7a/libopencv_highgui.so
-rw-r--r-- 1 vectoros vectoros 21684136 Oct  8 10:47 libs/armeabi-v7a/libopencv_imgcodecs.so
-rw-r--r-- 1 vectoros vectoros 26117996 Oct  8 10:46 libs/armeabi-v7a/libopencv_imgproc.so
-rw-r--r-- 1 vectoros vectoros  6184124 Oct  8 10:46 libs/armeabi-v7a/libopencv_ml.so
-rw-r--r-- 1 vectoros vectoros  3730776 Oct  8 10:49 libs/armeabi-v7a/libopencv_objdetect.so
-rw-r--r-- 1 vectoros vectoros  5078460 Oct  8 10:47 libs/armeabi-v7a/libopencv_photo.so
-rw-r--r-- 1 vectoros vectoros  7637264 Oct  8 10:49 libs/armeabi-v7a/libopencv_stitching.so
-rw-r--r-- 1 vectoros vectoros  3361504 Oct  8 10:47 libs/armeabi-v7a/libopencv_videoio.so
-rw-r--r-- 1 vectoros vectoros  3026088 Oct  8 10:49 libs/armeabi-v7a/libopencv_video.so
```

5. 讲生成的文件应用到AndroidStudio NDK Projects里面

## 编译配置解析
```cmake
-DCMAKE_TOOLCHAIN_FILE=/opt/ndk/build/cmake/android.toolchain.cmake # 指定编译工具链cmake脚本
-DANDROID_NDK=/opt/ndk # 指定NDK路径
-DANDROID_NATIVE_API_LEVEL=android-21 # 指定NDK API Level
-DCMAKE_BUILD_TYPE=Release # 指定编译类型，默认是Debug类型，Release会有性能上的提升
-DENABLE_VFPV3=ON # 开启VFP优化选项
-DANDROID_ARM_NEON=TRUE # 设置NDK支持NEON
-DENABLE_NEON=ON # 开启O彭CV NEON优化选项
-DBUILD_PNG=ON # 编译PNG模块
-DBUILD_JASPER=ON # 编译JASPER模块
-DBUILD_JPEG=ON # 编译JPEG模块
-DBUILD_TIFF=ON # 编译TIFF模块
-DBUILD_ZLIB=ON # 编译ZLIB模块
-DWITH_JPEG=ON # 生成JPEG模块
-DWITH_PNG=ON # 生成PNG模块
-DWITH_JASPER=ON # 生成JASPER模块
-DWITH_TIFF=ON # 生成TIFF模块
-DSOFTFP=ON # 开启SOFTFP
-DBUILD_JAVA=OFF # 编译Java，这里只需要C++版本，如果需要java，设置为ON
-DBUILD_ANDROID_EXAMPLES=OFF # 编译Android Examples
-DBUILD_ANDROID_PROJECTS=OFF # 编译Android Projects
-DANDROID_STL=c++_shared # 指定STL版本，新版NDK里面默认使用c++_shared
-DANDROID_ABI=armeabi-v7a # 指定生成ABI版本
-DBUILD_SHARED_LIBS=ON # 是否生成共享so库，如果不设置，默认生成.a静态库
-DCMAKE_INSTALL_PREFIX=opencv/install # 安装路径，用于生成需要的头文件和so或者.a静态库（可选）
.. # 指定OpenCV代码根目录（重要）
```

## 参考
1. [https://www.learnopencv.com/install-opencv-on-android-tiny-and-optimized/](https://www.learnopencv.com/install-opencv-on-android-tiny-and-optimized/)
2. [https://www.sisik.eu/blog/android/ndk/opencv-without-java](https://www.sisik.eu/blog/android/ndk/opencv-without-java)