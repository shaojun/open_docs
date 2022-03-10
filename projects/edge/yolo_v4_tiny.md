it basically followed the [deepstream_tao_apps](https://github.com/NVIDIA-AI-IOT/deepstream_tao_apps), or see below my steps.


* Clone source

```

git clone https://github.com/NVIDIA-AI-IOT/deepstream_tao_apps.git
```

* update the `libnvinfer_plugin.so.x.y.z`

`x.y.z` is version of the `.so` file, you need download and update by your board,

check your jetpack version, the SD card image mostly explained the jetpack version:
jetson-nx-jp46-sd-card-image.zip indicates its jetpack 4.6
jetson-nano-2gb-jp461-sd-card-image.zip indicates its jetpack 4.6.1

for DS6.0 GA	and jetpack 4.6, should download TRT [8.0.1](https://github.com/NVIDIA-AI-IOT/deepstream_tao_apps/tree/master/TRT-OSS/Jetson/TRT8.0)
for DS6.0.1 GA	and jetpack 4.6, should download TRT [8.2.1](https://github.com/NVIDIA-AI-IOT/deepstream_tao_apps/tree/master/TRT-OSS/Jetson/TRT8.2)

then backup the old file in your current board, and copy in the new one just download, and reload:


```
sudo mv /usr/lib/aarch64-linux-gnu/libnvinfer_plugin.so.8.x.y ${HOME}/libnvinfer_plugin.so.8.x.y.bak   // backup original libnvinfer_plugin.so.x.y
sudo cp libnvinfer_plugin.so.8.m.n  /usr/lib/aarch64-linux-gnu/libnvinfer_plugin.so.8.x.y  //m.n is the one just download, x.y is the one to be overwrite
sudo ldconfig

```
* Build the sample (for get the post_processor for yolo_v4_tiny)



```
cd ~/deepstream_tao_apps
export CUDA_VER=xy.z                                      // xy.z is CUDA version, e.g. 10.2
make
```
make sure then you can see the file `nvdsinfer_custombboxparser_tao.so` is in `~/deepstream_tao_apps/post_processor`


this is the real can work pgie config file, of course you need copy that `nvdsinfer_custombboxparser_tao.so` to the target path :

################################################################################
# Copyright (c) 2021, NVIDIA CORPORATION. All rights reserved.
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
################################################################################

[property]
gpu-id=0
net-scale-factor=1.0
offsets=103.939;116.779;123.68
model-color-format=1
labelfile-path=/opt/nvidia/deepstream/deepstream-6.0/sources/deepstream_python_apps/apps/deepstream-test51-on-test4/models/yolo_v4_tiny/export/labels.txt
model-engine-file=../../models/yolov4-tiny/yolov4_cspdarknet_tiny_epoch_080.etlt.onnx.etlt_b1_gpu0_int8.engine
#int8-calib-file=../../models/yolov4-tiny/cal.bin
tlt-encoded-model=/opt/nvidia/deepstream/deepstream-6.0/sources/deepstream_python_apps/apps/deepstream-test51-on-test4/models/yolo_v4_tiny/export/yolov4_cspdarknet_tiny_epoch_120.etlt
tlt-model-key=nvidia_tlt
infer-dims=3;1280;960
maintain-aspect-ratio=1
uff-input-order=0
uff-input-blob-name=Input
batch-size=1
## 0=FP32, 1=INT8, 2=FP16 mode
network-mode=1
num-detected-classes=2
interval=0
gie-unique-id=1
is-classifier=0
#network-type=0
cluster-mode=3
output-blob-names=BatchedNMS
parse-bbox-func-name=NvDsInferParseCustomBatchedNMSTLT
custom-lib-path=/opt/nvidia/deepstream/deepstream-6.0/sources/deepstream_python_apps/apps/deepstream-test51-on-test4/models/yolo_v4_tiny/export/libnvds_infercustomparser_tao.so

[class-attrs-all]
pre-cluster-threshold=0.3
roi-top-offset=0
roi-bottom-offset=0
detected-min-w=0
detected-min-h=0
detected-max-w=0
detected-max-h=0