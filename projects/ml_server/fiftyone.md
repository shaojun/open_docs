
```
conda activate fiftyone

export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=https://127.0.0.1:7890
export ALL_PROXY=socks5://127.0.0.1:7890
export NO_PROXY=localhost,127.0.0.0,127.0.1.1,127.0.1.1,local.home

python3
import fiftyone as fo
import fiftyone.zoo as foz
dataset = foz.load_zoo_dataset("open-images-v6", split="train", label_types=["detections"], classes=["Bicycle"], max_samples=100, seed=51, shuffle=True, dataset_name="open-images-bicycle-mini-set",)
#show local web ui to check downloaded images.
session = fo.launch_app(dataset)

#upload to CVAT
view = dataset.view()
anno_key="cvat_existing_field_final"
view.annotate(anno_key, label_field="detections",label_type="detections",classes=["Bicycle","Person"])
```
