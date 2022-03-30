
```
conda activate fiftyone

#export http_proxy='http://127.0.0.1:7890'
#export https_proxy='https://127.0.0.1:7890'
#export ALL_PROXY=socks://127.0.0.1:7890
#export NO_PROXY=localhost,dev-iot.ipos.biz,127.0.0.0,127.0.1.1,127.0.1.1,local.home

python3
import fiftyone as fo
import fiftyone.zoo as foz
import sys
import uuid
dataset = foz.load_zoo_dataset("open-images-v6", split="train", label_types=["detections"], classes=["Bicycle"], 
    max_samples=2000, seed=49, shuffle=True, 
    dataset_name="oi-bi-"+str(uuid.uuid4().hex),)
#show local web ui to check downloaded images.
#session = fo.launch_app(dataset)

#upload to CVAT
view = dataset.view()
# view = dataset.exists("metadata",False)
# filepaths = view.values("filepath")
# for fp in filepaths:
#     missing_metadata = fo.ImageMetadata.build_for(fp)
#     sample = fo.Sample(filepath=fp, metadata=missing_metadata)
#     dataset.add_sample(sample)
#     print(sample)
# dataset.reload()
#print(filepaths)
anno_key="cvat_"+str(uuid.uuid4().hex)
view.annotate(anno_key,task_size=200, label_field="detections",label_type="detections",classes=["Bicycle","Person"])
```
