# CASE: `detectnet_v2` - resnet18 - 4 classes


## dataset intro

99% `bicycle` are from public dataset (open image), 99% `electric-bicycle` are from proprietary.

```
Number of images in the train/val set. 3296
Number of labels in the train/val set. 3296
Number of images in the test set. 359

TFrecords conversion spec file for kitti training
kitti_config {
  root_directory_path: "/workspace/tao-experiments/data/training"
  image_dir_name: "image_2"
  label_dir_name: "label_2"
  image_extension: ".jpg"
  partition_mode: "random"
  num_partitions: 2
  val_split: 8
  num_shards: 10
}
image_directory_path: "/workspace/tao-experiments/data/training"

root: Cumulative object statistics
2022-03-07 02:11:04,732 [INFO] root: {
    "electric_bicycle": 2041,
    "people": 4439,
    "door_warning_sign": 2242,
    "bicycle": 1975
}
```
## spec

### detectnet_v2_train_resnet18_kitti.txt

## result
bicycle ap is around 40%.
others ap are above 80%.


# CASE: `yolo_v4_tiny` - cspdarknet_tiny - 2 classes

## dataset intro

99% `bicycle` are from public dataset (open image), 99% `electric-bicycle` are from proprietary.

```
Number of images in the train/val set. 3296
Number of labels in the train/val set. 3296
Number of images in the test set. 359

!tao yolo_v4_tiny kmeans -l $DATA_DOWNLOAD_DIR/training/label_2 \
                          -i $DATA_DOWNLOAD_DIR/training/image_2 \
                          -n 6 \
                          -x 960 \
                          -y 1280
Start optimization iteration: 1
Start optimization iteration: 11
Start optimization iteration: 21
Please use following anchor sizes in YOLO config:
(60.00, 43.00)
(101.00, 161.00)
(210.00, 257.00)
(311.00, 417.00)
(427.00, 326.00)
(498.00, 489.00)

!tao yolo_v4_tiny dataset_convert -d $SPECS_DIR/yolo_v4_tiny_tfrecords_kitti_train.txt \
                             -o $DATA_DOWNLOAD_DIR/training/tfrecords/train
root: Cumulative object statistics
2022-03-09 06:09:18,812 [INFO] root: {
    "bicycle": 1644,
    "people": 3918,
    "electric_bicycle": 1853,
    "door_warning_sign": 2034
}

```

## spec

### yolo_v4_tiny_train_kitti



```
random_seed: 42
yolov4_config {
  big_anchor_shape: "[(498.00, 489.00), (427.00, 326.00), (311.00, 417.00)]"
  mid_anchor_shape: "[(210.00, 257.00), (101.00, 161.00), (60.00, 43.00)]"
  box_matching_iou: 0.25
  matching_neutral_box_iou: 0.5
  arch: "cspdarknet_tiny"
  loss_loc_weight: 1.0
  loss_neg_obj_weights: 1.0
  loss_class_weights: 1.0
  label_smoothing: 0.0
  big_grid_xy_extend: 0.05
  mid_grid_xy_extend: 0.05
  freeze_bn: false
  #freeze_blocks: 0
  force_relu: false
}
training_config {
  batch_size_per_gpu: 4
  num_epochs: 80
  enable_qat: true
  checkpoint_interval: 10
  learning_rate {
    soft_start_cosine_annealing_schedule {
      min_learning_rate: 1e-7
      max_learning_rate: 1e-4
      soft_start: 0.3
    }
  }
  regularizer {
    type: L1
    weight: 3e-5
  }
  optimizer {
    adam {
      epsilon: 1e-7
      beta1: 0.9
      beta2: 0.999
      amsgrad: false
    }
  }
  pretrain_model_path: "/workspace/tao-experiments/yolo_v4_tiny/pretrained_cspdarknet_tiny/pretrained_object_detection_vcspdarknet_tiny/cspdarknet_tiny.hdf5"
}
eval_config {
  average_precision_mode: SAMPLE
  batch_size: 4
  matching_iou_threshold: 0.5
}
nms_config {
  confidence_threshold: 0.001
  clustering_iou_threshold: 0.5
  force_on_cpu: true
  top_k: 200
}
augmentation_config {
  hue: 0.1
  saturation: 1.5
  exposure:1.5
  vertical_flip:0
  horizontal_flip: 0.5
  jitter: 0.3
  output_width: 960
  output_height: 1280
  output_channel: 3
  randomize_input_shape_period: 10
  mosaic_prob: 0.5
  mosaic_min_ratio:0.2
}
dataset_config {
  data_sources: {
      tfrecords_path: "/workspace/tao-experiments/data/training/tfrecords/train*"
      image_directory_path: "/workspace/tao-experiments/data/training"
  }
  include_difficult_in_training: true
  image_extension: "png"
  target_class_mapping {
      key: "electric_bicycle"
      value: "electric_bicycle"
  }
  target_class_mapping {
      key: "bicycle"
      value: "bicycle"
  }
  validation_data_sources: {
      tfrecords_path: "/workspace/tao-experiments/data/val/tfrecords/val*"
      image_directory_path: "/workspace/tao-experiments/data/val"
  }
}


```

