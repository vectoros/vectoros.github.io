---
title: Android Camera 格式
tags:
  - Android
  - Camera
  - Camera2
  - CameraService
categories:
  - Android
  - Framework
  - Camera
abbrlink: 69b727d7
date: 2022-04-04 17:34:14
---


Andriod Camera 推荐使用YUV格式，在Camera API1 中，推荐使用 NV21 和 YV12，因为这两种格式支持所有的Camera设备。
在Camera API2 中，推荐试用 YUV_420_888

# YUV简介
不同于RGB颜色模型，使用R,G,B三个颜色分量来表示颜色。YUV使用明亮度"Y", 色度"U,V"来表示颜色。
其中明亮度"Y"(Luninance或Luma)就是我们常见的灰度值, 色度(Chrominance或Chroma)用于指定像素的颜色。
其中：U(Cb), 代表蓝色； V(Cr), 代表红色。所有YUV还有一种说法就是YCbCr

YUV的优点：
1. 优化彩色视频信号传输，可以很方便的兼容老式黑白电视。黑白电视只需要处理Y分量，彩色在其基础上加上U(Cb), V(Cr)两个分量即可。
2. 降低传输的数据量，以常见的YUV420来看，每4个Y共用一组UV分量给，一个YUV占用 8+2+2 = 12bits = 1.5Byte；而RGB需要8+8+8 = 24bits = 3Byte；节省了一半的数据。


# 常用的格式
* YUV420SP
    * NV21
    ```
    YYYYYYYY VUVU
    ```

    * NV12
    ```
    YYYYYYYY UVUV
    ```


* YUV420P 平面模式， Y,U,V分别在不同的平面，是标准的4:2:0
    * YV12
    ```
    YYYYYYYY VVUU
    ```

    * I420(YU12)
    ```
    YYYYYYYY UUVV
    ```


# 内存布局
在实际使用中，一个YUV420图像大小，往往会比按照图像宽高计算的来的大。主要的原因就是一个数据对齐。这里有下面几个概念：
* width 图像宽度
* rowStride 跨度，图像实际存储的宽度 
* height 图像高度
* scanLines 扫描线。图像实际存储高度

在Android手机的Camera框架中，HAL层的Buffer，都是通过`GraphicBufferMapper`去分配处理的。我们调用`GraphicBufferMapper`的lock接口去获取`GraphicBuffer`时，传入的参数是图像的宽高，返回的实际内存大小是经过对齐后的。

## 高通 NV12
```c
/* Venus NV12:
 * YUV 4:2:0 image with a plane of 8 bit Y samples followed
 * by an interleaved U/V plane containing 8 bit 2x2 subsampled
 * colour difference samples.
 *
 * <-------- Y/UV_Stride -------->
 * <------- Width ------->
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  ^           ^
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  |           |
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  Height      |
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  |          Y_Scanlines
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  |           |
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  |           |
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  |           |
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  V           |
 * . . . . . . . . . . . . . . . .              |
 * . . . . . . . . . . . . . . . .              |
 * . . . . . . . . . . . . . . . .              |
 * . . . . . . . . . . . . . . . .              V
 * U V U V U V U V U V U V . . . .  ^
 * U V U V U V U V U V U V . . . .  |
 * U V U V U V U V U V U V . . . .  |
 * U V U V U V U V U V U V . . . .  UV_Scanlines
 * . . . . . . . . . . . . . . . .  |
 * . . . . . . . . . . . . . . . .  V
 * . . . . . . . . . . . . . . . .  --> Buffer size alignment
 *
 * Y_Stride : Width aligned to 128
 * UV_Stride : Width aligned to 128
 * Y_Scanlines: Height aligned to 32
 * UV_Scanlines: Height/2 aligned to 16
 * Extradata: Arbitrary (software-imposed) padding
 * Total size = align((Y_Stride * Y_Scanlines
 *          + UV_Stride * UV_Scanlines
 *          + max(Extradata, Y_Stride * 8), 4096)
 */

```
## 高通NV21
```c
/* Venus NV21:
 * YUV 4:2:0 image with a plane of 8 bit Y samples followed
 * by an interleaved V/U plane containing 8 bit 2x2 subsampled
 * colour difference samples.
 *
 * <-------- Y/UV_Stride -------->
 * <------- Width ------->
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  ^           ^
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  |           |
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  Height      |
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  |          Y_Scanlines
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  |           |
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  |           |
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  |           |
 * Y Y Y Y Y Y Y Y Y Y Y Y . . . .  V           |
 * . . . . . . . . . . . . . . . .              |
 * . . . . . . . . . . . . . . . .              |
 * . . . . . . . . . . . . . . . .              |
 * . . . . . . . . . . . . . . . .              V
 * V U V U V U V U V U V U . . . .  ^
 * V U V U V U V U V U V U . . . .  |
 * V U V U V U V U V U V U . . . .  |
 * V U V U V U V U V U V U . . . .  UV_Scanlines
 * . . . . . . . . . . . . . . . .  |
 * . . . . . . . . . . . . . . . .  V
 * . . . . . . . . . . . . . . . .  --> Padding & Buffer size alignment
 *
 * Y_Stride : Width aligned to 128
 * UV_Stride : Width aligned to 128
 * Y_Scanlines: Height aligned to 32
 * UV_Scanlines: Height/2 aligned to 16
 * Extradata: Arbitrary (software-imposed) padding
 * Total size = align((Y_Stride * Y_Scanlines
 *          + UV_Stride * UV_Scanlines
 *          + max(Extradata, Y_Stride * 8), 4096)
 */
```
