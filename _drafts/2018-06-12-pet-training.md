---
title:  "Training on the Pet Dataset"
tags:   object-detection
feature-img: "assets/img/posts/oxford_pet.png"
thumbnail:   "assets/img/posts/oxford_pet.png"
date:   2018-06-11 21:00:00 +0900
layout: post
---

구글은 object detection model을 이용하여 [Pet Dataset을 학습시키는 예제](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/running_pets.md)를 제공하고 있습니다. 예제는 Google Cloud Platform 상에서 학습을 시키도록 되어 있는데, 여기서는 local PC에서 학습 시키는 방법을 설명합니다. 그리고, Google의 예제에서 잘못된 부분을 고치고, 예제 실행시 만나는 문제의 해결책을 제시합니다.

이 예제에서는 실제로 제가 사용한 디렉토리 이름을 그대로 사용했고, 매 단계마다 작업 디렉토리를 명시하였으므로, 좀 더 실용적인 예제가 될 것으로 기대합니다.

## 디렉토리

이 예제에서 사용할 디렉토리는 아래와 같습니다.

* Home directory: `/home/rostude`
* Tensorflow model directory: `/home/rostude/work/tensorflow/models`
* Pet training directory: `/home/rostude/work/pet-training`

디렉토리 이름을 각자의 환경에 맞게 적당히 수정해야 합니다.

## TensorFlow Models

TensorFlow model이 설치되어 있어야 합니다. 설치 방법은 [여기]({{ site.url }}/python-object-detection-tensorflow/)를 참고하세요.

이 글을 작성 중이던 6월 6일 경에 GitHub의 tensorflow/models에 큰 변화가 있었습니다. 그 영향 때문인지, 6월 6일 코드로 `create_pet_tf_record.py`를 실행하면 `pet_train.record`, `pet_val.record` 파일이 예제와 다르게 생성되었습니다. 그래서, 저는 아래와 같이 6월 4일 code로 rollback 하여 테스트를 하였습니다. 자신이 가지고 있는 TensorFlow Model이 6월 4일 이전 버전이면 상관 없습니다.

```bash
$ cd /home/rostude/work/tensorflow/models/
$ git checkout 772964e --
```


## Pet Dataset

