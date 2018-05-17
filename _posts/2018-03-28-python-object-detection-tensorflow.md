---
title:  "Python Object Detection with Tensorflow"
date:   2018-03-28 21:15:00 +0900
tags: python object-detection tensorflow
feature-img: "assets/img/posts/object-detection-kite.jpg"
thumbnail:   "assets/img/posts/object-detection-kite.jpg"
layout: post
---

구글은 텐서플로로 구현된 많은 [모델](https://github.com/tensorflow/models)을 [아파치 라이센스](https://github.com/tensorflow/models/blob/master/LICENSE)로 공개하고 있습니다.
그 중에서 [object detection API](https://github.com/tensorflow/models/tree/master/research/object_detection) 사진에서 물체를 인식하는 모델을 쉽게 제작/학습/배포할 수 있는 오픈소스 프레임워크 입니다. 사물 인식은 매우 활발히 연구되고 빠르게 발전하는 모델로서, 글을 쓰는 현재 구글은 19개의 [pre-trained object detection model][modelzoo]을 공개했으며, 점점 더 많은 모델이 구현되고 공개될 것입니다.

이 API를 이용하면 아래 그림과 같이 웹캠을 이용한 실시간 사물 인식을 쉽게 구현할 수 있습니다. 아래 그림은 [여기][objexample2]에서 가져왔습니다.

![object-detection-example](https://cdn-images-1.medium.com/max/800/1*W3elu1yPiJ3bpj8MZrmvwA.gif)

## 필요 패키지 설치

파이썬과 텐서플로는 이미 설치되어 있다고 가정하겠습니다. 텐서플로 설치는 [여기](https://www.tensorflow.org/install/)를 참조하시면 됩니다.

이 설치 방법은 [여기][install]를 참고하였습니다. 이 글을 쓰는 시점 이후에 설치 방법이 바뀔 수도 있으니, 아래 방법이 잘 안될 경우에는 [원본 사이트][install]를 방문하시기 바랍니다.


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

간혹, old version의 protoc를 사용하면 아래와 같은 에러가 나는데요.

```
$ protoc object_detection/protos/*.proto --python_out=.
object_detection/protos/anchor_generator.proto:12:3: Expected "required", "optional", or "repeated".
object_detection/protos/anchor_generator.proto:12:32: Missing field number.
```

이 경우에는 protoc 3.3 (or higher)를 사용하면 됩니다. 아래와 같이 protoc를 다운받아 압축을 풀고 실행시키세요.
[https://github.com/tensorflow/models/issues/1834](https://github.com/tensorflow/models/issues/1834) 여기를 참고하세요.

```
$ wget https://github.com/google/protobuf/releases/download/v3.3.0/protoc-3.3.0-linux-x86_64.zip
$ unzip protoc-3.3.0-linux-x86_64.zip -d protoc-3.3.0
$ protoc-3.3.0/bin/protoc object_detection/protos/*.proto --python_out=.
```

### Add Libraries to PYTHONPATH

```
(py3) $ cd ~/work/tensorflow/models/research
(py3) $ export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim:`pwd`/object_detection
```
이 작업은 PYTHONPATH 환경 변수에 모델 디렉토리를 추가하는 것으로, shell이 생성될 때 마다 해 주어야 합니다. 또는 .bashrc에 아래 라인을 넣어 두면 shell이 생성될 때 마다 자동으로 실행되니 편합니다.

```
export PYTHONPATH=$PYTHONPATH:~/work/tensorflow/models/research/:~/work/tensorflow/models/research/slim:~/work/tensorflow/models/research/object_detection/
```

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
웹브라우저에서 localhost:8888/tree 가 열릴 것입니다. object_detection 디렉토리의 object_detection_tutorial.ipynb를 실행시켜 보세요. 아래 그림과 같이, 두 마리의 개가 인식이 되었나요? 성공입니다.

![obj-detection-install-test]({{ site.url }}/assets/img/posts/obj-detection-install-test.png)


## 웹캠을 이용한 실시간 사물 인식

인터넷 상에 [웹캠을 이용하여 실시간으로 사물을 인식할 수 있는 좋은 예제](objexample1)가 있어 소개를 하겠습니다. 아래과 같이 코드를 다운 받고, ipynb 파일을 object_detection 디렉토리로 복사한 후, Jupyter notebook을 실행시킵니다.

```
(py3) $ cd ~/work
(py3) $ git clone https://github.com/ElephantHunters/Real_time_object_detection_using_tensorflow
(py3) $ cp Real_time_object_detection_using_tensorflow/object_detection_tutorial_Webcam_1.ipynb ~/work/tensorflow/models/research/object_detection
(py3) $ cd /work/tensorflow/models/research/object_detection
(py3) $ jupyter notebook
```

웹브라우저에서 Jupyter notebook이 뜨면, object_detection_tutorial_Webcam_1.ipynb 파일을 열고, 아래 그림의 \[7\] 셀에서 filename을 적당히 수정합니다.

![change-filename]({{ site.url }}/assets/img/posts/change_filename.png)

```
filename="output.avi"
```

이렇게 수정한 후, 노트북을 실행시켜 보세요. 잠시 후 웹캠 화면이 뜨고, 화면에서 인식된 물체에 사각형이 그려지고 물체의 이름이 뜰 것입니다.

아래와 같이 notebook 파일을 파이썬 스크립트로 변환시켜 실행시켜도 잘 동작합니다. 이 때, 변환된 파이썬 코드에서 get_ipython()으로 시작하는 노트북 코드를 주석처리해야 합니다.

```
(py3) $ cd /work/tensorflow/models/research/object_detection
(py3) $ jupyter nbconvert --to script object_detection_tutorial_Webcam_1.ipynb
(py3) $ vi object_detection_tutorial_Webcam_1.py
# This is needed to display the images.
# get_ipython().run_line_magic('matplotlib', 'inline')
# 윗 줄처럼 파이썬 코드 중에서 get_ipython() 부분을 주석처리하세요.
(py3) $ python object_detection_tutorial_Webcam_1.py
```

어떻습니까? 잘 실행이 되나요? 그렇다면 성공입니다.

## 참고 사이트

* [Whole install procedure of Tensorflow object detection model][install]
* [Tensorflow object detection model zoo][modelzoo]
* [Real time object detection with Tensorflow][objexample1]
* [Is Google Tensorflow Object Detection API the easiest way to implement image recognition?][objexample2]

[install]:https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md
[modelzoo]: https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md
[objexample1]:https://towardsdatascience.com/real-time-object-detection-with-tensorflow-detection-model-e7fd20421d5d
[objexample2]: https://towardsdatascience.com/is-google-tensorflow-object-detection-api-the-easiest-way-to-implement-image-recognition-a8bd1f500ea0
[maskexample]: https://towardsdatascience.com/using-tensorflow-object-detection-to-do-pixel-wise-classification-702bf2605182
