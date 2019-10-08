---
title: Convert Yolov3 cfg and weights files to keras h5 model
tags:
  - Yolov3
  - Keras
  - TensorFlow
  - TFLite
categories:
  - Object Dection
  - TensorFlow
  - Keras
  - TFLite
abbrlink: c17fb91d
date: 2019-05-05 20:53:38
---

Yolov3 是一个非常有效的用于目标检查的One-Stage模型，Backbone使用的是Darknet53，在平时用Keras比较多，所以想将其cfg和weights与训练权重转换成Keras的h5模型，在Github上找寻了一番，发现[keras-yolo3](https://github.com/qqwweee/keras-yolo3)这个工具非常有效，故基于此项目做了一定的修改[keras_yolov3](https://github.com/vectoros/keras-yolo3)。

## 修改点：增加了对模型输入尺寸的自定义
原作的input size是未定义的输入尺寸，默认为416，为了使得转换模型更加灵活

* 将input size作为转换的输入参数
* input size必须是32的整数倍

## 如何界定输出的Tensor大小
仔细阅读论文，你会发现，输出的tensor尺寸和输入的tensor尺寸是相关联的，具体如下

* `outputs[0].shape = [input_size / 32, input_size / 32, (num_classes + 5) * 3] `
* `outputs[1].shape = [input_size * 2 / 32, input_size * 2 / 32, (num_classes + 5) * 3] `
* `outputs[2].shape = [input_size * 4 / 32, input_size * 4 / 32, (num_classes + 5) * 3] `

这个数据在后续转换为其他如TFLite的时候非常有用