### yolo_v4_tiny_retrain_kitti


```
random_seed: 42
yolov4_config {
  big_anchor_shape: "[(498.00, 489.00), (427.00, 326.00), (311.00, 417.00)]"
  mid_anchor_shape: "[(210.00, 257.00), (101.00, 161.00), (60.00, 43.00)]"
  box_matching_iou: 0.25
  matching_neutral_box_iou: 0.5
  arch: "cspdarknet_tiny"
  loss_loc_weight: 1.0
  loss_neg_obj_weights: 1.0
  loss_class_weights: 1.0
  label_smoothing: 0.0
  big_grid_xy_extend: 0.05
  mid_grid_xy_extend: 0.05
  freeze_bn: false
  #freeze_blocks: 0
  force_relu: false
}
training_config {
  batch_size_per_gpu: 4
  num_epochs: 120
  enable_qat: true
  checkpoint_interval: 10
  learning_rate {
    soft_start_cosine_annealing_schedule {
      min_learning_rate: 1e-7
      max_learning_rate: 1e-4
      soft_start: 0.3
    }
  }
  regularizer {
    type: NO_REG
    weight: 3e-9
  }
  optimizer {
    adam {
      epsilon: 1e-7
      beta1: 0.9
      beta2: 0.999
      amsgrad: false
    }
  }
  pruned_model_path: "/workspace/tao-experiments/yolo_v4_tiny/experiment_dir_pruned/yolov4_cspdarknet_tiny_pruned.tlt"
}
eval_config {
  average_precision_mode: SAMPLE
  batch_size: 4
  matching_iou_threshold: 0.5
}
nms_config {
  confidence_threshold: 0.001
  clustering_iou_threshold: 0.5
  top_k: 200
  force_on_cpu: true
}
augmentation_config {
  hue: 0.1
  saturation: 1.5
  exposure:1.5
  vertical_flip:0
  horizontal_flip: 0.5
  jitter: 0.3
  output_width: 960
  output_height: 1280
  output_channel: 3
  randomize_input_shape_period: 10
  mosaic_prob: 0.5
  mosaic_min_ratio:0.2
}
dataset_config {
  data_sources: {
      tfrecords_path: "/workspace/tao-experiments/data/training/tfrecords/train*"
      image_directory_path: "/workspace/tao-experiments/data/training"
  }
  include_difficult_in_training: true
  image_extension: "png"
  target_class_mapping {
      key: "electric_bicycle"
      value: "electric_bicycle"
  }
  target_class_mapping {
      key: "bicycle"
      value: "bicycle"
  }
  validation_data_sources: {
      tfrecords_path: "/workspace/tao-experiments/data/val/tfrecords/val*"
      image_directory_path: "/workspace/tao-experiments/data/val"
  }
}
```


## result

