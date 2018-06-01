---
title:  "Facial Landmarks"
date:   2018-04-02 21:51:00 +0900
tags:   python face-recognition
feature-img: "assets/img/pexels/model-face-beautiful-black-and-white-407035.jpeg"
layout: post
---

며칠 전에 [Python Face Recognition][python-face-recognition]에 대한 글을 올렸습니다. Face recognition은 다음과 같은 과정을 거칩니다. 이 중에서, 2번째 단계인 얼굴의 특징점을 추출하는 단계는 아주 재미있습니다.

1. 이미지에서 얼굴만 분리
1. 얼굴의 특징점 (facial landmarks) 추출
1. 얼굴 위치 교정과 투영
1. 교정된 얼굴을 인코딩
1. 기존 데이터베이스와 비교하여 가장 비슷한 얼굴 찾기

인터넷 상에 아래 그림과 같이 [웹캠 영상에서 실시간으로 얼굴의 특징점을 찾아내는 재미있는 예제][facial-landmarks]가 있어 소개하고자 합니다.

![realtime-facial-landmarks](https://www.pyimagesearch.com/wp-content/uploads/2017/04/realtime_facial_landmarks_demo.gif)

[사이트][facial-landmarks]에 방문해 보세요. dlib 패키지를 이용하여 구현된 파이썬 소스코드에 대한 설명이 잘 되어 있습니다. e-mail 주소를 적으면 소스코드를 다운받을 수 있습니다.

소스코드를 받으신 후, 다음 명령을 내리면 웹캠 영상에 실시간으로 얼굴의 특징점이 나타납니다. 직접 해 보시면 더 재미있습니다.

```
(py3) $ python video_facial_landmarks.py --shape-predictor shape_predictor_68_face_landmarks.dat
```

이 예제에서 추출하는 68개의 특징점은 아래 그림과 같습니다. (이 그림은 [여기][media_com]에서 가져왔습니다.)

![68-facial-landmarks](https://cdn-images-1.medium.com/max/800/1*96UT-D8uSXjlnyvs9DZTog.png)


이것을 응용하면 여러가지 재미있는 애플리케이션을 만들 수 있습니다.

예를 들어, 점 36과 39, 37과 41, 38과 40의 거리를 측정하면 눈의 좌우와 위아래의 비율을 측정할 수 있습니다. 좌우 거리에 비해 위 아래 거리가 너무 짧고, 그 시간이 일정시간 지속되면 그 사람은 자고 있다고 판단할 수 있습니다. 이 것이 졸음 측정의 원리 입니다.

입 주위 점의 위 아래 거리를 측정하면 입을 벌리고 있는지 다물고 있는지 알아낼 수 있죠.

## 참고 사이트

* [Real-time facial landmark detection with OpenCV, Python, and dlib][facial-landmarks]
* [딥러닝(Deep Learning)을 사용한 최신 얼굴 인식(Face Recognition)][media_com]

[python-face-recognition]: {{ site.url }}/python-face-recognition
[facial-landmarks]: https://www.pyimagesearch.com/2017/04/17/real-time-facial-landmark-detection-opencv-python-dlib/
[media_com]: https://medium.com/@jongdae.lim/%EA%B8%B0%EA%B3%84-%ED%95%99%EC%8A%B5-machine-learning-%EC%9D%80-%EC%A6%90%EA%B2%81%EB%8B%A4-part-4-63ed781eee3c
