---
title:  Unknown Face Classifier - 모르는 얼굴 분류하기
tags:   face-recognition
header:
  teaser: /assets/img/posts/unknown_face_classifier_title.png
  image: /assets/img/posts/unknown_face_classifier_title.png
date:   2020-08-16 10:00:00 +0900
sidebar:
  - nav: docs
---

이 포스트에서는 기존에 올렸던 얼굴 인식 구현인 [Python Face Recognition](/python-face-recognition/)을 개선합니다. 아는 사람의 이름을 표시하는 것 뿐 아니라, 모르는 얼굴을 모으고 비슷한 얼굴을 찾아내어 새로운 사람을 찾아내는 간단한 방법을 구현합니다. 소스코드는 [여기](https://github.com/ukayzm/opencv/tree/master/unknown_face_classifier)에서 받으실 수 있습니다.

# 얼굴 인식의 원리

얼굴 인식의 원리에 대해서는 이전에 포스트했던 [Python Face Recognition](/python-face-recognition/)나 [Python Face Clustering](/face-clustering/)에서 소개를 했습니다만, 여기에서 다시 한 번 설명을 하겠습니다.

## 얼굴 인식의 과정

얼굴을 인식하기 위해서는 다음과 같은 세 과정을 거칩니다.

{:refdef: style="text-align: center;"}
![Face Encoding](/assets/img/posts/face_encoding.png)
{: refdef}

1. 그림에서 얼굴이 있는 영역을 알아낸다. (face location)
2. 얼굴 영역에서 눈, 코, 입 등 68개의 주요 좌표를 추출한다. ([facial landmarks](/facial-landmarks/))
3. 68개의 좌표를 128개의 숫자로 변환한다. (face encoding)

파이썬 패키지 `dlib`에는 이 과정이 구현되어 있습니다. `face_recognition` 패키지의 `face_locations()`과 `face_encodings()` 함수는 `dlib`와 `numpy`에 쉽게 접근할 수 있도록 wrapping해 놓은 함수입니다.

위 과정에 대한 기술적인 내용은 이해하기 어려운 주제이므로, 여기에서는 face encoding의 결과물인 128개 숫자의 특성에 주목합시다.

* 128개 숫자는 deep learning의 결과물이기 때문에, 각 숫자의 의미는 알지 못합니다.
* 하지만 같은 사람의 얼굴을 입력하면 비슷한 숫자가 나옵니다.

## 얼굴 간의 거리 (유사성) 구하기

우리는 사전에 알고 있는 사람의 face_encoding 값을 가지고 있습니다. 그림에서 얼굴을 인식하고, 그 얼굴의 face_encoding을 사전에 알고 있는 사람의 face_encoding과 비교하면, 가장 거리가 가까운 (즉, 가장 비슷한) 사람을 찾아낼 수 있습니다.

face_encoding은 128 차원의 벡터 입니다. 이 벡터가 서로 얼마나 가까운지 어떻게 비교를 할까요? 수학적으로 [L2 norm (유클리드 거리)](https://ko.wikipedia.org/wiki/%EC%9C%A0%ED%81%B4%EB%A6%AC%EB%93%9C_%EA%B1%B0%EB%A6%AC)를 이용하면 두 벡터의 거리를 구할 수 있고, 파이썬에서는 `numpy`의 `linalg.norm()` 함수를 이용하면 됩니다. 

실제로 파이썬 패키지 `face_recognition`의 `face_distance()` 함수는 `numpy`의 `linalg.norm()` 함수의 wrapper로 구현되어 있습니다. 두 face_encoding의 거리는 0 ~ 1 사이의 값으로 구해지고, 이 값은 두 얼굴이 얼마나 비슷한지에 대한 척도로 사용할 수 있습니다.

## Similarity Threshold

이 때, 단순히 거리가 가장 가까운 사람만 찾는 것이 아니라, 사전에 정해진 기준 보다 가까워야 합니다. 기준보다 멀다면 "비슷하게 생겼지만 다른 사람"의 의미 입니다. 이 기준은 코드상의 `similarity_threshold` 입니다.

Threshold 값은 실험적으로 동양인의 얼굴은 이 값이 0.4 ~ 0.45 정도의 값이 적당하고, 서양인의 얼굴은 0.5 ~ 0.55정도의 값이 적당했습니다. (추측이지만, 아마도 dlib가 서양인 위주로 학습이 된 것이 아닐까요?)

이 값은 얼굴의 표정, 각도, 이미지 크기, 조명 상태 등 여러 인자에 의해 따라 달라집니다. 다양한 조건에서 이 값이 얼마가 적당한지는 많은 실험과 별도의 알고리즘으로 정해야 하는 문제로, 또 하나의 큰 주제가 될 것입니다.

# 알고리즘의 구현

## 얼굴 인식

이미지에서 얼굴을 인식하는 알고리즘은 `face_recognition` 패키지를 이용하여 아래와 같이 간단히 구현할 수 있습니다.

```python
def detect_faces(self, frame):
    rgb = frame[:, :, ::-1]    # 이미지를 RGB format으로 변환

    # 얼굴 영역을 알아낸다
    boxes = face_recognition.face_locations(rgb, model="hog")
    if not boxes:
        return []

    # 얼굴 영역을 찾았음. face_encoding을 계산
    encodings = face_recognition.face_encodings(rgb, boxes)

    # Face 생성하여 리턴
    faces = []
    for i, box in enumerate(boxes):
        face_image = self.get_face_image(frame, box)   # 얼굴 이미지 추출
        face = Face(str_ms + str(i) + ".png", face_image, encodings[i])
        faces.append(face)
    return faces
```

## 아는 사람의 얼굴과 비교

새로 인식된 얼굴을 기존에 아는 사람의 얼굴과 비교하는 알고리즘은 아래와 같습니다.

1. 새로 얼굴이 인식되면, 아는 사람들의 `face_encoding`과의 거리를 구한다.
2. 가장 가까운 사람과의 거리가 `similarity_threshold`보다 작으면 그 사람의 얼굴이라고 판단하고, 그 사람의 얼굴 DB에 얼굴을 추가한다.
3. 새로 추가된 얼굴을 포함하여 그 사람의 `face_encoding`을 업데이트 한다. (얼굴의 face_encoding의 평균)

얼굴이 추가될 때마다 그 사람의 모든 얼굴의 face_encoding의 평균을 구하여 놓습니다.

```python
def compare_with_known_persons(self, face, persons):
    # distance를 구함
    encodings = [person.encoding for person in persons]
    distances = face_recognition.face_distance(encodings, face.encoding)
    index = np.argmin(distances)
    min_value = distances[index]    # distance의 최소값을 구함
    if min_value < self.similarity_threshold:
        # distance의 최소값이 simalarity_threshold보다 작으면 같은 사람의 얼굴임
        persons[index].add_face(face)
        persons[index].calculate_average_encoding()   # 얼굴의 face_encoding의 평균
        return persons[index]
```

## 새로운 사람 찾아내기

모르는 사람의 얼굴을 비교하여 새로운 사람을 찾아내는 것은 아래와 같은 간단한 알고리즘으로 구현했습니다.

1. 모르는 얼굴은 `unknown_faces`에 따로 저장해 놓는다.
2. 새로 인식된 얼굴과 모르는 얼굴들(`unknown_faces`)과의 거리를 구한다.
3. 가장 가까운 얼굴와의 거리가 `similarity_threshold`보다 작으면 두 얼굴은 같은 사람의 얼굴이라고 판단히고, 새로운 사람을 만든다.
4. 가장 가까운 얼굴와의 거리가 `similarity_threshold`보다 크면 `unknown_faces`에 추가한다.

아래 코드는 이 알고리즘을 구현한 것입니다.

```python
def compare_with_unknown_faces(self, face, unknown_faces):
    # distance를 구함
    encodings = [face.encoding for face in unknown_faces]
    distances = face_recognition.face_distance(encodings, face.encoding)
    index = np.argmin(distances)
    min_value = distances[index]    # distance의 최소값을 구함
    if min_value < self.similarity_threshold:
        # two faces are similar - create new person with two faces
        person = Person()
        newly_known_face = unknown_faces.pop(index)
        person.add_face(newly_known_face)
        person.add_face(face)
        person.calculate_average_encoding()   # 얼굴의 face_encoding의 평균
        return person
    else:
        # unknown face
        unknown_faces.append(face)
        return None
```

새로 만들어진 사람의 이름은 person_## 으로 자동으로 부여되고, ##은 사람이 추가될 때마다 증가합니다 (`person_db.py`). 이 이름은 나중에 수정할 수 있습니다.

# Example Result

<iframe width="560" height="315" src="https://www.youtube.com/embed/f6NPRNQfmJ4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

유튜브에 올려진 영화 '기생충'의 캐릭터 영상으로 테스트한 결과를 보여드리겠습니다.

`youtube-dl`로 위 동영상을 다운 받아서 아래 명령을 실행했습니다. (threshold=0.4, 0.5초 간격)

```bash
$ python face_classifier.py -i ~/Videos/prc.mp4 -t 0.4 -c -S 0.5
```

## Unknown Face

영상은 총 2:06 길이이고, 아래 그림은 대략 1:14초의 결과입니다.

{:refdef: style="text-align: center;"}
![Face Encoding](/assets/img/posts/unknown_classifier_before.png)
{: refdef}

person_03과 person_13은 이전 화면에 나온 적이 있어서 이미 알고 있는 사람입니다. 사진 오른쪽의 두 아이는 이 프레임에서 처음 나왔기 때문에, 둘 다 unknown으로 표시 되었습니다.

## New Person

{:refdef: style="text-align: center;"}
![Face Encoding](/assets/img/posts/unknown_classifier_after.png)
{: refdef}

하지만 0.5초 뒤의 프레임에서는 두 아이가 각각 person_14, person_15로 표시되었습니다. 이전 프레임에서 `unknowns`에 저장된 사진과 비교해 보니 서로 거리가 가까워서 새로 출현한 사람이라고 판단한 것입니다.

# 실행하기

소스코드는 [GitHub](https://github.com/ukayzm/opencv/tree/master/unknown_face_classifier)에 올려져 있고, 마음대로 다운로드 받을 수 있습니다.

## 필요 패키지 설치

python 3.x로 실행해야 하고, virtualenv로 환경을 분리시키기를 추천합니다.
사전에 다음과 같은 패키지가 설치되어 있어야 합니다.

```bash
$ pip install opencv-python
$ pip install opencv-contrib-python
$ pip install dlib
$ pip install face_recognition
$ pip install imutils
```

## 실행

`face_classifier.py`에는 아래와 같은 옵션을 줄 수 있습니다.

```bash
$ python face_classifier.py -h
usage: face_classifier.py [-h] [-t THRESHOLD] [-S SECONDS] [-s STOP] [-k SKIP]
                          [-d] [-c CAPTURE]
                          inputfile

positional arguments:
  inputfile             video file to detect or '0' to detect from web cam

optional arguments:
  -h, --help            show this help message and exit
  -t THRESHOLD, --threshold THRESHOLD
                        threshold of the similarity (default=0.44)
  -S SECONDS, --seconds SECONDS
                        seconds between capture
  -s STOP, --stop STOP  stop detecting after # seconds
  -k SKIP, --skip SKIP  skip detecting for # seconds from the start
  -d, --display         display the frame in real time
  -c CAPTURE, --capture CAPTURE
                        save the frames with face in the CAPTURE directory
```

적당한 비디오 파일을 아래와 같이 `-d` 옵션을 주고 한 번 돌려보세요. 얼굴 영역이 화면에 표시되고, 분류된 사람의 이름이 화면에 나올 것입니다. 처음 실행하면, 사람 얼굴이 모두 `person_##`으로 출력됩니다.

```bash
$ python face_classifier.py <video_file> -d
```

## 종료

동영상 파일을 끝까지 읽었거나 중간에 ^C 또는 `'q'`를 누르면 인식 과정을 멈추고, 결과를 저장하고, 통계를 출력합니다.

```bash
$ python face_classifier.py ~/Videos/prc.mp4 -d
...
Start saving persons in the directory 'result'.
montages saved
result/face_encodings saved
Saving persons finished in 0.920 sec.
8 persons, 60 known faces, 10 unknown faces
person_01  [ 0.000 0.398 0.326 0.529 0.433 0.604 0.369 0.654 ] 0.238, 0.316, 0.432, 18 faces
person_02  [ 0.398 0.000 0.440 0.533 0.485 0.628 0.530 0.591 ] 0.237, 0.289, 0.336, 12 faces
person_03  [ 0.326 0.440 0.000 0.546 0.471 0.623 0.363 0.725 ] 0.243, 0.294, 0.383, 12 faces
person_04  [ 0.529 0.533 0.546 0.000 0.423 0.493 0.528 0.748 ] 0.088, 0.122, 0.161, 3 faces
person_05  [ 0.433 0.485 0.471 0.423 0.000 0.418 0.465 0.717 ] 0.124, 0.190, 0.303, 4 faces
person_06  [ 0.604 0.628 0.623 0.493 0.418 0.000 0.600 0.770 ] 0.210, 0.210, 0.210, 2 faces
person_07  [ 0.369 0.530 0.363 0.528 0.465 0.600 0.000 0.638 ] 0.214, 0.288, 0.370, 7 faces
person_08  [ 0.654 0.591 0.725 0.748 0.717 0.770 0.638 0.000 ] 0.164, 0.164, 0.164, 2 faces
```

## 결과 파일

인식의 얼굴과 분류된 사람은 `result` 디렉토리에 저장됩니다. `result` 디렉토리의 내용을 볼까요?

```bash
$ ls -l result
total 1744
-rw-rw-r-- 1 rostude rostude  77504 Aug 16 17:51 face_encodings
-rw-rw-r-- 1 rostude rostude 278451 Aug 16 17:51 montage.person_01-00.png
-rw-rw-r-- 1 rostude rostude 110646 Aug 16 17:51 montage.person_01-01.png
-rw-rw-r-- 1 rostude rostude 276885 Aug 16 17:51 montage.person_02-00.png
-rw-rw-r-- 1 rostude rostude 296456 Aug 16 17:51 montage.person_03-00.png
-rw-rw-r-- 1 rostude rostude  96322 Aug 16 17:51 montage.person_04-00.png
-rw-rw-r-- 1 rostude rostude  95596 Aug 16 17:51 montage.person_05-00.png
-rw-rw-r-- 1 rostude rostude  58560 Aug 16 17:51 montage.person_06-00.png
-rw-rw-r-- 1 rostude rostude 153792 Aug 16 17:51 montage.person_07-00.png
-rw-rw-r-- 1 rostude rostude  41319 Aug 16 17:51 montage.person_08-00.png
-rw-rw-r-- 1 rostude rostude 238057 Aug 16 17:51 montage.unknowns-00.png
drwxrwxr-x 2 rostude rostude   4096 Aug 16 17:51 person_01/Start saving
drwxrwxr-x 2 rostude rostude   4096 Aug 16 17:51 person_02/
drwxrwxr-x 2 rostude rostude   4096 Aug 16 17:51 person_03/
drwxrwxr-x 2 rostude rostude   4096 Aug 16 17:51 person_04/
drwxrwxr-x 2 rostude rostude   4096 Aug 16 17:51 person_05/
drwxrwxr-x 2 rostude rostude   4096 Aug 16 17:51 person_06/
drwxrwxr-x 2 rostude rostude   4096 Aug 16 17:51 person_07/
drwxrwxr-x 2 rostude rostude   4096 Aug 16 17:51 person_08/
drwxrwxr-x 2 rostude rostude   4096 Aug 16 17:51 unknowns/

```

| file | 설명 |
|------|-----|
| `face_encodings` | 모든 얼굴의 encoding 값이 저장된 pickle 파일 |
| 디렉토리 (사람이름) | 같은 사람으로 판단한 얼굴의 png 파일이 저장됨 |
| montage.*.png | 각 디렉토리에 저장된 얼굴을 보기 쉽게 몽타쥬 형식으로 모은 파일 |

## 결과 재 분류

montage 파일을 열어보고, 같은 사람으로 분류된 얼굴을 확인해 보세요. `similarity_threshold` 값에 따라 결과가 다르겠지만, 대체로 썩 쓸만하게 분류가 된 것을 알 수 있습니다.

디렉토리의 이름은 그 사람의 이름입니다. 디렉토리의 이름을 실제 사람의 이름으로 바꾸어 줍니다.

각 디렉토리에 있는 `.png` 파일을 열고, 분류된 얼굴이 실제로 그 사람의 얼굴이 맞는지 확인해 봅니다. 몇몇 잘못 분류된 얼굴이 보일 수 있습니다. 잘못 분류된 얼굴 파일은 제 디렉토리로 옮겨 줍니다. (복사가 아니라 옮깁니다.) 이 작업을 하면 다음에 실행할 때 얼굴 인식의 정확도가 향상됩니다.

분류할 수 없는 얼굴은 `unknowns` 디렉토리에 옮깁니다.

## 재분류된 데이터 로딩

다음과 같이, 사람의 이름을 바꾸고 다시 한 번 돌려 봅시다.

```bash
$ mv result/person_02 result/gitaek
$ mv result/person_05 result/choongsuk
$ mv result/person_04 result/gijung
$ mv result/person_03 result/giwoo
$ python face_classifier.py ~/Videos/prc.mp4 -d -t 0.42
```

이전에 실행할 때에 비해서 세 가지가 달라진 것을 눈치 채셨나요?

(1) 아래와 같이, 이전에 저정해 두었던 사람과 얼굴 정보를 로딩합니다.

```bash
$ python face_classifier.py ~/Videos/prc.mp4 -d -t 0.42
...
Start loading persons in the directory 'result'.
78 face_encodings in result/face_encodings
unknowns has 12 faces
giwoo has 7 faces
gijung has 8 faces
person_01 has 14 faces
person_07 has 4 faces
person_06 has 3 faces
person_08 has 2 faces
person_09 has 5 faces
gitaek has 16 faces
choongsuk has 7 faces
Loading persons finished in 0.565 sec.
9 persons, 66 known faces, 12 unknown faces
...
```

(2) 로딩한 사람의 이름이 `giwoo`, `gijung`, `gitaek`, `choongsuk` 등으로 변경되었습니다.

(3) 아래 그림과 같이, 화면에 출력할 때에도 변경된 이름이 출력됩니다.

{:refdef: style="text-align: center;"}
![Face Encoding](/assets/img/posts/20200816_183725.946.png)
{: refdef}

## 저장된 결과 지우기

`result` 디렉토리를 삭제하면 다시 실행할 때 이전 결과를 로딩하지 않게 됩니다.

# 응용하기

## 웹캠으로 실시간 인식

다음과 같이 inputfile을 `0`으로 주면 비디오 파일이 아닌 웹캠으로 실시간 캡처를 하고 결과를 모니터 화면에 출력합니다.
저해상도 웹캠 또는 고성능 PC를 사용하시는 분은 -S 옵션을 낮게 주면 더 자연스러운 영상을 얻을 수 있습니다.

```bash
$ python face_classifier.py 0 -d -S 0.5
```

친구들을 웹캠으로 찍어보세요. 결과물의 디렉토리 이름을 친구 이름으로 바꾸고 다시 돌려 보시고, 친구 얼굴 아래 이름이 제대로 출력되는지 보세요. `-t` 옵션으로 정확도를 조절해 보세요.

## 스마트 CCTV

웹캠으로 복도나 출입구 등 주요 장소를 찍어보세요. 웹캠 앞을 지나간 사람들의 얼굴이 자동으로 분류됩니다. 누가 언제 지나갔는지 기록을 남길 수 있습니다. 간단한 인공 지능 CCTV 같지 않나요?

# Reference

* [Python Face Recognition](/python-face-recognition/)
* [Python Face Clustering](/face-clustering/)
* [Source Code - https://github.com/ukayzm/opencv/tree/master/unknown_face_classifier](https://github.com/ukayzm/opencv/tree/master/unknown_face_classifier)
