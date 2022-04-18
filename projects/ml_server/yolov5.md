dataset.yaml:
names:
- electric_bicycle
- people
- door_warning_sign
- bicycle
nc: 4
train: ./data/elenet/images/train/
val: ./data/elenet/images/val/

```
python3 train.py --epochs 50 --img 1280 --data data/elenet/dataset.yaml --weights ./yolov5s.pt --batch-size 24
```