먼저 이 예제에서 사용할 Pet Dataset이 어떤 것인지 [http://www.robots.ox.ac.uk/~vgg/data/pets/](http://www.robots.ox.ac.uk/~vgg/data/pets/)를 방문하여 대략 훑어보세요.

## 사전작업

먼저 아래와 같이, training을 시킬 디렉토리를 만듭니다.
```bash
$ mkdir -p ~/work/pet-training
$ mkdir -p ~/work/pet-training/data         # for data, model and config
$ mkdir -p ~/work/pet-training/download     # for downloaded files
```

## Download

아래와 같이, Pet dataset image, annotation을 다운로드 받습니다. 그리고, Google이 공개한 Faster RCNN 모델을 다운로드 받습니다. Faster RCNN 모델을 기반으로 Pet dataset을 학습시킬 것입니다.

```bash
$ cd ~/work/pet-training/download
$ wget http://www.robots.ox.ac.uk/~vgg/data/pets/data/images.tar.gz
$ wget http://www.robots.ox.ac.uk/~vgg/data/pets/data/annotations.tar.gz
$ tar -xvf images.tar.gz
$ tar -xvf annotations.tar.gz
$ wget http://storage.googleapis.com/download.tensorflow.org/models/object_detection/faster_rcnn_resnet101_coco_11_06_2017.tar.gz
$ tar -xvf faster_rcnn_resnet101_coco_11_06_2017.tar.gz
```

## Data 준비

아래와 같이, data 디렉토리를 준비합니다.
* Label file: Tensorflow model에서 미리 준비해 놓은 `pet_label_map.pbtxt` 파일을 북사
* Training, validation 데이터: `create_pet_tf_record.py` 파일을 실행시켜 생성함.

```bash
$ cd ~/work/pet-training/data
$ cp ~/work/tensorflow/models/research/object_detection/data/pet_label_map.pbtxt .
$ python ~/work/tensorflow/models/research/object_detection/dataset_tools/create_pet_tf_record.py \
  --label_map_path=./pet_label_map.pbtxt \
  --data_dir=../download \
  --output_dir=`pwd`
$ ls
pet_label_map.pbtxt  pet_train_with_masks.record  pet_val_with_masks.record
```

`create_pet_tf_record.py`를 실행시키면 위와 같이, `pet_train_with_masks.record`, `pet_val_with_masks.record` 파일이 생성됩니다. 그런데, 이 파일 이름을 아래와 같이 `pet_train.record`, `pet_val.record`로 바꿔 주어야만 이후 단계에서 에러가 발생하지 않습니다.

```bash
$ mv pet_train_with_masks.record pet_train.record
$ mv pet_val_with_masks.record pet_val.record
$ ls
pet_label_map.pbtxt  pet_train.record  pet_val.record
```

## Model

```bash
$ cd ~/work/pet-training/data
$ cp ../download/faster_rcnn_resnet101_coco_11_06_2017/model* .
```

## Config

```bash
$ cd ~/work/pet-training/data
$ cp ~/work/tensorflow/models/research/object_detection/samples/configs/faster_rcnn_resnet101_pets.config .
$ sed -i "s|PATH_TO_BE_CONFIGURED|/home/rostude/work/pet-training/data|g" faster_rcnn_resnet101_pets.config
```

`faster_rcnn_resnet101_pets.config` 파일에는 학습을 위한 각종 config가 들어 있습니다. 이 중에서 반드시 해야 하는 것은 `PATH_TO_BE_CONFIGURED`를 실제 디렉토리 이름으로 바꾸는 것입니다. 위와 같이 sed 명령으로 한번에 바꿔도 되고, vi 로 직접 편집해도 됩니다. 주의할 점은, `~/work/pet-trainig/data` 이런 식으로 ~로 시작하는 경로를 지정하면 안되고, /로 시작하는 절대 경로를 사용해야 합니다.

## Training and evaluation

아래 명령은 아무 디렉토리에서 실행시켜도 됩니다. 여기서도 마찬가지로, 파라미터로 ~로 시작하는 경로를 지정하면 안됩니다.

아래와 같이 python명령으로 Training을 시키면 주기적으로 로그가 출력됩니다.
```bash
$ python ~/work/tensorflow/models/research/object_detection/train.py \
    --logtostderr \
    --pipeline_config_path=/home/rostude/work/pet-training/data/faster_rcnn_resnet101_pets.config \
    --train_dir=/home/rostude/work/pet-training/training
...
INFO:tensorflow:global step 599: loss = 0.9922 (14.020 sec/step)
INFO:tensorflow:global step 600: loss = 0.8763 (16.702 sec/step)
INFO:tensorflow:global step 601: loss = 1.4698 (15.401 sec/step)
INFO:tensorflow:Recording summary at step 601.
INFO:tensorflow:global step 602: loss = 0.3024 (22.146 sec/step)
INFO:tensorflow:global step 603: loss = 0.6957 (18.191 sec/step)
INFO:tensorflow:global step 604: loss = 0.6043 (16.251 sec/step)
INFO:tensorflow:global step 605: loss = 2.1987 (17.995 sec/step)
INFO:tensorflow:global step 606: loss = 0.9268 (17.557 sec/step)
INFO:tensorflow:global step 607: loss = 1.2230 (19.084 sec/step)
INFO:tensorflow:Recording summary at step 607.
INFO:tensorflow:Recording summary at step 607.
INFO:tensorflow:global step 608: loss = 2.8359 (178.634 sec/step)
INFO:tensorflow:global step 609: loss = 0.2520 (49.319 sec/step)
INFO:tensorflow:Saving checkpoint to path /home/rostude/work/pet-training/training/model.ckpt
INFO:tensorflow:global step 610: loss = 0.4853 (63.031 sec/step)
INFO:tensorflow:Recording summary at step 610.
INFO:tensorflow:global step 611: loss = 0.2766 (41.864 sec/step)
INFO:tensorflow:global step 612: loss = 1.2626 (58.015 sec/step)
...
```

터미널을 하나 더 띄워서 아래와 같이 tensorboard를 실행시키고 web browser에서 "localhost:6006"을 띄워서 진행 과정을 모니터링 합니다.
```bash
$ tensorboard --logdir=/home/rostude/work/pet-training/
```

학습이 잘 진행되는지 검증하기 위해, 터미널을 하나 더 띄워서 아래와 같이 evaluation을 실행합니다.
```bash
$ python ~/work/tensorflow/models/research/object_detection/eval.py \
    --logtostderr \
    --pipeline_config_path=/home/rostude/work/pet-training/data/faster_rcnn_resnet101_pets.config \
    --checkpoint_dir=/home/rostude/work/pet-training/training \
    --eval_dir=/home/rostude/work/pet-training/eval
```
저의 경우에는 노트북에서 실행을 시켰는데, evaluation을 실행하면 컴퓨터가 매우 느려졌습니다. 대신에, tensorboard의 image tab에서 evaluation 결과를 확인할 수 있습니다.

![pet-tensorboard-image]({{ site.url }}/assets/img/posts/pet-tensorboard-image.png)

## Checkpoint and Frozen Graph

```bash
$ cd ~/work/pet-training/training
$ ls
checkpoint                          model.ckpt-440.data-00000-of-00001
events.out.tfevents.1528331021.a42  model.ckpt-440.index
events.out.tfevents.1528336914.a42  model.ckpt-440.meta
graph.pbtxt                         model.ckpt-494.data-00000-of-00001
model.ckpt-332.data-00000-of-00001  model.ckpt-494.index
model.ckpt-332.index                model.ckpt-494.meta
model.ckpt-332.meta                 model.ckpt-547.data-00000-of-00001
model.ckpt-385.data-00000-of-00001  model.ckpt-547.index
model.ckpt-385.index                model.ckpt-547.meta
model.ckpt-385.meta                 pipeline.config
$ python ~/work/tensorflow/models/research/object_detection/export_inference_graph.py --input_type image_tensor --pipeline_config_path ../data/faster_rcnn_resnet101_pets.config --trained_checkpoint_prefix=./model.ckpt-547 --output_directory ../freeze
$ ls ../freeze/
checkpoint                 model.ckpt.data-00000-of-00001  model.ckpt.meta  saved_model/
frozen_inference_graph.pb  model.ckpt.index                pipeline.config
```


## Trouble Shooting

구글이 설명한 예제를 그대로 따라해 보면 여러 에러를 만나게 되는데, 에러에 대한 대처법을 아래와 같이 정리하였습니다.

### ValueError
Training을 시키면, 아래와 같은 에러가 발생할 수 있습니다.
```
ValueError: Tried to convert 't' to a tensor and failed. Error: Argument must be a dense tensor: range(0, 3) - got shape [3], but wanted [].
```
이것은 Python version 때문에 발생하는 에러라고 합니다. 이 때는 `~/work/tensorflow/models/research/object_detection/utils/learning_schedules.py` 파일을 아래와 같이 수정합니다. 168째 줄을 `list()`로 감싸는 것입니다.

```python
167 rate_index = tf.reduce_max(tf.where(tf.greater_equal(global_step, boundaries),
168                                       list(range(num_boundaries)),
169                                       [0] * num_boundaries))
```

이 방법은 [https://github.com/EdjeElectronics/TensorFlow-Object-Detection-API-Tutorial-Train-Multiple-Objects-Windows-10/issues/11](https://github.com/EdjeElectronics/TensorFlow-Object-Detection-API-Tutorial-Train-Multiple-Objects-Windows-10/issues/11)를 참고하였습니다.

### OutOfRangeError

```
OutOfRangeError (see above for traceback): FIFOQueue '_2_prefetch_queue' is closed and has insufficient elements (requested 1, current size 0)
```

이 에러는 input data를 읽을 수 없을 때 발생합니다. pet_train.record, pet_val.record 파일이 정상적으로 생성되었는지, 파라미터로 지정한 path가 틀리지는 않았는지, faster_rcnn_resnet101_pets.config에 지정한 경로는 정상인지 검토해 보세요.


### contextlib2

Python을 실행시켰을 때, 아래와 같은 에러가 발생하면 contextlib2를 설치하세요.

```bash
ImportError: No module named 'contextlib2'
$ pip install contextlib2
```


## References

* [https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/running_pets.md](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/running_pets.md)
* [http://www.robots.ox.ac.uk/~vgg/data/pets/](http://www.robots.ox.ac.uk/~vgg/data/pets/)
