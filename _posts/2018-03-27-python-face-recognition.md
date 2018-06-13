---
title:  "Python Face Recognition in Real Time"
date:   2018-03-27 21:30:34 +0900
tags: face-recognition
thumbnail:   "assets/img/thumbnails/face_recognition_capture.png"
layout: post
---

사진에서 사람 얼굴을 인식하는 face_recognition이라는, 아주 쓰기 쉬운 파이썬 패키지가 있습니다. 이 패키지를 이용하면 웹캠을 이용하여 실시간으로 사람 얼굴을 인식하는 프로그램을 쉽게 제작할 수 있습니다. 파이썬을 설치하고, 필요한 패키지를 설치하고 소스코드를 다운 받고, knowns 디렉토리에 사람 얼굴이 있는 사진을 넣으면 동작합니다. 바로 아래 화면처럼요.

![face-recognition-example-picture](https://cloud.githubusercontent.com/assets/896692/24430398/36f0e3f0-13cb-11e7-8258-4d0c9ce1e419.gif)

Face_recognition 패키지는 [https://github.com/ageitgey/face_recognition](https://github.com/ageitgey/face_recognition)에서 운영되고 있습니다.

얼굴 인식 프로그램을 실행하려면 webcam이 연결된 PC가 필요합니다. 노트북에 내장된  webcam도 잘 동작합니다. 이 예제는 Ubuntu 14.04 linux와 Windows 10에서 테스트 되었습니다만, 다른 버전의 linux나 MacOS에서도 잘 동작할 것입니다.


## 필요 패키지 설치

### Linux

Linux를 사용한다면, 패키지를 설치하기 전에 아래와 같이 virtualenv로 환경을 분리시키기를 추천합니다.

```
$ sudo apt-get install python3 python3-dev python3-venv
$ python3 -m venv py3
$ source py3/bin/activate
(py3) $ pip install --upgrade pip
```

그 다음, 아래와 같이 필요한 패키지를 설치합니다.

```
(py3) $ pip install opencv-python
(py3) $ pip install opencv-contrib-python
(py3) $ pip install dlib
(py3) $ pip install face_recognition
(py3) $ pip install flask
```

Flask 패키지는 face recognition과 직접적인 관련은 없지만, 동영상을 스트리밍하기 위해 설치하는 것입니다.

### Windows

Windows 에서는 먼저 [https://www.python.org/downloads/](https://www.python.org/downloads/) 에서 Python 3을 64버전으로 설치합니다. 그 다음, 커맨드창에서 다음과 같이 필요한 패키지를 설치합니다.

```
C:\> pip install opencv-python
C:\> pip install opencv-contrib-python
C:\> pip install dlib
C:\> pip install face_recognition
C:\> pip install flask
```

### Windows에서 dlib 설치

Windows에서 dlib 설치시 에러가 발생할 수 있습니다. 이것을 해결하기 위해서 [https://pypi.python.org/simple/dlib](https://pypi.python.org/simple/dlib)에 접속하여 가장 최신 버전의 xxx-win_amd64.whl 파일을 다운받습니다.

이 글을 쓰는 시점에는 dlib-19.8.1-cp36-cp36m-win_amd64.whl 파일이 가장 최신 파일이었습니다.
그 다음, 아래 명령으로 파일을 설치합니다.

```
C:\> pip install dlib-19.8.1-cp36-cp36m-win_amd64.whl
```

(이 방법은 [https://github.com/charlielito/install-dlib-python-windows](https://github.com/charlielito/install-dlib-python-windows) 를 참고했습니다.)

## 코드 다운로드와 실행

먼저, [https://github.com/ukayzm/opencv/tree/master/face_recognition](https://github.com/ukayzm/opencv/tree/master/face_recognition) 또는 다음 zip 파일을 다운로드 받습니다. [face_recognition.zip][face_recognition.zip]
압축을 풀면 아래와 같은 파일과 디렉토리가 생성됩니다.

```
$ tree
.
├── camera.py
├── face_recog.py
├── knowns
├── live_streaming.py
└── templates
    └── index.html
```

knowns 디렉토리에 아래와 같이 주변 사람의 사진을 복사합니다.

```
$ ls knowns/
HyoRi.jpg  Lenna.jpg  DongGun.jpg
```

* 사진에는 한 사람의 얼굴만 들어 있어야 합니다.
* 파일 이름은 "사람이름.jpg"으로 합니다. 파일 이름은 비디오상에 출력되는 이름으로 사용됩니다.
* 스마트폰으로 찍은 사진을 사용하는 경우에는, 회전 속성이 들어있지 않은 사진을 사용해야 합니다. 얼굴 인식이 잘 되지 않을 때는, 사진을 그림판에서 읽은 후, 90/180도 좌/우로 회전 시킨 다음 저장하세요.
* 사진에서 얼굴이 인식되지 않으면 에러가 발생합니다.

zip 파일에는 3개의 파이썬 파일이 들어 있습니다. 각 파이썬 파일의 기능은 아래와 같습니다.

* camera.py - 웹캠의 동영상을 모니터로 출력합니다. 카메라가 정상적으로 동작하는지 검사하기 위해 실행합니다.
* face_recog.py - 웹캠 동영상에 있는 얼굴을 감지하여 knowns 디렉토리에 있는 얼굴과 비교하고 감지되는 이름을 출력합니다.
* live_streaming.py - 위 동영상을 네트워크 상으로 전송합니다. 파이썬이 실행되는 머신에 모니터가 달려있지 않은 경우에 사용할 수 있습니다. 임의의 PC의 웹브라우저에서 http://IP_addr:5000 으로 접속이 가능합니다.

터미널에서 아래 명령으로 각 파이썬 파일을 하나씩 실행시켜 보세요.  리눅스의 경우에는 먼저 virtualenv를 활성화시키는 것을 잊지 마세요.

```
(py3) $ python camera.py             # 카메라가 정상적으로 동작하는지 검사하기 위해 실행
(py3) $ python face_recog.py         # knowns 디렉토리에 있는 얼굴을 감지하여 출력
(py3) $ python live_streaming.py     # 비디오를 네트워크 상으로 전송. 웹브라우저에서 http://IP_addr:5000 으로 접속하여 확인
```

얼굴 아래에 이름이 잘 표시 되나요? Unknown이라고 나오면 jpg 파일이 회전되어 있는 것은 아닌지 그림판으로 열어보세요.

윈도우에서 'q' 키를 누르거나 터미널에서 ^C를 누르면 종료됩니다.

## 소스코드 설명


얼굴 인식의 핵심 역할을 하는 face_recog.py 파일을 좀 더 깊게 알아보겠습니다.

face_recog.py는 많은 머신 러닝 알고리즘이 구현되어 있는 dlib와 그것을 얼굴 인식 기능에 촛점을 맞춘 wrapper인 face_recognition 패키지를 이용하여 구현되어 있습니다.

<script src="https://gist.github.com/ukayzm/05363345935cabe958627d75a113c825.js"></script>

### Line 20-25
knowns 디렉토리에서 사진 파일을 읽습니다. 파일 이름으로부터 사람 이름을 추출합니다.

### Line 27-29
사진에서 얼굴 영역을 알아내고, face landmarks라 불리는 68개 얼굴 특징의 위치를 분석한 데이터를 known_face_encodings에 저장합니다. 이 작업의 원리는 [이 사이트][media_com]에 잘 설명되어 있습니다. 아주 쉽게 설명되어 있으므로, 꼭 한 번 읽어보시길 강력 추천 드립니다.

### Line 42-45
카메라로부터 frame을 읽어서 1/4 크기로 줄입니다. 이것은 계산양을 줄이기 위해서 입니다.

### Line 51
계산 양을 더 줄이기 위해서 두 frame당 1번씩만 계산합니다.

### Line 53-54
읽은 frame에서 얼굴 영역과 특징을 추출합니다.

### Line 59-60
Frame에서 추출한 얼굴 특징과 knowns에 있던 사진 얼굴의 특징을 비교하여, (얼마나 비슷한지) 거리 척도로 환산합니다. 거리(distance)가 가깝다는 (작다는) 것은 서로 비슷한 얼굴이라는 의미 입니다.

### Line 64-65
실험상, 거리가 0.6 이면 다른 사람의 얼굴입니다. 이런 경우의 이름은 Unknown 입니다.

### Line 65-67
거리가 0.6 이하이고, 최소값을 가진 사람의 이름을 찾습니다.

### Line 74-87
찾은 사람의 얼굴 영역과 이름을 비디오 화면에 그립니다.

## 참고 사이트

* [딥러닝(Deep Learning)을 사용한 최신 얼굴 인식(Face Recognition)][media_com]
* [Face recognition python package](https://github.com/ageitgey/face_recognition)
* [Live streaming](http://www.chioka.in/python-live-video-streaming-example/)

## 소스 코드 다운로드

* [face_recognition.zip][face_recognition.zip]
* [https://github.com/ukayzm/opencv/tree/master/face_recognition](https://github.com/ukayzm/opencv/tree/master/face_recognition)

## See Also

Object detection 예제도 재미있습니다. [{{ site.url }}/tensorflow-instance-segmentation/]({{ site.url }}/tensorflow-instance-segmentation/)를 방문해 보세요.


[face_recognition.zip]: {{ site.url }}/assets/face_recognition.zip
[media_com]: https://medium.com/@jongdae.lim/%EA%B8%B0%EA%B3%84-%ED%95%99%EC%8A%B5-machine-learning-%EC%9D%80-%EC%A6%90%EA%B2%81%EB%8B%A4-part-4-63ed781eee3c
