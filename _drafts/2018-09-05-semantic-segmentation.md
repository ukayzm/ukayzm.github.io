---
title:  "Semantic Segmentation with OpenCV and ENet"
tags:   object-detection
feature-img: "assets/img/posts/face_clustering.png"
thumbnail:   "assets/img/posts/face_clustering.png"
date:   2018-09-05 12:00:00 +0900
layout: post
---

## Install ENet

REF: [Tutorial on how to train and test ENet on Cityscapes dataset](https://github.com/TimoSaemann/ENet/tree/master/Tutorial)

### clone ENet

```bash
$ git clone --recursive https://github.com/TimoSaemann/ENet.git
```

### compile the modified Caffe framework caffe-enet

It supports all necessary layers for ENet.

```bash
$ sudo apt-get install -y libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev protobuf-compiler gfortran libjpeg62 libfreeimage-dev libatlas-base-dev git python-dev python-pip libgoogle-glog-dev libbz2-dev libxml2-dev libxslt1-dev libffi-dev libssl-dev libgflags-dev liblmdb-dev
$ cd ENet/caffe-enet
$ mkdir build && cd build
$ cmake ..
$ make all -j8
```

### Download Dataset

Download city scape dataset from https://www.cityscapes-dataset.com/
Please use your work or university email address as we use this to verify your technical and research interest. And you'd better specify your work/university name and homepage when registering, to speed up the process.

Once registered, go to download tab and download the fine labeled Cityscapes dataset leftImg8bit_trainvaltest.zip (11GB) and the corresponding ground truth gtFine_trainvaltest.zip (241MB).

And In addition, clone the cityscapesScripts repository:
```
$ git clone https://github.com/mcordts/cityscapesScripts.git
```
After that, run the /preparation/createTrainIdLabelImags.py script, to convert annotations in polygonal format to png images with label IDs, where pixels encode "train IDs" (that you can define in labels.py). 


## References

* [ENet: A Deep Neural Network Architecture for Real-Time Semantic Segmentation](https://modeldepot.io/timosaemann/enet)
* [Tutorial on how to train and test ENet on Cityscapes dataset](https://github.com/TimoSaemann/ENet/tree/master/Tutorial)
* [Semantic segmentation with OpenCV and deep learning in PyImageSearch site](https://www.pyimagesearch.com/2018/09/03/semantic-segmentation-with-opencv-and-deep-learning/)

## Source Code Download
