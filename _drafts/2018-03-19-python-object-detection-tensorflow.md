---
title:  "Python Object Detection with Tensorflow"
date:   2018-03-27 21:30:34 +0900
#categories: python face-recognition
tags: python object-detection tensorflow
---

## 구글 텐서플로 object detection 모델

구글은 텐서플로로 구현된 많은 [모델](https://github.com/tensorflow/models)을 [아파치 라이센스](https://github.com/tensorflow/models/blob/master/LICENSE)로 공개하고 있습니다.
그 중에서 [object detection](https://github.com/tensorflow/models/tree/master/research/object_detection) 모델은 사진에서 물체를 인식하는 모델입니다. 이것은 매우 활발히 연구되고 빠르게 발전하는 모델로서, 글을 쓰는 현재 구글은 19개의 [pre-trained object detection model][modelzoo]을 공개했으며, 점점 더 많은 모델이 구현되고 공개될 것입니다.

이 모델 API를 이용하면 아래 그림과 같이 웹캠을 이용한 실시간 사물 인식을 쉽게 구현할 수 있습니다. 아래 그림은 [여기][objexample2]에서 가져왔습니다.

![object-detection-example](https://cdn-images-1.medium.com/max/800/1*W3elu1yPiJ3bpj8MZrmvwA.gif)

## 필요 패키지 설치

파이썬과 텐서플로는 이미 설치되어 있다고 가정하겠습니다. 텐서플로 설치는 [여기](https://www.tensorflow.org/install/)를 참조하시면 됩니다.

이 설치 방법은 [여기][install]를 참고하였습니다. 이 글을 쓰는 시점 이후에 설치 방법이 바뀔 수도 있으니, 아래 방법이 잘 안 될 경우에는 [원본 사이트][install]를 방문하시기 바랍니다.


### Install Necessary Libraries and Packages

아래 명령을 입력하여 필요한 라이브러리와 파이썬 패키지를 설치합니다. 파이썬 패키지는 py3 이름의 virtualenv상에 설치하는 것으로 가정합니다.

```
$ sudo apt-get install protobuf-compiler python-pil python-lxml python-tk
(py3) $ pip install jupyter
(py3) $ pip install matplotlib
(py3) $ pip install Pillow
```

### Clone TensorFlow Models

그 다음, GitHub로부터 텐서플로 모델을 복사합니다. 여기서는 ~/work/tensorflow 디렉토리에 복사하는 것으로 가정합니다.

```
$ mkdir -p ~/work/tensorflow
$ cd ~/work/tensorflow
$ git clone https://github.com/tensorflow/models
```

### Install COCO API (optional)

이 부분은 COCO API를 설치하는 것인데, 건너 뛰어도 좋습니다.

```
(py3) $ pip install cython
$ git clone https://github.com/cocodataset/cocoapi.git
$ cd cocoapi/PythonAPI
$ make
$ cp -r pycocotools ~/work/tensorflow/models/research/
```

### Protobuf Compilation

Protobuf를 라이브러리를 컴파일합니다.

```
$ cd ~/work/tensorflow/models/research
$ protoc object_detection/protos/*.proto --python_out=.
```
이렇게 하면 ~/work/tensorflow/models/research/object_detection/protos 디렉토리에 box.predictor_pb2.py, faster_rcnn_box_coder_pb2.py 등의 파일이 생성됩니다.

### Add Libraries to PYTHONPATH

```
(py3) $ cd ~/work/tensorflow/models/research
(py3) $ export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
```
이 작업은 PYTHONPATH 환경 변수에 모델 디렉토리를 추가하는 것으로, shell이 생성될 때 마다 해 주어야 합니다. 또는 .bashrc에 넣어 두면 shell이 생성될 때 마다 자동으로 실행되니 편합니다.

### Testing the Installation

아래 명령으로 모델이 잘 설치되었는지 검사합니다. 맨 마지막에 OK라는 글자가 나오면 됩니다.

```
(py3) $ cd ~/work/tensorflow/models/research
(py3) $ python object_detection/builders/model_builder_test.py
.............
----------------------------------------------------------------------
Ran 13 tests in 0.125s

OK
(py3) $
```

이제 object detection 모델이 설치가 완료되었습니다.

### Let's try!!!

설치가 완료 되었으니, 한 번 맛을 볼까요? 아래 명령으로 jupyter notebook을 실행시킵니다.

```
(py3) $ cd ~/work/tensorflow/models/research
(py3) $ jupyter notebook
```
웹브라우저에서 localhost:8888/tree 가 열릴 것입니다. object_detection 디렉토리의 object_detection_tutorial.ipynb를 실행시켜 보세요. 아래 그림과 같이, 두 마리의 개가 인식이 되었나요?

![obj-detection-install-test]({{ site.url }}/assets/obj-detection-install-test.png)


## ElephantHunters의 예제:

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
* [Is Google Tensorflow Object Detection API the easiest way to implement image recognition?][objexample2]
* [Using Tensorflow Object Detection to do Pixel Wise Classification][maskexample]

[install]:https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md
[modelzoo]: https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md
[objexample1]:https://towardsdatascience.com/real-time-object-detection-with-tensorflow-detection-model-e7fd20421d5d
[objexample2]: https://towardsdatascience.com/is-google-tensorflow-object-detection-api-the-easiest-way-to-implement-image-recognition-a8bd1f500ea0
[maskexample]: https://towardsdatascience.com/using-tensorflow-object-detection-to-do-pixel-wise-classification-702bf2605182
