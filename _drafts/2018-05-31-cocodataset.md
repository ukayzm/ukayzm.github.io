---
title:  "COCO Dataset"
tags:   object-detection
feature-img: "assets/img/posts/instance-segmentation-road.png"
thumbnail:   "assets/img/posts/coco-examples.png"
layout: post
---

## Install packages

```
$ source py3/bin/activate
(py3) $ pip install numpy
(py3) $ pip install scikit-image
(py3) $ pip install matplotlib
```

## Install cocoapi

```
(py3) $ pip install cython
$ git clone https://github.com/cocodataset/cocoapi.git
$ cd cocoapi/PythonAPI
$ make
$ cp -r pycocotools WORKING_DIR
```

## References

* [https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/instance_segmentation.md](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/instance_segmentation.md)
* [https://towardsdatascience.com/using-tensorflow-object-detection-to-do-pixel-wise-classification-702bf2605182](https://towardsdatascience.com/using-tensorflow-object-detection-to-do-pixel-wise-classification-702bf2605182)
* [https://stackoverflow.com/questions/33947823/what-is-semantic-segmentation-compared-to-segmentation-and-scene-labeling](https://stackoverflow.com/questions/33947823/what-is-semantic-segmentation-compared-to-segmentation-and-scene-labeling)
* [https://medium.com/@rohitrpatil/how-to-use-tensorflow-object-detection-api-on-windows-102ec8097699](https://medium.com/@rohitrpatil/how-to-use-tensorflow-object-detection-api-on-windows-102ec8097699)
