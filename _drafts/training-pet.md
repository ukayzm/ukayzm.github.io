
make sure you are using the latest version of tensorflow models

```
$ mkdir -p ~/work/pet-training
$ cd ~/work/pet-training
$ mkdir data
$ mkdir download            # for intermediate files
```

```
$ cd download               # working directory is ~/work/pet-training/download
$ wget http://www.robots.ox.ac.uk/~vgg/data/pets/data/images.tar.gz
$ wget http://www.robots.ox.ac.uk/~vgg/data/pets/data/annotations.tar.gz
$ tar -xvf images.tar.gz
$ tar -xvf annotations.tar.gz
$ wget http://storage.googleapis.com/download.tensorflow.org/models/object_detection/faster_rcnn_resnet101_coco_11_06_2017.tar.gz
$ tar -xvf faster_rcnn_resnet101_coco_11_06_2017.tar.gz
```
```
$ cd ../data                # working directory is ~/work/pet-training/data
$ cp ~/work/tensorflow/models/research/object_detection/data/pet_label_map.pbtxt .
$ python ~/work/tensorflow/models/research/object_detection/dataset_tools/create_pet_tf_record.py --label_map_path=./pet_label_map.pbtxt --data_dir=../download --output_dir=`pwd`
$ mv pet_train_with_masks.record pet_train.record
```

output:
pet_train_with_masks.record  pet_val_with_masks.record

You have to change the file name - by the bug in create_pet_tf_record.py.

```
$ mv pet_val_with_masks.record pet_val.record
$ ls
pet_label_map.pbtxt  pet_train.record  pet_val.record
```


```
$ cd ../data                # working directory is ~/work/pet-training/data
$ cp ../download/faster_rcnn_resnet101_coco_11_06_2017/model* model/
$ cp ~/work/tensorflow/models/research/object_detection/samples/configs/faster_rcnn_resnet101_pets.config .
$ sed -i "s|PATH_TO_BE_CONFIGURED|/home/rostude/work/pet-training/data|g" faster_rcnn_resnet101_pets.config
```

```
$ python ~/work/tensorflow/models/research/object_detection/train.py --logtostderr --pipeline_config_path=/home/rostude/work/pet-training/data/faster_rcnn_resnet101_pets.config --train_dir=/home/rostude/work/pet-training/training
$ python ~/work/tensorflow/models/research/object_detection/eval.py --logtostderr --pipeline_config_path=/home/rostude/work/pet-training/data/faster_rcnn_resnet101_pets.config --checkpoint_dir=/home/rostude/work/pet-training/training --eval_dir=/home/rostude/work/pet-training/eval

```

```
$ tensorboard --logdir=/home/rostude/work/pet-training/training   # for tensorboard
```
Then open web browser and go to localhost:6006

## Trouble Shooting

1.
refer to https://github.com/EdjeElectronics/TensorFlow-Object-Detection-API-Tutorial-Train-Multiple-Objects-Windows-10/issues/11
when you have this error
```
ValueError: Tried to convert 't' to a tensor and failed. Error: Argument must be a dense tensor: range(0, 3) - got shape [3], but wanted [].
```
modify ~/work/tensorflow/models/research/object_detection/utils/learning_schedules.py as commented.

2.
CAUTION: do not use directory name as ~/XXX directory (beginning with ~).

