---
title:  "Python Face Clustering"
tags:   face-recognition
feature-img: "assets/img/posts/face_clustering.png"
thumbnail:   "assets/img/posts/face_clustering.png"
date:   2018-06-12 11:00:00 +0900
layout: post
---

이전에 포스트했던 [Python Face Recognition]({{ site.url }}/python-face-recognition/)은 웹캠으로 입력받은 비디오에서 이미 알고 있는 얼굴과 비슷한 얼굴을 찾는 예제를 소개했습니다. 사전에 알고 있는 얼굴이 아닌 얼굴은 unknown으로 분류했습니다. 여기서 한 발 더 나아가 unknown 얼굴을 더 활용할 방법이 있는데, 바로 Face Clustering입니다.

* 참고 사이트: [Face Clustering with Python in PyImageSearch site](https://www.pyimagesearch.com/2018/07/09/face-clustering-with-python/) - 이 사이트에서 많은 부분을 참고하였습니다.

Face clustering은 주어진 얼굴들을 비슷한 얼굴끼리 모으고 분류합니다. 기준이 되는 얼굴이나 몇 명의 얼굴이 존재하는지에 대한 정보는 주어지지 않은 채로 말이죠. 이런 면에서, face clustering은 unsupervised learning 입니다. Clustering이란 이름에서 알 수 있듯이, k-NN, k-mean 등의 clustering을 사용할 수 있지만, 분류해야 하는 사람의 수를 모르기 때문에, DBSCAN이나 Chinese whispers clustering 알고리즘을 사용합니다.

여기에서는 python을 이용하여 face clustering을 구현해 보겠습니다. Source code는 [여기](https://github.com/ukayzm/opencv/tree/master/face_clustering)에서 다운로드 받을 수 있습니다.

# Implementation Overview

대충 감이 오셨겠지만, face clustering은 인식된 얼굴을 분류하는 것입니다. 즉, 크게 두 단계로 나눌 수 있습니다.

1. 얼굴 인식하기
1. 인식된 얼굴 분류하기

## Face Recognition

얼굴을 인식하는 것은 [Face Recognition]({{ site.url }}/python-face-recognition/)에서 사용한 방법과 동일합니다. Python의 [face_recognition](https://github.com/ageitgey/face_recognition) 패키지를 이용하여 쉽게 구현할 수 있습니다.

```python
import face_recognition
face_recognition.face_locations() # 이미지에서 인식된 얼굴 영역을 리턴합니다.
face_recognition.face_encodings() # 아래와 같이, 얼굴 영역을 128 크기의 list로 인코딩합니다.
```

![Encoding Face]({{ site.url }}/assets/img/posts/encoding_face.png)

## Face Clustering

인식된 얼굴을 분류하는 DBSCAN 알고리즘은 [Scikit-learn](http://scikit-learn.org/) 패키지에 이미 구현되어 있습니다. 우리는 이것을 가져다 쓰기만 하면 되죠.

```python
from sklearn.cluster import DBSCAN
clt = DBSCAN(metric="euclidean")
clt.fit(encodings) # encodings는 위에서 구한 인코딩의 list 입니다.
```

## References

* [Face Clustering with Python in PyImageSearch site](https://www.pyimagesearch.com/2018/07/09/face-clustering-with-python/)