after training:


```
42/742 [==============================] - 231s 312ms/step - loss: 21.6840
Epoch 62/80
742/742 [==============================] - 226s 304ms/step - loss: 20.3377
Epoch 63/80
742/742 [==============================] - 226s 304ms/step - loss: 21.0430
Epoch 64/80
742/742 [==============================] - 223s 300ms/step - loss: 20.0373
Epoch 65/80
742/742 [==============================] - 224s 302ms/step - loss: 20.0550
Epoch 66/80
742/742 [==============================] - 225s 303ms/step - loss: 20.0125
Epoch 67/80
742/742 [==============================] - 227s 305ms/step - loss: 20.8231
Epoch 68/80
742/742 [==============================] - 223s 300ms/step - loss: 19.3916
Epoch 69/80
742/742 [==============================] - 228s 307ms/step - loss: 19.9891
Epoch 70/80
742/742 [==============================] - 222s 299ms/step - loss: 19.3283
Producing predictions: 100%|████████████████████| 83/83 [00:08<00:00,  9.98it/s]
Start to calculate AP for each class
*******************************
bicycle       AP    0.44239
electric_bicycleAP    0.66367
              mAP   0.55303
*******************************
Validation loss: 24.070940362401757

Epoch 00070: saving model to /workspace/tao-experiments/yolo_v4_tiny/experiment_dir_unpruned/weights/yolov4_cspdarknet_tiny_epoch_070.tlt
Epoch 71/80
742/742 [==============================] - 223s 300ms/step - loss: 18.6482
Epoch 72/80
742/742 [==============================] - 223s 300ms/step - loss: 18.8654
Epoch 73/80
742/742 [==============================] - 228s 307ms/step - loss: 18.6542
Epoch 74/80
742/742 [==============================] - 224s 302ms/step - loss: 18.8403
Epoch 75/80
742/742 [==============================] - 223s 301ms/step - loss: 17.6739
Epoch 76/80
742/742 [==============================] - 223s 301ms/step - loss: 18.6666
Epoch 77/80
742/742 [==============================] - 224s 301ms/step - loss: 17.8138
Epoch 78/80
742/742 [==============================] - 232s 313ms/step - loss: 18.9792
Epoch 79/80
742/742 [==============================] - 236s 318ms/step - loss: 18.1511
Epoch 80/80
742/742 [==============================] - 229s 308ms/step - loss: 17.7056
Producing predictions: 100%|████████████████████| 83/83 [00:08<00:00,  9.73it/s]
Start to calculate AP for each class
*******************************
bicycle       AP    0.42806
electric_bicycleAP    0.73193
              mAP   0.58
*******************************
Validation loss: 22.93538437693952
```

pruning to a 8M file size model:

```

2022-03-09 15:17:29,992 [INFO] __main__: Pruning ratio (pruned model / original model): 0.33196298006278785
```

after retrain:


