---
title:  Unknown Face Classifier
tags:   face-recognition
header:
  teaser: /assets/img/posts/unknown_face_classifier_title.png
  image: /assets/img/posts/unknown_face_classifier_title.png
date:   2020-08-16 10:00:00 +0900
sidebar:
  - nav: docs
---

이 포스트에서는 기존에 올렸던 [Python Face Recognition](/python-face-recognition/)을 개선합니다. 모르는 사람의 얼굴을 모아서, 비슷한 얼굴을 찾아내어 새로운 사람을 찾아내는 간단한 방법을 구현합니다. 소스코드는 [여기](https://github.com/ukayzm/opencv/tree/master/unknown_face_classifier)에서 받으실 수 있습니다.

# 얼굴 인식의 원리

얼굴 인식의 원리에 대해서는 이전에 포스트했던 [Python Face Recognition](/python-face-recognition/)나 [Python Face Clustering](/face-clustering/)에서 소개를 했습니다만, 다시 한 번 설명을 하겠습니다.

## 얼굴 인식의 과정

얼굴을 인식하기 위해서는 다음과 같은 세 과정을 거칩니다.

{:refdef: style="text-align: center;"}
![Face Encoding](/assets/img/posts/face_encoding.png)
{: refdef}

1. 그림에서 얼굴이 있는 영역을 알아낸다. (face location)
2. 얼굴 영역에서 눈, 코, 입 등 68개의 주요 좌표를 추출한다. (facial landmarks)
3. 68개의 좌표를 128개의 숫자로 변환한다. (face encoding)

파이썬 패키지 `face_recognition`의 `face_locations()`과 `face_encodings()` 함수는 각각 1 과정과 2, 3 과정을 쉽게 할 수 있도록 만들어 놓은 함수입니다. 

위 과정에 대한 기술적인 내용은 이해하기 어려운 주제이므로, 여기에서는 face encoding의 결과물인 128개 숫자의 특성에 주목합시다.
* 128개 숫자는 deep learning의 결과물이기 때문에, 각 숫자의 의미는 알지 못합니다.
* 하지만 동일한 사람의 얼굴을 입력하면 비슷한 숫자가 나옵니다.

## 얼굴 간의 거리 (유사성) 구하기

우리는 사전에 알고 있는 사람의 얼굴의 face_encoding 값을 가지고 있습니다. 그림에서 얼굴을 인식하면, 그 얼굴의 face_encoding을 기존에 알고 있는 사람의 face_encoding과 비교하여, 가장 거리가 가까운 (즉, 가장 비슷한) 것을 찾아내는 것입니다. 

그렇다면, 128개 숫자 벡터가 서로 얼마나 가까운지 어떻게 비교를 할까요? 수학적으로 [L2 norm (유클리드 거리)](https://ko.wikipedia.org/wiki/%EC%9C%A0%ED%81%B4%EB%A6%AC%EB%93%9C_%EA%B1%B0%EB%A6%AC)를 이용하면 두 벡터의 거리를 구할 수 있고, 파이썬에서는 `numpy`의 `linalg.norm()` 함수를 이용하면 됩니다. 

실제로 파이썬 패키지 `face_recognition`의 `face_distance()` 함수는 `linalg.norm()` 함수의 wrapper로 구현되어 있습니다. 두 face_encoding의 거리는 0 ~ 1 사이의 값으로 구해지고, 이 값은 두 얼굴이 얼마나 비슷한지에 대한 척도로 사용할 수 있습니다.

## Similarity Threshold

이 때, 단순히 거리가 가까운 사람만 찾는 것이 아니라, 사전에 정해진 기준 보다 가까워야 합니다. 사전에 정해진 기준이란 코드상에 `similarity_threshold` 값 입니다. 

Threshold 값은 실험적으로 동양인의 얼굴은 이 값이 0.4 ~ 0.45 정도의 값이 적당하고, 서양인의 얼굴은 0.5 ~ 0.55정도의 값이 적당했습니다. 정해진 값은 없었습니다.

이 값은 얼굴의 크기, 각도, 조명 상태 등 여러 인자에 의해 따라 달라집니다. 다양한 조건에서 이 값이 얼마가 적당한지는 많은 실험과 별도의 알고리즘으로 정해야 하는 문제로, 또 하나의 큰 주제가 될 것입니다.

# 아는 사람의 얼굴과 비교

새로 인식된 얼굴을 기존에 아는 사람의 얼굴과 비교하는 코드는 아래와 같습니다.

1. 새로 얼굴이 인식되면, 아는 사람의 얼굴과의 거리를 구한다.
2. 가장 가까운 사람과의 거리가 `similarity_threshold`보다 작으면 그 사람의 얼굴이라고 판단하고, 그 사람의 얼굴 DB에 얼굴을 추가한다.
3. 새로 추가된 얼굴을 포함하여 그 사람의 `face_encoding`을 업데이트 한다. (face_encoding의 평균을 구함, person_db.py에 구현되어 있음)

{% highlight python linenos %}
def compare_with_known_persons(self, face, persons):
    # see if the face is a match for the faces of known person
    encodings = [person.encoding for person in persons]
    distances = face_recognition.face_distance(encodings, face.encoding)
    index = np.argmin(distances)
    min_value = distances[index]
    if min_value < self.similarity_threshold:
        # face of known person
        persons[index].add_face(face)
        face.name = persons[index].name
        return persons[index]
{% endhighlight %}

# 새로운 사람 찾아내기

모르는 사람의 얼굴을 비교하여 새로운 사람을 찾아내는 것은 아래와 같은 알고리즘으로 구현할 수 있습니다.

1. 모르는 얼굴은 `unknown_faces`에 따로 저장해 놓는다.
2. 새로 인식된 얼굴과 모르는 얼굴(`unknown_faces`)과의 거리를 구한다.
3. 가장 가까운 얼굴와의 거리가 `similarity_threshold`보다 작으면 두 얼굴은 같은 사람의 얼굴이라고 판단히고, 새로운 사람을 만든다.
4. 가장 가까운 얼굴와의 거리가 `similarity_threshold`보다 크면 `unknown_faces`에 추가한다.

아래 코드는 이 알고리즘을 구현한 것입니다.

{% highlight python linenos %}

def compare_with_unknown_faces(self, face, unknown_faces):
    encodings = [face.encoding for face in unknown_faces]
    distances = face_recognition.face_distance(encodings, face.encoding)
    index = np.argmin(distances)
    min_value = distances[index]
    if min_value < self.similarity_threshold:
        # two faces are similar - create new person with two faces
        person = Person()
        newly_known_face = unknown_faces.pop(index)
        person.add_face(newly_known_face)
        person.add_face(face)
        newly_known_face.name = person.name
        return person
    else:
        # unknown face
        unknown_faces.append(face)
        return None

{% endhighlight %}

## 동영상으로 테스트한 결과

<iframe width="560" height="315" src="https://www.youtube.com/embed/f6NPRNQfmJ4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
유튜브에 올려진 영화 '기생충'의 캐릭터 영상으로 테스트한 결과를 보여드리겠습니다. 영상은 총 2:06 길이이고, 아래 결과는 대략 1:14초의 것입니다. 아래 명령으로 캡처한 것입니다. `(threshold=0.4)`

```bash
python face_classifier.py -i ~/Videos/prc.mp4 -t 0.4 -c -s 0.5
```

{:refdef: style="text-align: center;"}
![Face Encoding](/assets/img/posts/unknown_classifier_before.png)
{: refdef}

person_03과 person_13은 이전 화면에 나온 적이 있어서 이미 알고 있는 사람입니다. 사진 오른쪽의 두 아이는 처음 나왔기 때문에 이 프레임에서는 둘 다 unknown으로 처리 되었습니다.

{:refdef: style="text-align: center;"}
![Face Encoding](/assets/img/posts/unknown_classifier_after.png)
{: refdef}

하지만 그 다음 프레임에서는 두 아이가 각각 person_14, person_15로 분류되었습니다. 이전 프레임에서 unknowns에 저장된 사진과 비교해 보니 서로 거리가 가까워서 새로 출현한 사람이라고 판단한 것입니다.

# 프로그램의 사용법

소스코드는 두 파일로 구성되어 있습니다.

## person_db.py

`person_db.py` 파일은 'Face'와 'Person'을 file로 저장하고 로딩하는 기능이 구현되어 있습니다. 

# 웹캠으로 가지고 놀기



# Reference

* [Python Face Recognition](/python-face-recognition/)
* [Source Code](https://github.com/ukayzm/opencv/tree/master/unknown_face_classifier)
