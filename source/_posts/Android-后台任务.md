---
title: Android 后台任务
tags:
  - Android
  - Background Tasks
categories:
  - Android
  - Application
  - Service
abbrlink: 9dbfc6f1
date: 2019-04-15 15:14:26
---

我们知道，在Android应用开发的过程中，不能在主线程中处理耗时任务，否则会导致UI卡顿，甚至ANR，非常影响用户体验，这就需要把这些耗时的任务，转移到后台去运行。

## 主线程工作
1. 更新UI（包括测量和更新View）
2. 协调用户交互
3. 接收生命周期事件

## 以下操作需要在独立的线程里面
1. 处理图片资源
2. 访问磁盘，读写文件
3. 处理网络请求
4. 任何其他耗时达到几百毫秒的操作
5. 当应用没有运行在前台，并且要在后台同步数据的时候

## 后台应用挑战
1. 后台应用会消耗设备的有限的资源，比如RAM和电池，这个可能会给用户带来不好的体验
2. 为了最大化电池的使用，Android系统会强制限制后台应用程序
    * Android 6.0，引入了Doze 和 App standby
    * Android 7.0，限制隐式广播和引入Doze-on-the-Go
    * Android 8.0, 限制后台行为，比如定位和Wekelocks
    * Android 9.0, 引入App Standby Buckets

## 如何选择合适的方案
1. 任务是否可以延迟，还是需要立刻执行？
2. 独立工作还是依赖系统环境？
3. 是否需要精确定时运行？

## Google推荐的方案
* WorkManager
    * Android Libraries (API Level 14+)
* Forground Service
    * 告知用户正在处理的重要内容
    * 前台服务是可见的
* AlarmManager
    * 定时任务
* DownloadManager
    * HTTP长连接

## 具体可以参考下面的流程
<img src="img/bg-job-choose.svg">

## 参考
* https://developer.android.com/guide/background/