```
742/742 [==============================] - 214s 288ms/step - loss: 15.5172
Epoch 99/120
742/742 [==============================] - 214s 289ms/step - loss: 16.5217
Epoch 100/120
742/742 [==============================] - 214s 289ms/step - loss: 16.0177
Producing predictions: 100%|████████████████████| 83/83 [00:08<00:00,  9.47it/s]
Start to calculate AP for each class
*******************************
bicycle       AP    0.37325
electric_bicycleAP    0.7106
              mAP   0.54192
*******************************
Validation loss: 25.636139005063526

Epoch 00100: saving model to /workspace/tao-experiments/yolo_v4_tiny/experiment_dir_retrain/weights/yolov4_cspdarknet_tiny_epoch_100.tlt
Epoch 101/120
742/742 [==============================] - 214s 289ms/step - loss: 15.3933
Epoch 102/120
742/742 [==============================] - 214s 289ms/step - loss: 16.1672
Epoch 103/120
742/742 [==============================] - 215s 290ms/step - loss: 16.7339
Epoch 104/120
742/742 [==============================] - 214s 288ms/step - loss: 15.0344
Epoch 105/120
742/742 [==============================] - 216s 291ms/step - loss: 16.1969
Epoch 106/120
742/742 [==============================] - 222s 299ms/step - loss: 15.4994
Epoch 107/120
742/742 [==============================] - 217s 293ms/step - loss: 14.9973
Epoch 108/120
742/742 [==============================] - 218s 294ms/step - loss: 15.5617
Epoch 109/120
742/742 [==============================] - 218s 294ms/step - loss: 15.6581
Epoch 110/120
742/742 [==============================] - 217s 292ms/step - loss: 15.5053
Producing predictions: 100%|████████████████████| 83/83 [00:08<00:00, 10.33it/s]
Start to calculate AP for each class
*******************************
bicycle       AP    0.43033
electric_bicycleAP    0.65164
              mAP   0.54099
*******************************
Validation loss: 24.454951493136853

Epoch 00110: saving model to /workspace/tao-experiments/yolo_v4_tiny/experiment_dir_retrain/weights/yolov4_cspdarknet_tiny_epoch_110.tlt
Epoch 111/120
742/742 [==============================] - 217s 293ms/step - loss: 14.6844
Epoch 112/120
742/742 [==============================] - 218s 294ms/step - loss: 15.5103
Epoch 113/120
742/742 [==============================] - 217s 293ms/step - loss: 15.2786
Epoch 114/120
742/742 [==============================] - 216s 291ms/step - loss: 15.0304
Epoch 115/120
742/742 [==============================] - 217s 293ms/step - loss: 14.9933
Epoch 116/120
742/742 [==============================] - 217s 292ms/step - loss: 15.0265
Epoch 117/120
742/742 [==============================] - 217s 292ms/step - loss: 14.5450
Epoch 118/120
742/742 [==============================] - 221s 298ms/step - loss: 15.1534
Epoch 119/120
742/742 [==============================] - 215s 290ms/step - loss: 13.7307
Epoch 120/120
742/742 [==============================] - 218s 294ms/step - loss: 14.8809
Producing predictions: 100%|████████████████████| 83/83 [00:07<00:00, 10.38it/s]
Start to calculate AP for each class
*******************************
bicycle       AP    0.40626
electric_bicycleAP    0.69524
              mAP   0.55075
*******************************
Validation loss: 23.972072923039814

Epoch 00120: saving model to /workspace/tao-experiments/yolo_v4_tiny/experiment_dir_retrain/weights/yolov4_cspdarknet_tiny_epoch_120.tlt
```


by manual checking inference testing images, confusing object almost gone, but some frames have no labels at all, some are mis-labelled.


# CASE: `classification` - resnet18 - 2 classes

## dataset intro

cropped from previous dataset, that bicycle most from public dataset, electric-bicycle most from proprietary.

## spec

### classification_spec.cfg
```
model_config {
  arch: "resnet",
  n_layers: 18
  # Setting these parameters to true to match the template downloaded from NGC.
  use_batch_norm: true
  all_projections: true
  freeze_blocks: 0
  freeze_blocks: 1
  input_image_size: "3,224,224"
}
train_config {
  train_dataset_path: "/workspace/tao-experiments/data/split/train"
  val_dataset_path: "/workspace/tao-experiments/data/split/val"
  pretrained_model_path: "/workspace/tao-experiments/classification/pretrained_resnet18/pretrained_classification_vresnet18/resnet_18.hdf5"
  optimizer {
    sgd {
    lr: 0.01
    decay: 0.0
    momentum: 0.9
    nesterov: False
  }
}
  batch_size_per_gpu: 64
  n_epochs: 80
  n_workers: 16
  preprocess_mode: "caffe"
  enable_random_crop: True
  enable_center_crop: True
  label_smoothing: 0.0
  mixup_alpha: 0.1
  # regularizer
  reg_config {
    type: "L2"
    scope: "Conv2D,Dense"
    weight_decay: 0.00005
  }

  # learning_rate
  lr_config {
    step {
      learning_rate: 0.006
      step_size: 10
      gamma: 0.1
    }
  }
}
eval_config {
  eval_dataset_path: "/workspace/tao-experiments/data/split/test"
  model_path: "/workspace/tao-experiments/classification/output/weights/resnet_080.tlt"
  top_k: 3
  batch_size: 256
  n_workers: 8
  enable_center_crop: True
}

```

