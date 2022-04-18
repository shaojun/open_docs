put elenet dataset into the folder of  _yolov5_ , refer folder structure:   _yolov5/data/elenet/images/train/_ 

the content of _dataset.yaml_ :
```
names:
- electric_bicycle
- people
- door_warning_sign
- bicycle
nc: 4
train: ./data/elenet/images/train/
val: ./data/elenet/images/val/
```

`train` scripts, batch size 24 will use almost 20G GPU memory:
```
python3 train.py --epochs 50 --img 1280 --data data/elenet/dataset.yaml --weights ./yolov5s.pt --batch-size 24
```

`detect` scripts:
```
python3 detect.py --weights ./runs/train/exp3/weights/last.pt --imgsz 1280 --source ~/Videos/yang_office_demoEle_combined_multiple_sections_4classes.mp4 
```

`export`:
```
python3 export.py --weights runs/train/exp3/weights/last.pt --img 1280 --batch 1 --opset 12
```
should see `last.onnx` and `last.torchscript` are there under `runs/train/exp3/weights/`