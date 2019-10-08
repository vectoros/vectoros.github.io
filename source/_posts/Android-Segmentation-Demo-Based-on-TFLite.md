---
title: Android Segmentation Demo Based on TFLite
tags: 'Android, TFlite, Segmentation'
categories: 'Android, TensorFlow'
abbrlink: 1ce7ae9a
date: 2019-03-24 15:10:26
---
[TFLite](https://www.tensorflow.org/lite) 是Google基于TensorFlow 推出的移动端深度学习工具。我们可以利用这个工具，将训练好的模型部署到移动端设备上，支持(Android和iOS).

### 如何应用

#### [选择模型](https://www.tensorflow.org/lite/devguide#1_choose_a_model)


#### [转换成TFLite模型](https://www.tensorflow.org/lite/devguide#2_convert_the_model_format)
利用TensorFlow官方提供的转换工具，可以建多种不同类型的模型，转换成TFLite支持的模型。目前支持
1. tf.GraphDef (.pb or .pbtxt)
2. tf.keras (.h5)


#### [部署到移动端设备上](https://www.tensorflow.org/lite/devguide#3_use_the_tensorflow_lite_model_for_inference_in_a_mobile_app)
将生成的.tflite模型部署到移动端或者嵌入式设备上。

#### [优化模型](https://www.tensorflow.org/lite/devguide#4_optimize_your_model_optional)
通过量化方式，将32位浮点型转换成更有效的8位整型，或者在移动端的GPU上运行。


### 实例分割Demo

#### 下载Google提供的GPU版本DeeplabV3 tflite模型

[下载地址](https://storage.googleapis.com/download.tensorflow.org/models/tflite/gpu/deeplabv3_257_mv_gpu.tflite)

1. 输入
2. 输出

#### 参考[TensorFlow examples](https://github.com/tensorflow/examples)

里面提供了多种类型的Demo，支持各个平台


### [开启GPU支持](https://www.tensorflow.org/lite/performance/gpu_advanced)
1. Module Gradle 依赖需要使用nightly版本
```gradle
repositories {
    maven {
        url 'https://google.bintray.com/tensorflow'
    }
}
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    /** ... **/
    implementation 'org.tensorflow:tensorflow-lite:0.0.1-gpu-experimental'
}
```
2. Java 代码中开启支持
```Java
// NEW: Prepare GPU delegate.
GpuDelegate delegate = new GpuDelegate();
Interpreter.Options options = (new Interpreter.Options()).addDelegate(delegate);

// Set up interpreter.
Interpreter interpreter = new Interpreter(model, options);

// Run inference.
writeToInputTensor(inputTensor);
interpreter.run(inputTensor, outputTensor);
readFromOutputTensor(outputTensor);

// Clean up.
delegate.close();
```
