---
title:  "Tensorflow Instance Segmentation"
date:   2018-05-04 18:40:00 +0900
tags:   tensorflow object-detection
feature-img: "assets/img/pexels/gadget-google-google-wifi-1054554.jpg"
thumbnail:   "assets/img/pexels/gadget-google-google-wifi-1054554.jpg"
layout: post
---

There are several algorithms that implement instance segmentation but the one used by Tensorflow Object Detection API is Mask RCNN.

```
(py3) $ pip install --upgrade moviepy
```

https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md
여기를 보면 구글이 공개한 pre-trained models들을 다운받을 수 있습니다. 그 중에서 mask_rcnn_inception_v2_coco을 사용할 것입니다. 아래와 같이 model을 다운로드 받고 압축을 풉니다.

```
$ wget http://download.tensorflow.org/models/object_detection/mask_rcnn_inception_v2_coco_2018_01_28.tar.gz
$ tar zxf mask_rcnn_inception_v2_coco_2018_01_28.tar.gz
```

```
$ mkdir data
$ cp ~/work/tensorflow/models/research/object_detection/data/mscoco_label_map.pbtxt data
```

## References

https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/instance_segmentation.md
https://towardsdatascience.com/using-tensorflow-object-detection-to-do-pixel-wise-classification-702bf2605182
https://stackoverflow.com/questions/33947823/what-is-semantic-segmentation-compared-to-segmentation-and-scene-labeling
