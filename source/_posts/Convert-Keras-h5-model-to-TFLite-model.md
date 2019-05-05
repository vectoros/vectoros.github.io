---
title: Convert Keras h5 model to TFLite model
date: 2019-05-05 21:15:09
tags: [Keras, TFLite]
categories:
- TensorFlow
- Keras
- Python
---

目前在移动端（Android）使用比较广泛的深度模型框架是TFLite，这个也是Google大力推广的。但是目前很多高效的网络都没有官方的TensorFlow版本，所以在使用的时候，我们需要将其他格式的模型，转换成TFLite格式。
好在Google提供了非常全的转换工具，在最新版本中，Keras已经成为了官方推荐的前端。

## 模型转换前的准备
* 确定源模型的格式
* 确定源模型的输入名称，输入尺寸
* 确定源模型的输出名称，输出尺寸

## 下面以Yolov3的cfg和weights文件转换成TFLite为例
Yolo的Backbone是Darknet，首先需要转换成支持的keras模型

1. 转换成Keras h5 格式
使用修改后的工具[keras_yolov3](https://github.com/vectoros/keras-yolo3), 用里面的convert.py文件，将模型转成keras的h5格式

```python
wget https://pjreddie.com/media/files/yolov3.weights
python convert.py yolov3.cfg yolov3.weights 320 model_data/yolov3.h5
```

2. 读取转换后模型的参数

```python
def get_output_names(keras_model):
    outputs = keras_model.outputs
    output_names = []
    for _output in outputs:
        output_names.append(_output.name[:-2])
    return output_names


def get_input_names(keras_model):
    inputs = keras_model.inputs
    input_names = []
    for _input in inputs:
        input_names.append(_input.name[:-2])
    return input_names
``` 

3. 使用官方提供的`lite.TFLiteConverter.from_keras_model_file`转换模型

```python
def keras_to_tflite(input_keras_model_file, output_tflite_file):
    keras_model = load_model(input_keras_model_file)
    input_names = get_input_names(keras_model)
    output_names = get_output_names(keras_model)
    print("input names:", input_names, " shapes:", keras_model.input_shape)
    print("output names:", output_names, " shapes:", keras_model.output_shape)
    converter = lite.TFLiteConverter.from_keras_model_file(model_file=input_keras_model_file,
                                                           input_arrays=input_names,
                                                           output_arrays=output_names)
    model = converter.convert()
    file = open(output_tflite_file, "wb")
    file.write(model)
    print("save tflite file to: ", output_tflite_file)
```

4. 获取转换后的模型

```python
def main():
    keras_model_file = "./model_data/yolov3.h5"
    tflite_model_file = "./model_data/yolov3.tflite"
    keras_to_tflite(keras_model_file, tflite_model_file)


if __name__ == '__main__':
    main()
```

## 预告
如何在PC端验证测试转换后的TFLite模型
