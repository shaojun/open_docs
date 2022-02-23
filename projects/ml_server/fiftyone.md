
```
conda activate fiftyone
python3
import fiftyone as fo
import fiftyone.zoo as foz
dataset = foz.load_zoo_dataset("open-images-v6", split="training", label_types=["detections"], attrs=["Bicycle"], classes=["Bicycle"], max_samples=100, seed=51, shuffle=True, dataset_name="open-images-bicycle-mini-set",)
session = fo.launch_app(dataset)
```
