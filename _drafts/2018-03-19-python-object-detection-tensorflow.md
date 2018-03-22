---
title:  "Python Object Detection with Tensorflow"
#categories: python face-recognition
tags: python object-detection tensorflow

---

## Install

Refer to [this site][install].

### install tensorflow
- https://www.tensorflow.org/install/

### install necessary libraries and packages

```
$ sudo apt-get install protobuf-compiler python-pil python-lxml python-tk
(py3) $ pip install jupyter
(py3) $ pip install matplotlib
(py3) $ pip install Pillow
```

### clone TensorFlow Models

Assuming to clone to ~/work/tensorflow directory.

```
$ mkdir -p ~/work/tensorflow
$ cd ~/work/tensorflow
$ git clone https://github.com/tensorflow/models
```

### install COCO API (optional)

```
(py3) $ pip install cython
$ git clone https://github.com/cocodataset/cocoapi.git
$ cd cocoapi/PythonAPI
$ make
$ cp -r pycocotools ~/work/tensorflow/models/research/
```

### Protobuf Compilation

```
$ cd ~/work/tensorflow/models/research
$ protoc object_detection/protos/*.proto --python_out=.
```
이렇게 하면 ~/work/tensorflow/models/research/object_detection/protos 디렉토리에 box.predictor_pb2.py, faster_rcnn_box_coder_pb2.py 등의 파일이 생성되는 듯...

### Add Libraries to PYTHONPATH

```
(py3) $ cd ~/work/tensorflow/models/research
(py3) $ export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
```
You have to do this on every terminal.

### Testing the Installation

```
(py3) $ cd ~/work/tensorflow/models/research
(py3) $ python object_detection/builders/model_builder_test.py
```

## Let's try!!!

### Using Jupyter

```
(py3) $ cd ~/work/tensorflow/models/research
(py3) $ jupyter notebook
```

localhost:8888/tree is opened on web browser. Open object_detection/object_detection_tutorial.ipynb

ElephantHunters의 예제:

```
(py3) $ cd ~/work
(py3) $ git clone https://github.com/ElephantHunters/Real_time_object_detection_using_tensorflow
(py3) $ cp Real_time_object_detection_using_tensorflow/object_detection_tutorial_Webcam_1.ipynb ~/work/tensorflow/models/research/object_detection
```

On jupyter notebook, open object_detection/object_detection_tutorial_Webcam_1.ipynb. Change the filename in [7]. Then run.

## 참고 사이트

* [Whole install procedure of Tensorflow object detection model][install]
* [Tensorflow object detection model zoo][modelzoo]
* [Real time object detection with Tensorflow][objexample1]
* [Using Tensorflow Object Detection to do Pixel Wise Classification][objexample2]
* [Is Google Tensorflow Object Detection API the easiest way to implement image recognition?][maskexample]

[install]:https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md
[modelzoo]: https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md
[objexample1]:https://towardsdatascience.com/real-time-object-detection-with-tensorflow-detection-model-e7fd20421d5d
[objexample2]: https://towardsdatascience.com/using-tensorflow-object-detection-to-do-pixel-wise-classification-702bf2605182
[maskexample]: https://towardsdatascience.com/is-google-tensorflow-object-detection-api-the-easiest-way-to-implement-image-recognition-a8bd1f500ea0
