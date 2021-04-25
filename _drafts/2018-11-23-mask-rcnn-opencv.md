---
title:  "Mask RCNN using OpenCV"
tags:   object-detection
feature-img: "assets/img/posts/face_clustering.png"
thumbnail:   "assets/img/posts/face_clustering.png"
date:   2018-11-23 12:00:00 +0900
---

## OpenCV 

OpenCV provides dnn.
Necessary files:
Download [model file](http://download.tensorflow.org/models/object_detection/mask_rcnn_inception_v2_coco_2018_01_28.tar.gz) and untar it.
Config file can be acquired [here](https://github.com/opencv/opencv_extra/blob/master/testdata/dnn/mask_rcnn_inception_v2_coco_2018_01_28.pbtxt).
Download config file using this command
```bash
$ wget https://raw.githubusercontent.com/opencv/opencv_extra/master/testdata/dnn/mask_rcnn_inception_v2_coco_2018_01_28.pbtxt
```

## Reference

* [OpenCV TensorFlow Object Detection API](https://github.com/opencv/opencv/wiki/TensorFlow-Object-Detection-API)

