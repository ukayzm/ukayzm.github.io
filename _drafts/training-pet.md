
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
$ ls
pet_label_map.pbtxt  pet_train_with_masks.record  pet_val_with_masks.record
```

output:
pet_train_with_masks.record  pet_val_with_masks.record

You have to change the file name - by the bug in create_pet_tf_record.py.

```
$ mv pet_train_with_masks.record pet_train.record
$ mv pet_val_with_masks.record pet_val.record
$ ls
pet_label_map.pbtxt  pet_train.record  pet_val.record
```


```
$ cd ../data                # working directory is ~/work/pet-training/data
$ cp ../download/faster_rcnn_resnet101_coco_11_06_2017/model* .
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

```
$ cd ../training            # working directory is ~/work/pet-training/training
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

1.
refer to https://github.com/EdjeElectronics/TensorFlow-Object-Detection-API-Tutorial-Train-Multiple-Objects-Windows-10/issues/11
when you have this error
```
ValueError: Tried to convert 't' to a tensor and failed. Error: Argument must be a dense tensor: range(0, 3) - got shape [3], but wanted [].
```
modify ~/work/tensorflow/models/research/object_detection/utils/learning_schedules.py as commented.

```
167 rate_index = tf.reduce_max(tf.where(tf.greater_equal(global_step, boundaries),
168                                       list(range(num_boundaries)),
169                                       [0] * num_boundaries))
```

2.
CAUTION: do not use directory name as ~/XXX directory (beginning with ~).

3.
ImportError: No module named 'contextlib2'
$ pip install contextlib2

4. In the tensorflow model, there was a big merge on June 6th, on which the tutorial does not work. 
I used commit 772964e for this blog.
```
$ cd /home/rostude/work/tensorflow/models/
$ git checkout 772964e --
```