### classification_retrain_spec.cfg


```
model_config {
  arch: "resnet",
  n_layers: 18
  use_batch_norm: true
  all_projections: true
  input_image_size: "3,224,224"
}
train_config {
  train_dataset_path: "/workspace/tao-experiments/data/split/train"
  val_dataset_path: "/workspace/tao-experiments/data/split/val"
  pretrained_model_path: "/workspace/tao-experiments/classification/output/resnet_pruned/resnet18_nopool_bn_pruned.tlt"
  optimizer {
    sgd {
    lr: 0.01
    decay: 0.0
    momentum: 0.9
    nesterov: False
  }
}
  batch_size_per_gpu: 64
  n_epochs: 80
  n_workers: 16
  preprocess_mode: "caffe"
  enable_random_crop: True
  enable_center_crop: True
  label_smoothing: 0.0
  mixup_alpha: 0.1
  # regularizer
  reg_config {
    type: "L2"
    scope: "Conv2D,Dense"
    weight_decay: 0.00005
  }

  # learning_rate
  lr_config {
    step {
      learning_rate: 0.006
      step_size: 10
      gamma: 0.1
    }
  }
}
eval_config {
  eval_dataset_path: "/workspace/tao-experiments/data/split/test"
  model_path: "/workspace/tao-experiments/classification/output_retrain/weights/resnet_080.tlt"
  top_k: 3
  batch_size: 256
  n_workers: 8
  enable_center_crop: True
}
```


## result

after train:


```
71/71 [==============================] - 9s 126ms/step - loss: 0.3371 - acc: 0.9738 - val_loss: 0.2588 - val_acc: 0.9864
Epoch 73/80
71/71 [==============================] - 9s 125ms/step - loss: 0.3425 - acc: 0.9667 - val_loss: 0.2588 - val_acc: 0.9859
Epoch 74/80
71/71 [==============================] - 9s 122ms/step - loss: 0.3440 - acc: 0.9734 - val_loss: 0.2593 - val_acc: 0.9855
Epoch 75/80
71/71 [==============================] - 9s 125ms/step - loss: 0.3443 - acc: 0.9733 - val_loss: 0.2585 - val_acc: 0.9864
Epoch 76/80
71/71 [==============================] - 9s 127ms/step - loss: 0.3411 - acc: 0.9716 - val_loss: 0.2586 - val_acc: 0.9855
Epoch 77/80
71/71 [==============================] - 9s 126ms/step - loss: 0.3468 - acc: 0.9690 - val_loss: 0.2588 - val_acc: 0.9855
Epoch 78/80
71/71 [==============================] - 9s 126ms/step - loss: 0.3480 - acc: 0.9720 - val_loss: 0.2594 - val_acc: 0.9855
Epoch 79/80
71/71 [==============================] - 9s 126ms/step - loss: 0.3382 - acc: 0.9729 - val_loss: 0.2582 - val_acc: 0.9855
Epoch 80/80
71/71 [==============================] - 9s 127ms/step - loss: 0.3423 - acc: 0.9735 - val_loss: 0.2600 - val_acc: 0.9859
2022-03-08 07:36:56,729 [INFO] __main__: Total Val Loss: 0.26003143191337585
2022-03-08 07:36:56,729 [INFO] __main__: Total Val accuracy: 0.9859340786933899
2022-03-08 07:36:56,729 [INFO] __main__: Training finished successfully.
```

after pruning:

```

2022-03-08 07:39:09,820 [INFO] iva.common.magnet_prune: Pruning ratio (pruned model / original model): 0.6353601900291853
```

after retrain:


