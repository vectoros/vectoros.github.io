---
title: Android 后台任务之 IntentService
date: 2019-04-15 15:10:22
tags: [Android, Background Tasks, IntentService]
categories:
- Android
- Develop
---

IntentService 是Android SDK中引入的一个非常好用的，有效的单线程后台任务工具

## 优点
* 不影响UI响应
* 不受生命周期影响

## 限制
* 不能和UI直接交互
* 任务顺序运行，一次只能运行一个任务
* 不能被中断

## 创建方式
1. 继承IntentService类
2. 提供一个默认的构造函数
3. 实现`onHandleIntent(Intent workIntent)`来处理后台任务
4. 在Manifest文件中声明

## 使用IntentService
1. 创建一个Intent，将任务数据用Intent发送给IntentService
2. 调用`IntentService.enqueWork()`启动`IntentService`

## 如何将IntentService处理完的数据返回到Activity
1. 使用`LocalBroadcastManager`，任务完成后，通过广播把数据用Intent返回到Activity


## Demo
```Java
```


## 参考
* https://developer.android.com/guide/components/services
* https://developer.android.com/training/run-background-service/send-request
* https://developer.android.com/training/run-background-service/report-status



