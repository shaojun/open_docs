put elenet dataset into the folder of  _yolov5_ , refer folder structure:   _yolov5/data/elenet/images/train/_ 

the content of _dataset.yaml_ , please make sure the class name sequence:
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

`export` for latest yolov5 repo:
```
python3 export.py --data=data/elenet/dataset.yaml --weights runs/train/exp3/weights/last.pt --img 1280 --batch 1 --opset 12
```
 **onnx 1.6.0 only support python3.7** 
`export` for rknn specified(commit id:  _c5360f6e7009eb4d05f14d1cc9dae0963e949213_  use: `git checkout c5360f6e7009eb4d05f14d1cc9dae0963e949213` to switch to, or `git checkout origin` to re-point to latest HEAD) yolov5 repo:
```
python3 export.py --weights runs/train/exp/weights/last.pt --img 1280 --batch 1 --opset 12
```
should see `last.onnx` and `last.torchscript` are there under `runs/train/exp3/weights/`

`simplifier` for rknn specified version, for the `.onnx` model as required by RKNN:
pip3 install onnx-simplifier
```
python3 -m onnxsim runs/train/exp/weights/last.onnx  runs/train/exp/weights/elenet_yolov5s.onnx
```