```
71/71 [==============================] - 6s 88ms/step - loss: 0.2779 - acc: 0.9618 - val_loss: 0.1837 - val_acc: 0.9811
Epoch 68/80
71/71 [==============================] - 7s 96ms/step - loss: 0.2769 - acc: 0.9573 - val_loss: 0.1848 - val_acc: 0.9811
Epoch 69/80
71/71 [==============================] - 6s 89ms/step - loss: 0.2795 - acc: 0.9570 - val_loss: 0.1808 - val_acc: 0.9815
Epoch 70/80
71/71 [==============================] - 6s 90ms/step - loss: 0.2784 - acc: 0.9588 - val_loss: 0.1813 - val_acc: 0.9829
Epoch 71/80
71/71 [==============================] - 6s 90ms/step - loss: 0.2669 - acc: 0.9632 - val_loss: 0.1819 - val_acc: 0.9820

Epoch 72/80
71/71 [==============================] - 7s 96ms/step - loss: 0.2829 - acc: 0.9555 - val_loss: 0.1807 - val_acc: 0.9815
Epoch 73/80
71/71 [==============================] - 6s 90ms/step - loss: 0.2719 - acc: 0.9618 - val_loss: 0.1804 - val_acc: 0.9820
Epoch 74/80
71/71 [==============================] - 6s 88ms/step - loss: 0.2727 - acc: 0.9610 - val_loss: 0.1803 - val_acc: 0.9815
Epoch 75/80
71/71 [==============================] - 7s 97ms/step - loss: 0.2717 - acc: 0.9636 - val_loss: 0.1805 - val_acc: 0.9820
Epoch 76/80
71/71 [==============================] - 6s 88ms/step - loss: 0.2768 - acc: 0.9604 - val_loss: 0.1806 - val_acc: 0.9820
Epoch 77/80
71/71 [==============================] - 7s 96ms/step - loss: 0.2842 - acc: 0.9599 - val_loss: 0.1810 - val_acc: 0.9815
Epoch 78/80
71/71 [==============================] - 6s 90ms/step - loss: 0.2835 - acc: 0.9570 - val_loss: 0.1804 - val_acc: 0.9815
Epoch 79/80
71/71 [==============================] - 6s 91ms/step - loss: 0.2789 - acc: 0.9568 - val_loss: 0.1812 - val_acc: 0.9820
Epoch 80/80
71/71 [==============================] - 6s 90ms/step - loss: 0.2829 - acc: 0.9594 - val_loss: 0.1790 - val_acc: 0.9824
2022-03-08 07:50:13,232 [INFO] __main__: Total Val Loss: 0.17897455394268036
2022-03-08 07:50:13,232 [INFO] __main__: Total Val accuracy: 0.9824175834655762
2022-03-08 07:50:13,232 [INFO] __main__: Training finished successfully.
2022-03-08 15:50:14,599 [INFO] tlt.components.docker_handler.docker_handler: Stopping container.

```

after testing:


```
predictions (Dense)             (None, 2)            914         flatten[0][0]                    
==================================================================================================
Total params: 7,334,274
Trainable params: 7,296,370
Non-trainable params: 37,904
__________________________________________________________________________________________________

Found 318 images belonging to 2 classes.
2022-03-08 07:50:58,374 [INFO] __main__: Processing dataset (evaluation): /workspace/tao-experiments/data/split/test
Evaluation Loss: 0.3027826488580344
Evaluation Top K accuracy: 1.0
Found 318 images belonging to 2 classes.
2022-03-08 07:51:03,005 [INFO] __main__: Calculating per-class P/R and confusion matrix. It may take a while...
Confusion Matrix
[[ 70  12]
 [  3 233]]
Classification Report
                  precision    recall  f1-score   support

         bicycle       0.96      0.85      0.90        82
electric_bicycle       0.95      0.99      0.97       236

        accuracy                           0.95       318
       macro avg       0.95      0.92      0.94       318
    weighted avg       0.95      0.95      0.95       318
```






