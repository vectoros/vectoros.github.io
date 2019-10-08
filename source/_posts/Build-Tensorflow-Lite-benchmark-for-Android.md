---
title: Build Tensorflow Lite benchmark for Android
tags:
  - TFLite
  - Android
categories:
  - Tensorflow
  - TFLite
  - Android
  - Benchmark
abbrlink: 1b1cf1e2
date: 2019-06-15 16:42:50
---

我们在部署模型到Android端的时候，需要先评估模型的性能，Tensorflow官方给我们提供了TFLite的benchmark工具

# 整体流程

1. 准备编译环境
2. 安装依赖项目
3. 安装编译工具
4. 下载tensorflow源码
5. 编译benchmark
6. 部署到Android设备上
7. 运行


## 准备编译环境

建议用Linux系统，这个部分可以参考[官方指南](https://www.tensorflow.org/install/source)， 我这边使用的Ubuntu 18.04
```bash
# install python and prerequisites
sudo apt install python-dev python-pip
pip install -U --user pip six numpy wheel mock
pip install -U --user keras_applications==1.0.6 --no-deps
pip install -U --user keras_preprocessing==1.0.5 --no-deps

# install openjdk-8
sudo apt-get install openjdk-8-jdk
```

## 安装编译工具`bazel`

参考[bazel官网](https://docs.bazel.build/versions/master/install.html)，编译最新版本需要bazel版本>=0.24.1

```bash
# install the prerequisites
sudo apt-get install pkg-config zip g++ zlib1g-dev unzip python

# download bazel from github
# Next, download the Bazel binary installer named bazel-<version>-installer-linux-x86_64.sh from the Bazel releases page on GitHub.
wget https://github.com/bazelbuild/bazel/releases/download/0.24.1/bazel-0.24.1-installer-linux-x86_64.sh


# run the installer 0.24.1
chmod +x bazel-0.24.1-installer-linux-x86_64.sh
./bazel-0.24.1-installer-linux-x86_64.sh --user

# setup environment
export PATH="$PATH:$HOME/bin"
```

## 安装Android SDK和NDK

编译Android平台的benchmark需要Android SDK和NDK（里面包含Build tools, Platform tools）
```bash
mkdir ~/Android
cd Android
# NDK
wget https://dl.google.com/android/repository/android-ndk-r17c-linux-x86_64.zip
unzip android-ndk-r17c-linux-x86_64.zip
mkdir Sdk
mv android-ndk-r17c Sdk/ndk-bundle

# SDK tools
wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip
unzip sdk-tools-linux-4333796.zip 
mv tools/ Sdk/


# Platform tools
wget https://dl.google.com/android/repository/platform-tools-latest-linux.zip
unzip platform-tools-latest-linux.zip 
mv platform-tools Sdk/

# Using sdkmanager to install tools
cd Sdk
chmod a+x tools/bin/sdkmanager
./tools/bin/sdkmanager "platform-tools" "platforms;android-28" "platforms;android-29" "build-tools;29.0.0" 

```

## 编译tflite benchmark

参考[官方文档](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/tools/benchmark)

```bash
# download tensorflow code
git clone https://github.com/tensorflow/tensorflow.git

# build benchmark for android
bazel build -c opt \
  --config=android_arm \
  --cxxopt='--std=c++11' \
  tensorflow/lite/tools/benchmark:benchmark_model

# build benchmark for desktop
bazel build -c opt --cxxopt='--std=c++11' tensorflow/lite/tools/benchmark:benchmark_model
```

## 在Android上运行

编译好的benchmark_model是一个Android平台的可运行的二进制文件，可以在adb shell里面直接运行

```bash
adb push bazel-bin/tensorflow/lite/tools/benchmark/benchmark_model /data/local/tmp

adb shell chmod +x /data/local/tmp/benchmark_model

adb push mobilenet_v2_1.0_224_quant.tflite /data/local/tmp

/data/local/tmp/benchmark_model \
  --graph=/data/local/tmp/mobilenet_v2_1.0_224_quant.tflite \
  --num_threads=10

# tashset f0 是 Pixel系列手机上，让程序运行在Big core上的命令
adb shell taskset f0 /data/local/tmp/benchmark_model \
  --graph=/data/local/tmp/mobilenet_v2_1.0_224_quant.tflite \
  --enable_op_profiling=true
```

## 运行结果

```bash
# mobilenet
taskset f0 /data/local/tmp/benchmark_model \
  --graph=/data/local/tmp/mobilenet_v2_1.0_224_quant.tflite \
  --enable_op_profiling=true

STARTING!
Min num runs: [50]
Min runs duration (seconds): [1]
Inter-run delay (seconds): [-1]
Num threads: [1]
Benchmark name: []
Output prefix: []
Min warmup runs: [1]
Min warmup runs duration (seconds): [0.5]
Graph: [/data/local/tmp/mobilenet_v2_1.0_224_quant.tflite]
Input layers: []
Input shapes: []
Use nnapi : [0]
Use legacy nnapi : [0]
Use gpu : [0]
Allow fp16 : [0]
Enable op profiling: [1]
Loaded model /data/local/tmp/mobilenet_v2_1.0_224_quant.tflite
resolved reporter
INFO: Initialized TensorFlow Lite runtime.
Initialized session in 1.892ms
Running benchmark for at least 1 iterations and at least 0.5 seconds
count=6 first=91810 curr=85097 min=84834 max=91810 avg=86184.3 std=2519

Running benchmark for at least 50 iterations and at least 1 seconds
count=50 first=118810 curr=114219 min=88216 max=129193 avg=115062 std=7678

Average inference timings in us: Warmup: 86184.3, Init: 1892, no stats: 115062
============================== Run Order ==============================
                     [node type]                  [start]         [first]        [avg ms]            [%]          [cdf%]  [mem KB]      [times called]  [Name]
                         CONV_2D                    0.000           5.567           7.237         6.297%          6.297%     0.000              1       [MobilenetV2/Conv/Relu6]
               DEPTHWISE_CONV_2D                    7.241           3.518           3.726         3.243%          9.539%     0.000              1       [MobilenetV2/expanded_conv/depthwise/Relu6]
                         CONV_2D                   10.971           2.399           3.711         3.229%         12.768%     0.000              1       [MobilenetV2/expanded_conv/project/add_fold]
                         CONV_2D                   14.688          12.247          17.846        15.529%         28.297%     0.000              1       [MobilenetV2/expanded_conv_1/expand/Relu6]
               DEPTHWISE_CONV_2D                   32.539           2.594           3.486         3.033%         31.330%     0.000              1       [MobilenetV2/expanded_conv_1/depthwise/Relu6]
                         CONV_2D                   36.028           1.460           2.462         2.142%         33.472%     0.000              1       [MobilenetV2/expanded_conv_1/project/add_fold]
                         CONV_2D                   38.493          12.619           7.820         6.805%         40.277%     0.000              1       [MobilenetV2/expanded_conv_2/expand/Relu6]
               DEPTHWISE_CONV_2D                   46.318           4.750           3.683         3.205%         43.482%     0.000              1       [MobilenetV2/expanded_conv_2/depthwise/Relu6]
                         CONV_2D                   50.005           4.853           3.305         2.876%         46.358%     0.000              1       [MobilenetV2/expanded_conv_2/project/add_fold]
                             ADD                   53.313           0.279           0.293         0.255%         46.613%     0.000              1       [MobilenetV2/expanded_conv_2/add]
                         CONV_2D                   53.607          12.993           7.827         6.811%         53.424%     0.000              1       [MobilenetV2/expanded_conv_3/expand/Relu6]
               DEPTHWISE_CONV_2D                   61.438           2.546           1.647         1.434%         54.857%     0.000              1       [MobilenetV2/expanded_conv_3/depthwise/Relu6]
                         CONV_2D                   63.089           1.239           1.009         0.878%         55.735%     0.000              1       [MobilenetV2/expanded_conv_3/project/add_fold]
                         CONV_2D                   64.099           2.261           2.151         1.872%         57.607%     0.000              1       [MobilenetV2/expanded_conv_4/expand/Relu6]
               DEPTHWISE_CONV_2D                   66.253           1.251           1.238         1.077%         58.685%     0.000              1       [MobilenetV2/expanded_conv_4/depthwise/Relu6]
                         CONV_2D                   67.493           1.074           1.126         0.980%         59.665%     0.000              1       [MobilenetV2/expanded_conv_4/project/add_fold]
                             ADD                   68.621           0.097           0.106         0.092%         59.757%     0.000              1       [MobilenetV2/expanded_conv_4/add]
                         CONV_2D                   68.728           2.234           2.056         1.789%         61.546%     0.000              1       [MobilenetV2/expanded_conv_5/expand/Relu6]
               DEPTHWISE_CONV_2D                   70.786           1.135           1.180         1.027%         62.573%     0.000              1       [MobilenetV2/expanded_conv_5/depthwise/Relu6]
                         CONV_2D                   71.968           1.073           1.174         1.022%         63.594%     0.000              1       [MobilenetV2/expanded_conv_5/project/add_fold]
                             ADD                   73.145           0.096           0.098         0.086%         63.680%     0.000              1       [MobilenetV2/expanded_conv_5/add]
                         CONV_2D                   73.244           1.992           2.172         1.890%         65.570%     0.000              1       [MobilenetV2/expanded_conv_6/expand/Relu6]
               DEPTHWISE_CONV_2D                   75.419           0.409           0.469         0.408%         65.978%     0.000              1       [MobilenetV2/expanded_conv_6/depthwise/Relu6]
                         CONV_2D                   75.889           0.395           0.445         0.387%         66.366%     0.000              1       [MobilenetV2/expanded_conv_6/project/add_fold]
                         CONV_2D                   76.335           1.277           1.172         1.020%         67.386%     0.000              1       [MobilenetV2/expanded_conv_7/expand/Relu6]
               DEPTHWISE_CONV_2D                   77.509           0.778           0.624         0.543%         67.929%     0.000              1       [MobilenetV2/expanded_conv_7/depthwise/Relu6]
                         CONV_2D                   78.135           1.041           0.779         0.678%         68.606%     0.000              1       [MobilenetV2/expanded_conv_7/project/add_fold]
                             ADD                   78.915           0.067           0.054         0.047%         68.653%     0.000              1       [MobilenetV2/expanded_conv_7/add]
                         CONV_2D                   78.969           1.059           1.147         0.998%         69.651%     0.000              1       [MobilenetV2/expanded_conv_8/expand/Relu6]
               DEPTHWISE_CONV_2D                   80.118           0.639           0.619         0.539%         70.190%     0.000              1       [MobilenetV2/expanded_conv_8/depthwise/Relu6]
                         CONV_2D                   80.738           0.620           0.767         0.667%         70.857%     0.000              1       [MobilenetV2/expanded_conv_8/project/add_fold]
                             ADD                   81.506           0.052           0.051         0.044%         70.902%     0.000              1       [MobilenetV2/expanded_conv_8/add]
                         CONV_2D                   81.558           1.271           1.139         0.991%         71.892%     0.000              1       [MobilenetV2/expanded_conv_9/expand/Relu6]
               DEPTHWISE_CONV_2D                   82.699           0.595           0.582         0.506%         72.399%     0.000              1       [MobilenetV2/expanded_conv_9/depthwise/Relu6]
                         CONV_2D                   83.282           0.853           0.750         0.652%         73.051%     0.000              1       [MobilenetV2/expanded_conv_9/project/add_fold]
                             ADD                   84.033           0.055           0.053         0.046%         73.097%     0.000              1       [MobilenetV2/expanded_conv_9/add]
                         CONV_2D                   84.087           1.078           1.127         0.981%         74.078%     0.000              1       [MobilenetV2/expanded_conv_10/expand/Relu6]
               DEPTHWISE_CONV_2D                   85.216           0.635           0.595         0.518%         74.596%     0.000              1       [MobilenetV2/expanded_conv_10/depthwise/Relu6]
                         CONV_2D                   85.813           1.022           1.106         0.963%         75.559%     0.000              1       [MobilenetV2/expanded_conv_10/project/add_fold]
                         CONV_2D                   86.921           2.031           2.034         1.770%         77.328%     0.000              1       [MobilenetV2/expanded_conv_11/expand/Relu6]
               DEPTHWISE_CONV_2D                   88.957           0.999           0.980         0.852%         78.181%     0.000              1       [MobilenetV2/expanded_conv_11/depthwise/Relu6]
                         CONV_2D                   89.939           1.476           1.588         1.382%         79.562%     0.000              1       [MobilenetV2/expanded_conv_11/project/add_fold]
                             ADD                   91.529           0.074           0.077         0.067%         79.629%     0.000              1       [MobilenetV2/expanded_conv_11/add]
                         CONV_2D                   91.607           2.472           2.057         1.790%         81.419%     0.000              1       [MobilenetV2/expanded_conv_12/expand/Relu6]
               DEPTHWISE_CONV_2D                   93.666           1.046           0.959         0.834%         82.253%     0.000              1       [MobilenetV2/expanded_conv_12/depthwise/Relu6]
                         CONV_2D                   94.627           1.548           1.564         1.361%         83.614%     0.000              1       [MobilenetV2/expanded_conv_12/project/add_fold]
                             ADD                   96.193           0.077           0.079         0.068%         83.683%     0.000              1       [MobilenetV2/expanded_conv_12/add]
                         CONV_2D                   96.272           2.107           2.032         1.768%         85.451%     0.000              1       [MobilenetV2/expanded_conv_13/expand/Relu6]
               DEPTHWISE_CONV_2D                   98.306           0.318           0.319         0.278%         85.729%     0.000              1       [MobilenetV2/expanded_conv_13/depthwise/Relu6]
                         CONV_2D                   98.626           0.717           0.732         0.637%         86.365%     0.000              1       [MobilenetV2/expanded_conv_13/project/add_fold]
                         CONV_2D                   99.359           1.280           1.413         1.230%         87.595%     0.000              1       [MobilenetV2/expanded_conv_14/expand/Relu6]
               DEPTHWISE_CONV_2D                  100.774           0.332           0.364         0.316%         87.911%     0.000              1       [MobilenetV2/expanded_conv_14/depthwise/Relu6]
                         CONV_2D                  101.142           1.191           1.238         1.077%         88.989%     0.000              1       [MobilenetV2/expanded_conv_14/project/add_fold]
                             ADD                  102.382           0.037           0.039         0.034%         89.023%     0.000              1       [MobilenetV2/expanded_conv_14/add]
                         CONV_2D                  102.421           1.305           1.459         1.270%         90.292%     0.000              1       [MobilenetV2/expanded_conv_15/expand/Relu6]
               DEPTHWISE_CONV_2D                  103.883           0.369           0.391         0.340%         90.633%     0.000              1       [MobilenetV2/expanded_conv_15/depthwise/Relu6]
                         CONV_2D                  104.275           1.214           1.239         1.078%         91.711%     0.000              1       [MobilenetV2/expanded_conv_15/project/add_fold]
                             ADD                  105.516           0.036           0.036         0.032%         91.743%     0.000              1       [MobilenetV2/expanded_conv_15/add]
                         CONV_2D                  105.552           1.312           1.405         1.222%         92.965%     0.000              1       [MobilenetV2/expanded_conv_16/expand/Relu6]
               DEPTHWISE_CONV_2D                  106.960           0.352           0.401         0.349%         93.314%     0.000              1       [MobilenetV2/expanded_conv_16/depthwise/Relu6]
                         CONV_2D                  107.362           2.322           2.311         2.011%         95.325%     0.000              1       [MobilenetV2/expanded_conv_16/project/add_fold]
                         CONV_2D                  109.675           3.883           3.400         2.958%         98.283%     0.000              1       [MobilenetV2/Conv_1/Relu6]
                 AVERAGE_POOL_2D                  113.078           0.129           0.125         0.109%         98.392%     0.000              1       [MobilenetV2/Logits/AvgPool]
                         CONV_2D                  113.204           1.732           1.844         1.604%         99.996%     0.000              1       [MobilenetV2/Logits/Conv2d_1c_1x1/BiasAdd]
                         RESHAPE                  115.050           0.005           0.005         0.004%        100.000%     0.000              1       [output]

============================== Top by Computation Time ==============================
                     [node type]                  [start]         [first]        [avg ms]            [%]          [cdf%]  [mem KB]      [times called]  [Name]
                         CONV_2D                   14.688          12.247          17.846        15.529%         15.529%     0.000              1       [MobilenetV2/expanded_conv_1/expand/Relu6]
                         CONV_2D                   53.607          12.993           7.827         6.811%         22.340%     0.000              1       [MobilenetV2/expanded_conv_3/expand/Relu6]
                         CONV_2D                   38.493          12.619           7.820         6.805%         29.145%     0.000              1       [MobilenetV2/expanded_conv_2/expand/Relu6]
                         CONV_2D                    0.000           5.567           7.237         6.297%         35.442%     0.000              1       [MobilenetV2/Conv/Relu6]
               DEPTHWISE_CONV_2D                    7.241           3.518           3.726         3.243%         38.684%     0.000              1       [MobilenetV2/expanded_conv/depthwise/Relu6]
                         CONV_2D                   10.971           2.399           3.711         3.229%         41.913%     0.000              1       [MobilenetV2/expanded_conv/project/add_fold]
               DEPTHWISE_CONV_2D                   46.318           4.750           3.683         3.205%         45.118%     0.000              1       [MobilenetV2/expanded_conv_2/depthwise/Relu6]
               DEPTHWISE_CONV_2D                   32.539           2.594           3.486         3.033%         48.151%     0.000              1       [MobilenetV2/expanded_conv_1/depthwise/Relu6]
                         CONV_2D                  109.675           3.883           3.400         2.958%         51.109%     0.000              1       [MobilenetV2/Conv_1/Relu6]
                         CONV_2D                   50.005           4.853           3.305         2.876%         53.985%     0.000              1       [MobilenetV2/expanded_conv_2/project/add_fold]

Number of nodes executed: 65
============================== Summary by node type ==============================
                     [Node type]          [count]         [avg ms]          [avg %]         [cdf %]       [mem KB]     [times called]
                         CONV_2D               36           92.625          80.619%         80.619%          0.000            36
               DEPTHWISE_CONV_2D               17           21.256          18.501%         99.120%          0.000            17
                             ADD               10            0.882           0.768%         99.888%          0.000            10
                 AVERAGE_POOL_2D                1            0.125           0.109%         99.997%          0.000             1
                         RESHAPE                1            0.004           0.003%        100.000%          0.000             1

Timings (microseconds): count=50 first=118487 curr=113927 min=88158 max=129072 avg=114924 std=7657
Memory (bytes): count=0
65 nodes observed
```

## 常见问题

1. C++11编译异常

bazel编译命令后，加上`--cxxopt='--std=c++11'` 选项

2. Bazel版本过低或者过高

安装0.24.1版本的bazel

