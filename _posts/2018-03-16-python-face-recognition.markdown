# 실시간 webcam face recognition

다음 그림을 보면 동영상에 나온 사람의 이름을 실시간으로 알아냅니다. 이것은 face recognition 기술입니다.

![using OpenFace](https://cdn-images-1.medium.com/max/800/1*IHy4EB25kpO9Dh_ugCJp7Q.gif)

이미지 출처: [여기](https://medium.com/@jongdae.lim/%EA%B8%B0%EA%B3%84-%ED%95%99%EC%8A%B5-machine-learning-%EC%9D%80-%EC%A6%90%EA%B2%81%EB%8B%A4-part-4-63ed781eee3c)

위 이미지 출처 사이트에 가 보면 Face recognition에 대한 이론적인 설명이 잘 나와 있고, OpenFace 패키지를 이용하여 face recognition을 구현하고 있습니다.
마치 밥 로스 아저씨의 "참 쉽죠" 같죠?


그런데, 이것보다 더 쉽게 구현할 수 있는 방법이 있습니다. Python의 OpenCV와 face_recognition 패키지를 사용하는 방법입니다. 아래 화면처럼요.

![using face_recognition](https://cloud.githubusercontent.com/assets/896692/24430398/36f0e3f0-13cb-11e7-8258-4d0c9ce1e419.gif)

여기에서는 face_recognition을 이용하여 구현하는 방법에 대해서 설명하겠습니다.

이 예제를 실행하려면 webcam이 연결된 PC가 필요합니다. 노트북에 내장된  webcam도 잘 동작합니다. 이 예제는 Ubuntu 14.04 linux와 Windows 10에서 테스트 되었습니다만, 다른 버전의 linux나 MacOS에서도 잘 동작할 것입니다.

# 필요 패키지 설치

## Linux

Linux를 사용한다면, 패키지를 설치하기 전에 아래와 같이 virtualenv로 환경을 분리시키기를 추천합니다.

```bash
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

## Windows

Windows 에서는 먼저 [https://www.python.org/downloads/](https://www.python.org/downloads/) 에서 Python 3을 64버전으로 설치합니다. 그 다음, 커맨드창에서 다음과 같이 필요한 패키지를 설치합니다.

```
C:\> pip install opencv-python
C:\> pip install opencv-contrib-python
C:\> pip install dlib
C:\> pip install face_recognition
C:\> pip install flask
```

Windows에서는 dlib 설치시 에러가 발생할 수 있습니다. 이것을 해결하기 위해서 아래 사이트에 접속하여 가장 최신 버전의 *-win_amd64.whl 파일을 다운받습니다. 

https://pypi.python.org/simple/dlib/ 

이 글을 쓰는 시점에는 dlib-19.8.1-cp36-cp36m-win_amd64.whl 파일이 가장 최신 파일이었습니다.
그 다음, 아래 명령으로 파일을 설치합니다.

```
C:\> pip install dlib-19.8.1-cp36-cp36m-win_amd64.whl
```

(이 방법은 https://github.com/charlielito/install-dlib-python-windows 를 참고했습니다.)

# 실행

knowns 디렉토리에 샘플 사진을 복사합니다. 샘플 사진에는 한 사람의 얼굴만 들어 있어야 합니다.
파일 이름은 "사람이름.jpg"으로 합니다. 파일 이름은 비디오상에 출력되는 이름으로 사용됩니다.

```
$ ls knowns/
iu.jpg  Lenna.jpg  uain.jpg
```

그리고, OS 별로 아래와 같이 명령을 내립니다.

터미널에서 아래 명령으로 스크립트를 실행시킵니다.  리눅스의 경우에는 먼저 virtualenv를 활성화시키는 것을 잊지 마세요.

```
(py3) $ python camera.py             # 카메라가 정상적으로 동작하는지 검사하기 위해 실행
(py3) $ python face_recog.py         # knowns 디렉토리에 있는 얼굴을 감지하여 출력
(py3) $ python live_streaming.py     # 비디오를 네트워크 상으로 전송. 웹브라우저에서 http://IP_addr:5000 으로 접속하여 확인
```

얼굴 아래에 이름이 잘 표시 되지요?


# 소스코드 설명

Python 소스코드는 크게 세 파일로 이루어져 있습니다.

* camera.py - OpenCV를 이용하여 camera 영상을 캡처함.
* face_recog.py - face_recognition 패키지를 이용하여 얼굴을 분류함.
* live_streaming.py - mjpeg으로 스트리밍



Face recognition은 아래 사이트 참조
https://github.com/ageitgey/face_recognition

Streaming은 아래 사이트 참조
http://www.chioka.in/python-live-video-streaming-example/




