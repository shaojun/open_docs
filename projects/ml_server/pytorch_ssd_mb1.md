1. `cvdata_clean`
1. `cvdata_split_with_structure`
2. adding lables files to each subset folder
3. start training

```
python3 train_ssd.py --datasets ~/cvdata_source/cvdata/src/cvdata/pascal_dataset/split_with_structure/train/ --validation_dataset ~/cvdata_source/cvdata/src/cvdata/pascal_dataset/split_with_structure/val/ --net mb1-ssd --pretrained_ssd models/mobilenet-v1-ssd-mp-0_675.pth --scheduler cosine --lr 0.01 --t_max 100 --validation_epochs 5 --num_epochs 160 --base_net_lr 0.001  --batch_size 8
```
