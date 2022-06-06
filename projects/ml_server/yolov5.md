- Download the raw dataset with `KITTI` format.
- Create a folder: `prepare_yolov5_dataset_kitti` at the same level of the fiftyone script:
![输入图片说明](create_folder_of_dataset_kitti_at_the_script_same_level.png)


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






```


import fiftyone as fo


def print_hi(name):
    # Use a breakpoint in the code line below to debug your script.
    print(f'Hi, {name}')  # Press Ctrl+F8 to toggle the breakpoint.


# See PyCharm help at https://www.jetbrains.com/help/pycharm/

def load_dataset_from_local_path(dataset_local_dir: str, dataset_name: str,
                                 force_reload_dataset_if_exists: bool = False):
    if dataset_name in fo.list_datasets() and force_reload_dataset_if_exists:
        exists_dataset = fo.load_dataset(dataset_name)
        print("dataset: {} already exists, will delete it...".format(dataset_name))
        exists_dataset.delete()
        print("     deleted with result: {}".format(exists_dataset.deleted))
    elif dataset_name in fo.list_datasets() and not force_reload_dataset_if_exists:
        return fo.load_dataset(dataset_name)
    dataset_type = fo.types.VOCDetectionDataset
    dataset = fo.Dataset.from_dir(
        dataset_dir=dataset_local_dir,
        dataset_type=dataset_type,
        name=dataset_name,
    )
    dataset.persistent = True
    return dataset


def export_dataset_to(src_dataset, export_to_local_dir: str):
    # The splits to export
    splits = ["train", "val"]
    ds_len = len(src_dataset)

    # Perform a random 90-10 test-train split
    src_dataset.take(0.1 * len(src_dataset)).tag_samples("val")
    src_dataset.match_tags("val", bool=False).tag_samples("train")

    for split in splits:
        train_data_view = src_dataset.match_tags(split)
        train_data_view.export(
            export_dir=export_to_local_dir,
            dataset_type=fo.types.YOLOv5Dataset,
            label_field="ground_truth",
            split=split,
        )


# Press the green button in the gutter to run the script.
if __name__ == '__main__':
    src_dataset_local_path = "~/Downloads/project_labeling_door_closedsign_proj-2022_03_24_07_35_06-pascal voc 1.1/for_fiftyone_import"
    dataset_name = "first_time1"
    load_dataset = load_dataset_from_local_path(src_dataset_local_path, dataset_name)
    print(load_dataset)

    target_export_dataset_local_path = "~/Downloads/project_labeling_door_closedsign_proj-2022_03_24_07_35_06-pascal voc 1.1/for_fiftyone_export"
    export_dataset_to(load_dataset, target_export_dataset_local_path)
```
