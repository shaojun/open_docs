
```
conda activate fiftyone
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=https://127.0.0.1:7890
python3
import fiftyone as fo
import fiftyone.zoo as foz
dataset = foz.load_zoo_dataset("open-images-v6", split="train", label_types=["detections"], classes=["Bicycle"], max_samples=100, seed=51, shuffle=True, dataset_name="open-images-bicycle-mini-set",)
session = fo.launch_app(dataset)
```
