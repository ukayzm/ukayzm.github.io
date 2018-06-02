---
title:  "Tensorflow Instance Segmentation"
tags:   tensorflow object-detection
feature-img: "assets/img/posts/instance-segmentation-road.png"
thumbnail:   "assets/img/posts/instance-segmentation-road.png"
layout: post
---

Object detection 모델을 돌리면 object가 인식된 사각형 영역을 얻을 수 있습니다. Instance Segmentation은 이것을 확장하여 object가 존재하는 영역의 mask까지 얻어내는 것입니다. 구글은 텐서플로우로 만들어진 instance segmentation 모델을 공개하고 있습니다. 이 글에서는 Windows에서 동작하는 object detection과 instance segmentation 프로그램을 소개하겠습니다.

소스코드는 [https://github.com/ukayzm/opencv/tree/master/object_detection_tensorflow](https://github.com/ukayzm/opencv/tree/master/object_detection_tensorflow) 여기에서 다운로드 받을 수 있습니다.

## Model

구글이 공개한 instance segmentation 모델은 Mask R-CNN이라고 하는 것으로, Faster R-CNN과 FCN(Fully Convolution Network)을 결합한 것입니다. Faster R-CNN은 object detection의 역할을 하고, FCN은 mask를 얻는 역할을 합니다. 더 자세한 설명은 아래 사이트를 참고하세요.
[https://towardsdatascience.com/using-tensorflow-object-detection-to-do-pixel-wise-classification-702bf2605182](https://towardsdatascience.com/using-tensorflow-object-detection-to-do-pixel-wise-classification-702bf2605182)

## 비디오에 적용한 예

아래 비디오는 차량용 블랙박스에서 얻은 영상에 instance segmentation을 입힌 영상입니다. 이 글에서 소개하는 방식으로 실시간으로 동작시킨 것은 아니지만, 원리는 동일합니다. 꽤 그럴 듯 하지요?

<iframe width="1280" height="480" src="https://www.youtube.com/embed/PeNAT0dMw6g?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## Tensorflow 모델 설치

[{{ site.url }}/python-object-detection-tensorflow]({{ site.url }}/python-object-detection-tensorflow)에서 Linux에 Tensorflow model을 설치하는 방법을 설명했었는데요. Windows에 설치하는 것도 거의 비슷합니다. 여기에서는 python3가 이미 설치되어 있다고 가정하고 간략하게 설명하겠습니다. 이 설치 방법은 [https://medium.com/@rohitrpatil/how-to-use-tensorflow-object-detection-api-on-windows-102ec8097699](https://medium.com/@rohitrpatil/how-to-use-tensorflow-object-detection-api-on-windows-102ec8097699)를 참고하였습니다.

### Tensorflow와 필요 패키지 설치

```
C:\> pip install --upgrade tensorflow
C:\> pip install pillow
C:\> pip install lxml
C:\> pip install jupyter
C:\> pip install matplotlib
```

### Model download

```
C:\> mkdir tensorflow
C:\> cd tensorflow
C:\tensorflow> git clone https://github.com/tensorflow/models.git  
```

커맨드 라인에서 실행되는 git 대신에 [GitHub Desktop](https://desktop.github.com/) 같은 툴을 사용해도 됩니다.

### Download protoc & compile protobuf

[https://github.com/google/protobuf/releases](https://github.com/google/protobuf/releases)에서 protoc를 다운로드 받습니다. 참고로, 이 글을 쓰는 시점에서는 protoc-3.5.1가 최신 버전이었는데, 이 버전은 컴파일시 에러가 발생해서, protoc-3.4.0-win32.zip를 다운받았습니다.
다운받은 protoc의 압축을 풉니다. 여기에서는 \tensorflow\models\research 디렉토리에 풀었다고 가정합니다.
그 다음,
```
C:\tensorflow\models\research> protoc-3.4.0-win32\bin\protoc.exe object_detection/protos/*.proto --python_out=.
```
를 하면 object_detection\protos 디렉토리에 \*.py 파일이 생성됩니다.

### PYTHONPATH 환경 변수 설정

작업표시줄의 윈도우 버튼에서 오른쪽 버튼 > 시스템 > 고급 시스템 설정 > 고급 탭 > 환경변수 를 누릅니다.
새로 만들기를 눌러 PYTHONPATH 변수를 만들고, 아래 값을 입력합니다.
```
C:\tensorflow\models\research;C:\tensorflow\models\research\slim
```

## Example code

구글은 object detection API를 사용하는 예제 코드도 같이 공개하고 있습니다. 그 예제코드를 변형하여, webcam으로부터 실시간으로 object detection이나 instance segmentation을 하는 예제 코드를 만들었습니다.

[https://github.com/ukayzm/opencv/tree/master/object_detection_tensorflow](https://github.com/ukayzm/opencv/tree/master/object_detection_tensorflow) 여기에 방문하여 소스코드를 다운로드 받아 실행해 보세요.

예제 코드는 세 파일로 되어 있습니다.
* camera.py - webcam 테스트
* object_detector.py - object detection 또는 instance segmentation 실행
* live_streaming.py - 영상을 네트워크 상으로 전송. 웹브라우저에서 http://IP_addr:5000 으로 접속하여 확인


## References

* [https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/instance_segmentation.md](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/instance_segmentation.md)
* [https://towardsdatascience.com/using-tensorflow-object-detection-to-do-pixel-wise-classification-702bf2605182](https://towardsdatascience.com/using-tensorflow-object-detection-to-do-pixel-wise-classification-702bf2605182)
* [https://stackoverflow.com/questions/33947823/what-is-semantic-segmentation-compared-to-segmentation-and-scene-labeling](https://stackoverflow.com/questions/33947823/what-is-semantic-segmentation-compared-to-segmentation-and-scene-labeling)
* [https://medium.com/@rohitrpatil/how-to-use-tensorflow-object-detection-api-on-windows-102ec8097699](https://medium.com/@rohitrpatil/how-to-use-tensorflow-object-detection-api-on-windows-102ec8097699)
