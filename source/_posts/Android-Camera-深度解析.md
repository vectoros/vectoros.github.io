---
title: Android Camera 深度解析
abbrlink: 31ae23dc
date: 2022-01-02 22:14:59
tags:
categories:
---

这是一个系列的文章，主要从HAL层，Framework层，应用层三个方面来刨析Android Camera框架的实现原理和设计理念。源码以AOSP android 11版本为例。主要参考Android官网的文档。

这里我会从上到下依次按层级的方式，以Camera2 API和HAL3来解读Android Camera 的源码，中间回掺杂一些额外的补充知识点。

首先看下SDK中的相关内容

## 相关的类

在Camera API2 中，主要用到以下几个Package/类

* Package: android.hardware.camera2
* Camera: API1的接口
* SurfaceView: 处理预览，渲染的呈现
* MediaRecorder: 用于录像
* Intent: 提供`MediaStore.ACTION_IMAGE_CAPTURE` 或者 `MediaStore.ACTION_VIDEO_CAPTURE` 的Intent操作类型用于捕获图像或者视频，不需要直接使用Camera对象


## 清单声明

### 权限申请

* 相机权限
```xml
<uses-permission android:name="android.permission.CAMERA" />
```
使用现有相机来打开相机（Intent），不需要申请权限

* 存储权限
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

* 录音权限
```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

* 位置权限
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
...
<!-- Needed only if your app targets Android 5.0 (API level 21) or higher. -->
<uses-feature android:name="android.hardware.location.gps" />
```

### 功能声明

```xml
<uses-feature android:name="android.hardware.camera" />
```
非必需情况
```
<uses-feature android:name="android.hardware.camera" android:required="false" />
```


## 开发相机应用

在官方开发文档中，建议开发者使用Camera X来开发相机应用。




参考：
https://developer.android.com/guide/topics/media/camera

https://developer.android.com/training/camera2
